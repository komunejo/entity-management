---
id: ISSUE-007
entity: issue
title: a record named like its own directory disappears without a word
status: open
date: 2026-07-15
channel: use
tags: [engine, integrity]
---

Measured on the real engine: a file at `entities/proposal/proposal.md`, with an `id`, an `entity`, a title and everything the schema asks for, is skipped by record discovery and the run reports `OK: 0 entities across 1 schemas — 0 error(s), 0 warning(s)`. The record does not exist as far as the registry is concerned, and nothing says so.

The exemption is [index links follow the index’s location; folder notes are never records](../decision/DEC-012.md)^[DEC-012](../decision/DEC-012.md), and it is right about the case it was written for: in a vault, a Markdown file named exactly like its directory is an Obsidian folder note, not a record. But the test is the filename alone, so it cannot tell a folder note from a record that happens to be called that — and a folder note has no `id`, which is a difference the engine can see and does not look at.

It bites hardest where the filename is the project's to choose. A store whose records are named in prose is exactly the kind that will call something `proposal.md` inside `proposal/`, and the failure it gets is silence: not a red, not a warning, an absence. A green run over a record nobody can find is the shape [contract changes are never silent](../requirement/REQ-006.md)^[REQ-006](../requirement/REQ-006.md) warns about — trust in the whole result, spent on a count that is wrong.
