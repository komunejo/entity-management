# References between records

This is the registry's strongest guarantee: every reference resolves to a real record, of the right type, at the right file. When one of these messages fires, the guarantee is doing its work — something now points at nothing, and you are hearing about it while it is still cheap to fix.

## What makes a link a reference

Worth settling before the messages, because it is the single thing adopters most often get wrong: **the engine does not check every link in your prose. It checks the ones that carry an ID.**

A reference is written as an ordinary Markdown link whose *visible text* carries the ID of the target:

```markdown
[the reference form is one link carrying the ID (DEC-022)](../decision/DEC-022.md)
[DEC-022](../decision/DEC-022.md)
```

The first is the labeled form — a free label, then the ID in parentheses at the end. The second is what it collapses to when the label would just be the ID. In both, the ID's prefix must belong to a declared type; a link whose text carries no such ID is an ordinary link and none of the engine's business.

The reason the ID sits in the *visible text* rather than only in the destination is that a reader reads the text. A link whose ID hides in the path tells the human nothing and breaks the moment the file is renamed.

Records can also refer to each other through frontmatter fields declared as `ref`, which hold the bare ID:

```yaml
supersedes: DEC-017
```

Those are checked too, and they produce the first two messages below.

**Wikilinks.** The engine also recognizes `[[DEC-022]]` and `[[DEC-022|any alias]]`, the native Obsidian form. One known limitation, reported from the field and recorded as ISSUE-014: a wikilink whose destination is a *filename* rather than an ID — `[[some-note-title]]`, the ordinary way of linking in a vault that names files freely — is not seen as a reference at all. It is not reported as broken; it is simply invisible to the engine. If your vault's connective tissue is written that way, you are not getting reference validation on it yet, and knowing that is better than assuming a green covers it.

### Writing the destination when the filename has spaces or parentheses

The ID sits in parentheses at the end of the visible text, and it can follow a label that has parentheses of its own — `[a note (a draft, really) (DEC-022)](../decision/DEC-022.md)` reads correctly, because the last parenthesized group is the one taken as the ID. The destination is where care is needed:

```markdown
[label (DEC-004)](<some note (draft).md>)
[label (DEC-004)](<a name with spaces.md>)
```

**A destination holding parentheses must be wrapped in angle brackets, or the engine does not see the reference at all** — not reported broken, simply invisible, which is the worse of the two. That is a limitation on this side rather than a rule about how you may name files: parentheses in filenames are ordinary, and the Markdown standard allows them unwrapped when they are balanced. It is recorded against the project as ISSUE-017, and until it is lifted the angle brackets are the whole remedy.

A destination holding **spaces** is validated either way, so no green is lying to you there. Wrap it anyway: unwrapped, many renderers stop reading the destination at the first space and publish the link as literal text — checked and unreachable at the same time. Angle brackets cost nothing and cover both cases, which is why every destination the engine writes for you already has them.

## `reference 'X' does not resolve to any entity`

A frontmatter field points at an ID that does not exist. Three causes, in order of frequency:

1. **A mistyped ID.** Check the digits: `PROP-001` and `PROP-011` are one keystroke apart and both look right.
2. **The target was never created.** The reference was written in anticipation. Either create the record or remove the pointer.
3. **The target exists but the engine cannot see it.** It sits outside every declared location, or its own file has an error that stopped it loading. In that case that file has its own message somewhere in the same run, explaining why — fix that one first and this may resolve itself.

## `reference 'X' points at a 'y', expected 'z'`

The ID exists, but it names the wrong kind of thing: the field is declared to reference a `z` and `X` is a `y`.

Either you meant a different record, or the field's declaration in the schema is too narrow for what this project actually needs. Both are ordinary; only you can say which.

## `inline reference [[X]] uses unknown prefix 'P'`

A wikilink has the shape of a reference, but no type in this project declares that prefix. Usually a typo in the *letters* — `DEV-004` for `DEC-004` — or a reference copied in from a project whose types are not yours.

This fires for wikilinks only, and the asymmetry is worth knowing. In a **Markdown link**, ID-shaped text under an undeclared prefix is treated as an ordinary link and passes in silence — the engine cannot tell your `[SKU-100](…)` product code from a reference to a type you forgot to declare. So in prose links a typo in the digits goes red, while a typo in the letters goes quiet. If a reference you were sure about produces no message at all, that silence is the thing to check.

## `inline reference (X) does not resolve to any entity`

The same as the frontmatter case above, for a reference written in prose. Same three causes.

## `inline reference (X) links to 'p', which does not exist relative to this record's own directory`

The ID is fine; the path is not. Prose links resolve **relative to the file that contains them**, which is what usually goes wrong: a link copied from a record in another folder keeps that record's relative path, and from here it points at nothing.

Count the hops from this file's own folder. From `entities/decision/DEC-004.md`, a sibling is `DEC-005.md` and a record of another type is `../issue/ISSUE-003.md`.

## `inline reference (X) links to 'p', which is not the file of 'X' — expected 'q'`

The path exists and the ID exists, but they are not the same record — the link goes somewhere real and wrong, which is worse than going nowhere, because nothing looks broken to a reader.

The message does the arithmetic for you: `expected 'q'` is the correct path, ready to paste.

## `inline reference (X) repeats its own ID as the label — redundant; write [X](path) alone`

`[DEC-004 (DEC-004)](path)` says the same thing twice. The bare form is what it collapses to. Delete the label half.

## `inline reference ^[X] uses the legacy caret annotation, which Pandoc reads as an inline footnote (ISSUE-012) — write [label (X)](path), or [X](path) when unlabeled`

An older form of this project's reference grammar, from before version 0.7, wrote the ID as a caret annotation after the link: `[label](path)^[DEC-004](path)`.

It imitated a footnote mark that no Markdown dialect actually owns, and it broke where it mattered: Pandoc reads `^[…]` as an inline footnote, so the construct shattered in every footnote-aware renderer. It was retired for that reason and is recognized now only to be named.

Rewrite it as `[label (X)](path)`, or `[X](path)` where the label was only ever the ID.

## When you want to *mention* a reference without making one

One honest limitation, recorded as ISSUE-005 in the project's own registry: fenced code blocks are exempt from reference checking, but **inline code spans are not**. So writing *about* a reference between backticks, mid-sentence, still validates it as a live one.

Until that is resolved, lift example references into a fenced block:

````markdown
```
[label (DEC-999)](path/to/nowhere.md)
```
````

That is a real defect, not a convention to learn. It is written down as one.
