---
id: ISSUE-017
entity: issue
title: "a prose reference whose destination filename holds parentheses is invisible, not broken"
status: open
date: 2026-07-27
channel: self-inspection (0.7.1 verification protocol, negative-testing the free-filename fixtures)
tags: [engine, references, integrity, filenames]
---

A prose reference written with a bare destination whose filename contains parentheses — `[label (THG-001)](nota (borrador).md)` — is not recognized as a reference at all. It is not reported broken and not resolved: it does not exist for the engine. Sibling of [the wikilink reference carries its ID in the destination, not in the text the reader sees (ISSUE-014)](ISSUE-014.md): both are links a human reads as references and the engine never sees, so both turn a green into a statement about less than the reader believes.

## Measured

A `filename: free` project, one target record, one referring record, the reference deliberately carrying a nonexistent ID so that being *seen* is provable:

| destination as written | engine |
|---|---|
| `sencillo.md` | reported |
| `con%20espacios.md` | reported |
| `con espacios.md` (raw spaces) | reported |
| `con,coma.md` | reported |
| `acentuaci%C3%B3n%20%C3%B1.md` | reported |
| `con(parentesis).md` | **silent** |
| `con%20(parentesis).md` | **silent** |
| `<con (parentesis).md>` | reported |

So the trigger is the parenthesis alone, in the destination alone. Spaces are validated bare (they may still render badly, which is a separate matter); the angle-bracketed form always works.

**Both filename modes are exposed.** `prefixed` is not immune: a record filed as `THG-001 (borrador).md` — the ID plus a parenthesized handle, which [a filename is an ID and an optional handle; the name is the title (DEC-018)](../decision/DEC-018.md) permits — is equally invisible as a destination. The exposure is a property of the filename, not of the naming mode.

**What it cannot do.** The record itself is discovered and validated normally; no record goes missing and no count is wrong. And the engine never writes such a link: `md_link_dest` angle-brackets any destination holding a space or a parenthesis, so generated views are safe — the index emits `[entities/thing/nota (borrador).md](<entities/thing/nota (borrador).md>)`. What is exposed is references written by a human or by another tool, which is exactly the corpus ISSUE-014 reports: a vault of prose filenames whose connective tissue was hand-written.

## Provenance, since a defect introduced by the release that reports it would be a different matter

Not introduced in 0.7.1. The engine of every published version was run against the failing case: v0.3.0, v0.4.0, v0.4.1, v0.5.0, v0.6.0, v0.6.1 and v0.7.0 all stay silent. Reading the source rather than only the result:

- v0.3.0 and v0.4.x had no Markdown-link reference form at all — only `[[ID]]` wikilinks — so their silence is absence of the feature, not this defect.
- v0.5.0 introduced the prose form and with it the destination pattern `[^()\n]*`, character for character what stands today. That is where this begins.
- 0.7.0 rewrote the form under [a prose reference is one link whose text carries the ID (DEC-022)](../decision/DEC-022.md) and carried the same class forward, but *added* the angle-bracketed alternative. Before 0.7.0 a reference to a parenthesized filename could not be written at all; since 0.7.0 it can, in one spelling. The release improved this without closing it.

## The owner's reading, checked

The owner's instinct on being shown the defect: the check should work *backwards* from the end of the link mark, so that in `[nombre friki de fichero (con paréntesis) (ID)](enlace)` what is matched is the trailing `(ID)]`.

For the label that is already the behavior, and it was worth confirming rather than assuming: `REF_LABELED_RE` matches its label greedily, so the last parenthesized group wins and a label full of parentheses is read whole. Measured — `[nombre friki de fichero (con paréntesis) (THG-001)](sencillo.md)` validates green, and the same line with a nonexistent ID reports. The label is not where the gap is.

For the destination the equivalent of "backwards" is not a greedy label but paren balancing, which is what CommonMark specifies: a bare destination may contain parentheses as long as they are balanced, and anything else must be angle-bracketed. The engine is therefore stricter than the standard it targets, on a rationale that does not survive the distinction: `md_link_dest`'s docstring justifies the restriction with "the link silently degrades to literal text", which is true of spaces and not of balanced parentheses.

## Why the fix is not a patch, and not this release

The candidate fix touches the reference grammar — the one surface where this project has already paid for moving fast. [the reference annotation imitates a footnote mark, and footnote-aware renderers take it at its word (ISSUE-012)](ISSUE-012.md) is the precedent: a form the engine validated and a renderer destroyed. Accepting balanced parentheses bare must therefore be measured against the transformation targets before it is designed, not after, and that measurement (Pandoc's wikilink extensions, mistletoe, Markpub) is already the pending groundwork for the reference-grammar work ISSUE-014 governs. This belongs there, with its migration announced, not in a patch.

Until then the mitigation is cheap and ships with this release: the skill instructs the operating agent to keep parentheses out of filenames *it* chooses and to angle-bracket any destination that has them — bounded so that it never becomes a licence to rename a project's own files, which would be treating a corpus as the engine's servant rather than the reverse.

The rest of the debt is documentary, and it is this release's rather than a future one: the human layer never states the angle-bracket rule, and the agent layer states it for *spaces* and for a rendering consequence — the wrong character and the wrong risk. Worse, `docs/schemas.md` told the reader that under `filename: free` "the engine has no opinion about names at all", which this measurement makes false in the one mode where names are prose.
