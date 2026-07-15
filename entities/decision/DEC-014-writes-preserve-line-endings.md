---
id: DEC-014
entity: decision
title: Engine writes preserve a file's line endings, and default to LF
status: accepted
date: 2026-07-15
addresses: [REQ-002]
tags: [io, engine, bug]
---

`index --write` — the engine's only file write — wrote through Python's default newline translation, so on Windows every regenerated index came out CRLF regardless of what it had been. `detect_newline()` now reads the target's **bytes** to see what it actually uses, and the write goes through `open(..., newline=...)`: an existing index keeps its endings, a new one gets LF. Mixed endings resolve to LF.

Bytes rather than text on the read is the whole trick: universal-newline translation erases the thing being measured, so a text-mode read cannot detect what a text-mode write is about to destroy.

Motivation: the conversion churned every line of the file. Content identical, diff total — which defeats review at exactly the point where the index is supposed to make the registry legible ([[REQ-002]]), and buries a real change among hundreds of fake ones. It was also silent: no error, no warning, invisible in an editor, and visible only in `git diff` or a byte count. Measured before and after on a real project — a committed 68-line LF index regenerated as 68 CRLF lines by the old engine, and byte-identical by the new one.

LF as the default for new files is the convention in the projects this engine serves, verified rather than assumed: across three of them, 300/300 LF, 162 LF to 6 CRLF, and 292 LF to 8 CRLF.

Scope: I/O only. Validation, schema handling and the CLI surface are untouched, per the skill's own rule that a bug in the validator is worse than a bug in the data.

Found by using the engine, not by reading it. A project built with it had its own prose rewritten to CRLF by a separate utility; fixing that utility exposed the same defect here.
