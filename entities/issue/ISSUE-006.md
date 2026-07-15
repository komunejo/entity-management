---
id: ISSUE-006
entity: issue
title: the references the engine validates are dead links in every reader
status: resolved
date: 2026-07-15
channel: use
resolved_by: [DEC-017]
tags: [references, portability]
---

Every inline reference in this registry passes validation and reaches nothing. Verified against this repository's own published page: GitHub serves DEC-014's body with the literal characters of a wikilink in it — no anchor, nowhere to go. Double-bracket links are not Markdown. They are an Obsidian and wiki extension, and GitHub renders repository Markdown, not wiki markup.

Obsidian does not save it either, and the reason is ours rather than Obsidian's. A record is named `<ID>-slug.md`, a link names the ID alone, and Obsidian resolves a wikilink by exact basename — `REQ-002-human-readable-documents.md` is not `REQ-002`. Either rule on its own would work: an exact `REQ-002.md` linked by its ID, or `provenance.md` linked by its name. The combination the engine mandates is precisely the one that resolves nowhere, and one half of it — the filename rule — is not written down as a decision anywhere.

The engine already knows how to emit a link that opens. `index --write` produces ordinary Markdown, with the path computed relative to the index's own location ([index links follow the index’s location; folder notes are never records](../decision/DEC-012.md)^[DEC-012](../decision/DEC-012.md)). In prose it validates a syntax it would never use itself.

What makes this sharp rather than cosmetic is that the validation is green. The reference resolves, the run passes, and the writer is told the link is sound — while every reader that exists gets inert text. A registry whose claim is that its records stay legible in any text editor ([documents must remain human-readable and human-editable](../requirement/REQ-002.md)^[REQ-002](../requirement/REQ-002.md)) has been shipping a link syntax that no reader it targets can follow.
