# The record's frontmatter and its identity

A record is an ordinary Markdown file with a block of structured fields at the very top. That block — the *frontmatter* — is the part the engine validates; everything below it is prose and stays yours entirely.

```markdown
---
id: DEC-004
entity: decision
title: "Records are named by their ID"
status: accepted
date: 2026-07-12
---

Anything from here down is prose. Write it however you like.
```

The messages on this page all concern that block: whether it is there, whether it parses, and whether the record's identity is sound.

## `missing YAML frontmatter block (--- ... ---) at top of file`

The engine did not find the block at all. Three ways to trip this:

- **Something precedes the opening `---`.** It must be line 1 — not a blank line before it, not a title, nothing.
- **The closing `---` is missing.** Every block needs both delimiters.
- **The closing `---` is indented.** It must sit at column zero. An indented `---` inside a multi-line value is content, and the engine deliberately reads it as content — cutting the block there would truncate your frontmatter before it was ever parsed.

## `frontmatter is not valid YAML: …`

The block is there but could not be parsed. The trailing text is PyYAML's own message, and three of its shapes account for nearly everything:

- **`mapping values are not allowed here`** — almost always a colon-plus-space inside an unquoted value: `title: see: here`. YAML reads the second colon as a second field. Quote the value.
- **`found character '\t' that cannot start any token`** — a tab. YAML indentation is spaces only; most editors can show you invisible characters.
- **`duplicate key …`** — see [when the file looks fine and YAML disagrees](quoting.md).

## `frontmatter must be a YAML mapping`

The block parsed, but not into `field: value` pairs. The usual cause is a stray leading `-`, which turns the whole block into a list of things instead of a set of named fields.

## `missing or non-string 'id' in frontmatter`

Every record carries its own identity in an `id` field. The filename is not enough and is not consulted for this: filenames are a naming convention, and conventions get renamed, copied and dragged between folders. The frontmatter is where identity lives.

Add `id: <PREFIX-NNN>`. If you do not know which number is free, the engine will tell you — `new <type>` prints a stub carrying the next one.

## `id 'X' does not match pattern PREFIX-<N+ digits>`

The ID is there but does not have the shape this type declared: a prefix, a hyphen, and at least as many digits as the schema's `width` says.

Either the ID is malformed (`DEC4`, `dec-4`, `DEC-4` where the width is 3), or the schema's prefix is not the one you think it is. The prefix is case-sensitive and belongs to exactly one type.

## `duplicate id 'X' (also in <file>)`

Two files claim the same identity, and the engine names the other one. This is nearly always a record created by copying an existing file and editing everything except the `id`.

Decide which file is the real `X`, then give the other a fresh ID — `new <type>` prints the next free one. Do check whether anything already refers to the ID before you reassign it: if other records point at `X`, they are pointing at whichever file keeps it.

## `filename should be the id 'X', optionally followed by '-' and a handle (e.g. X.md, X-a-handle.md)`

Under the default naming policy, a record's filename begins with its ID, and a freely chosen handle may follow after a hyphen — `DEC-004.md` and `DEC-004-naming.md` are both fine, `naming-decision.md` is not.

Two ways to resolve it, and they are genuinely different decisions:

- **Rename the file.** Right when this project wants IDs visible in filenames — which makes every record findable by ID in any file browser.
- **Declare `filename: free` in the schema.** Right when this project wants to name its files in its own words. The ID then lives only in the frontmatter and the engine has no opinion about filenames at all.

The second is a project-level choice, not a per-file escape hatch; make it once, deliberately. Vaults of prose notes usually want `free`; registries of typed records usually want the default.

## `frontmatter entity 'x' does not match directory-implied entity 'y'`

The record declares itself one type, and it is sitting in another type's folder.

One of the two is wrong, and only you know which: move the file if the declaration is right, correct the `entity:` field if the location is right. What the engine will not do is pick — a misplaced file and a mistyped field look identical from the outside, and guessing would silently reclassify a record.

## `unknown field 'x' not declared in schema '<type>'`

The record carries a field its schema never declared. In order of likelihood:

1. **A typo in the field name.** Fix the spelling.
2. **Something that belongs in the prose body**, not in the structured block. Move it below the frontmatter, where it can be written as a sentence.
3. **A field this project genuinely needs now.** Then the schema is what is out of date: add the field to it. Evolving a schema is a normal, expected workflow, not an admission of failure — see [what adopting asks of you](../adopting.md). Log the change in `schemas/CHANGELOG.md`.
4. **A phantom.** An unquoted comma one line above can *manufacture* a field that you never wrote. If the “unknown field” is a fragment of the previous line's sentence, the real error is up there — and the pre-parse scan usually names it first. See [when the file looks fine and YAML disagrees](quoting.md).

A schema may also declare itself `strict: false`, in which case undeclared fields are reported as warnings rather than errors — the right setting for a corpus still finding its shape, and the wrong one for a registry that has settled.

## `field 'x' not declared in schema` (warning, not error)

The same situation, in a schema that declared `strict: false`. It is a note, not a block: the run can still be green. Treat it as a reading of where your records are drifting away from your schema, and revisit it when the drift becomes a pattern.
