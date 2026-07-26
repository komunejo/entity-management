# When the engine says no

This guide is for humans. It explains every error the validation engine (`entity_lint.py`) can raise: what actually happened, why it is worth stopping for, and how to fix it — with examples. The engine's reference documentation ([`schema-language.md`](../entity-management/references/schema-language.md)) specifies how things *should* be written; this page starts from the other end, the moment something *wasn't*, and works back.

One idea organizes everything below. The registry's whole value is that a green run means the records can be trusted; every message in this catalog exists because some way of writing a record would quietly make that green a lie. So when the engine blocks you, it is not enforcing style — it is refusing to vouch for something it cannot back. The fix is usually one line.

## How an error reaches you

Three doors, and it matters which one you came through:

- **You ran the validator yourself** (`validate`), or **a git pre-commit hook blocked your commit**. You are holding the raw output: one line per violation, each naming a file, usually a field or a line number, and a message from the catalog below. The pre-commit block is the door most hand-edits arrive through — it exists precisely because editing a record by hand, with no agent watching, is a normal and legitimate thing to do; the hook is what stands between an honest slip and the repository's history.

- **An agent brought it to you.** This door has a nuance the other two lack. Agents working under this skill are instructed to fix what they can decide on their own — their own typos, anything the conversation already settles — and to bring you *only* the flags they have no standing to resolve. So an agent's question is rarely "how do I fix this?"; it is "what did you mean?". A useful answer settles the intent, not the mechanics: "that comma is part of the sentence, keep it as text" resolves the flag; explaining YAML quoting rules back to the agent does not add anything it doesn't know. And when your answer establishes a rule ("in this project, titles are always literal text — quote them all"), a well-behaved agent records the criterion so the same question never comes back.

- **Exit codes**, if you are scripting: `0` means integrity holds; `1` means violations were found and listed; `2` means the run itself was invalid — no project root, missing PyYAML, a path filter that matched nothing. Never read exit 2 as "no violations": it means the engine could not even look.

## Quoting errors: the silent-misparse family

These four fire *before* the YAML parse, on the raw text you typed. They deserve their own section because they are the least intuitive family in the catalog: **the file looks perfectly fine to a human eye.** The problem is that YAML would have read it differently from how you wrote it — cleanly, without any error, into data that is not what you meant. These checks exist because nothing downstream can catch that: once the parse has happened, the evidence of what you actually typed is gone. (The project's own registry records the four measured cases as ISSUE-003, and the design as PROP-006.)

The single writer-side rule that avoids the whole family: **when a value holds prose, quote it.**

### `multi-word unquoted scalar '…' inside a flow collection — a ',' ':' or '#' in it is read as structure, not text; quote it`

Inside `{…}` or `[…]` (YAML's *flow* style), commas and colons are punctuation *of the data structure*, not of your sentence. An unquoted multi-word value there is standing in a minefield: the moment it grows a comma, YAML splits it into two entries without telling anyone.

```yaml
# What was written — meaning one description:
meta: {description: applies always, required: true}

# What YAML reads — two keys, description truncated, required set:
meta:
  description: applies always
  required: true
```

The engine cannot know whether the comma was your sentence or your structure, so it flags the ambiguity and asks for the two characters that settle it:

```yaml
meta: {description: "applies always, required: true"}   # one value, protected
```

If you genuinely meant two entries, quoting the words in each also silences the flag — either way, the text now says what you mean where YAML can see it.

### `unquoted value contains ' #' — from the '#' on, YAML reads a comment and the tail silently vanishes from the value; quote the value to keep it (or to make the comment unambiguous)`

A `#` preceded by whitespace starts a comment — anywhere, not just at the start of a line. The tail of your value evaporates, with no error:

```yaml
title: count # per item        # YAML stores: "count"
```

Two honest ways out, depending on what you meant:

```yaml
title: "count # per item"      # the # was part of the title
title: "count"  # per item     # the # really was a comment
```

### `' #' inside a flow collection starts a comment — the rest of the line silently vanishes; quote the scalar`

The same comment rule, caught inside `{…}`/`[…]` — where the vanished tail typically takes the closing bracket with it. Same remedy: quote the scalar the `#` belongs to.

### `duplicate key 'x' in mapping (line N) — the first value would be silently discarded`

YAML's default behavior with a repeated key is to keep the last value and throw the earlier ones away without a word. There is no reading of a mapping that names a key twice in which both writings were meant, so this one is never a matter of intent — delete the wrong line, keep the right one. The line number in the message points at the *second* occurrence. This usually comes from an editing slip: a field added at the bottom of frontmatter that already had it at the top.

## Frontmatter structure

### `missing YAML frontmatter block (--- ... ---) at top of file`

Every record starts with a frontmatter block: a `---` line, YAML fields, a closing `---` line, all at the very top — the opening delimiter must be line 1. Two easy ways to trip it: a blank line (or a BOM-less encoding surprise) before the first `---`, or a closing delimiter that is not where the engine looks. Note that the closing `---` must be at **column zero**: an *indented* `---` inside a multi-line value is content, not a delimiter, and the engine deliberately reads it as such.

### `frontmatter is not valid YAML: …`

The parse itself failed; the trailing message is PyYAML's own, and its most common shapes are worth translating:

- `mapping values are not allowed here` — usually a colon-plus-space inside an unquoted value (`title: see: here`). Quote the value.
- `found character '\t' that cannot start any token` — a tab character; YAML indentation is spaces only.
- `duplicate key …` — see the quoting family above.

### `frontmatter must be a YAML mapping`

The block parsed, but not to `field: value` pairs — typically a stray leading `-` turned it into a list.

### `missing or non-string 'id' in frontmatter`

Every record carries its identity in the `id` field — the filename is not enough (filenames are a naming convention; the frontmatter is the identity). Add `id: <PREFIX-NNN>`.

### `duplicate id 'X' (also in <file>)`

Two files claim the same ID. Usually a copy-paste record creation that kept the original's `id`. Decide which file is the real `X`; allocate a fresh ID for the other — the `new <type>` command prints the next free one.

### `filename should be the id 'X', optionally followed by '-' and a handle (e.g. X.md, X-a-handle.md)`

Under the default filename policy, a record's filename starts with its ID; a freely chosen handle may follow after a `-`. The file at hand is named some other way. Rename the file — or, if this project wants full naming freedom, declare `filename: free` in the schema (the ID then lives only in the frontmatter).

### `frontmatter entity 'x' does not match directory-implied entity 'y'`

The record declares itself one type but sits in another type's directory. One of the two is wrong; the fix is whichever matches your intent — move the file or correct the `entity:` field.

## Types and values

### `expected <type>, got <something> …`

The value's YAML type does not match the schema's declaration. The message often carries its own diagnosis:

- `(YAML parses yes/no/true/false as boolean — quote it if you meant text)` — `title: yes` stored a boolean, not the word. Write `title: "yes"`.
- `(YAML parsed this as a date — quote it if you meant text)` — the mirror case: an unquoted `2026-07-12` in a *string* field became a date object. Quote it.
- `expected date (YYYY-MM-DD, unquoted), got str …` — and the mirror of the mirror: a *quoted* date in a `date` field is a string. Date fields want the bare `2026-07-12`.
- `expected integer, got str '7'` — quoted numbers are strings; unquote, or change the schema if it really is a label.

### `required field 'x' is missing or null`

The schema promises every record of this type carries `x`; this one doesn't (or carries it empty — `x:` with nothing after it is null). Fill it from what you know. If nobody knows the value, that is not a formatting problem — see [the contract policy](#when-red-is-the-honest-answer) below.

### `unknown field 'x' not declared in schema '<type>'`

The record carries a field the schema never declared. In order of likelihood: a typo in the field name (fix the spelling); a field that belongs in the prose body, not the frontmatter (move it); or the schema is genuinely missing a field the project now needs (evolve the schema — that is a normal workflow, recorded in `schemas/CHANGELOG.md`). One more possibility worth knowing after the quoting family: an unquoted comma one line up can *manufacture* a phantom field — if the "unknown field" is a fragment of the previous line's sentence, the real error is there, and the pre-parse scan usually names it first.

### `value 'x' not in enum [a, b, c]`

The field only admits the listed values, and `x` is not one of them. Usually a spelling or casing variant of a legal value (`Open` for `open`); occasionally a genuinely new state, which is a schema evolution, not a typo — add it to `values` and log the change.

### `value 'x' does not match pattern '…'` / `value N below min M` / `value N above max M`

The schema constrains the field's shape or range and the value falls outside. The pattern is a full-match regular expression: the whole value must match, not a fragment.

## References

The registry's strongest guarantee: every reference resolves to a real record of the right type. These errors are the guarantee working.

### `reference 'X' does not resolve to any entity`

A frontmatter field points at an ID that does not exist. Either the ID is mistyped (check the digits — `PROP-001` vs `PROP-011`), or the target was never created, or it lives in a file the engine cannot see (wrong directory, bad filename — in which case that file has its own error explaining why).

### `reference 'X' points at a 'y', expected 'z'`

The ID exists but is the wrong kind of thing — the field is declared to reference a `z`. Either the ID is wrong, or the field's declaration is.

### `inline reference ^[X] does not resolve to any entity` / `uses unknown prefix 'P'`

Same as above, but for references written in prose as `[label](path/to/X.md)^[X](path/to/X.md)`. An unknown prefix usually means a typo in the ID's letters, or a reference to a type this project does not track.

### `inline reference ^[X] has two different destinations ('a' and 'b') — both links must point at the file of 'X'`

The visible link and the `^[…]` annotation disagree about where the record lives. Usually one of them survived a file move that the other didn't. Point both at the target's actual file.

### `inline reference ^[X] links to 'p', which does not exist relative to this record's own directory` / `… which is not the file of 'X' — expected 'q'`

Prose links resolve relative to the file that contains them — a link copied from a record in another directory keeps the wrong relative path. The second form of the message does the arithmetic for you: `expected 'q'` is the correct path, ready to paste.

### A reference you only *mention*

One honest limitation, recorded as ISSUE-005 in the project's own registry: inline code spans are not exempt from reference checking, so writing *about* a reference in backticks still validates it as one. Until that is resolved, lift example references into fenced code blocks, which are exempt.

## Aggregate constraints

### `constraint 'unique' on [f1, f2] violated: same values as X`

Two records carry identical values in a combination the schema declares must be unique. The message names the other record; decide which one is the truth and change or retire the other.

### `constraint 'max_count_per g' violated: 'v' has N records (max M): ID, ID, …`

A cap declared in the schema ("at most 3 per student") is exceeded for group `v`, and the message lists exactly which records make up the excess. The fix is domain-sense, not formatting: something must move, merge, or be retired — or the cap itself is stale and the schema should evolve.

## Placement

### `file is not inside any declared entity directory (known: …)`

A `.md` file sits under the entities tree where no schema claims it — a stray. Move it into its type's directory, or if it is not a record at all, out of the entities tree. Two placement rules worth knowing while you decide: locations are claimed per type and must not nest, and a file named exactly like its own directory (`proposal/proposal.md`) is treated as an Obsidian folder note and silently skipped — a record must not be named that way (the project's registry tracks this sharp edge as ISSUE-007).

### `schemas directory not found` / `entities directory not found`

The engine found a project root but not the expected layout. Usually the run is in the wrong directory — pass `--root`, or check `entity-manager.yaml`'s `schemas_dir`/`entities_dir` if the project relocated them.

## When red is the honest answer

Sometimes a record violates the schema and no available truth can fix it: a required field nobody knows the value of, a reference to something that was never registered. That is not a formatting problem, and the engine will not be talked out of it — what happens next is the project's *contract policy*, declared in `entity-manager.yaml` (`policy.on_unresolvable`). Under `block`, the red stays and is itself the deliverable: an honest list of what cannot be managed without a human decision. Under `relax-and-report`, the contract may be loosened to keep work flowing, but every loosening is logged in `schemas/CHANGELOG.md`. What is never acceptable, under either policy, is inventing a value to turn red into green — a fabricated green costs more than any red, because it spends the trust every other green depends on.

And if you believe the engine itself is wrong — the flag looks like a false positive, the message misnames your case — that is worth reporting, not working around: this skill's own repository tracks its defects as `ISSUE-*` records, and more than one rule in this catalog exists because someone reported exactly that.
