---
id: ISSUE-005
entity: issue
title: inline code spans are not exempt from ref resolution, so a ref cannot be mentioned mid-sentence
status: open
date: 2026-07-15
channel: use
tags: [engine, schema-language]
---

Fenced code blocks are skipped by inline-ref resolution, and `references/schema-language.md` says why: "so examples never trigger false errors". Inline code spans are not skipped. A wikilink wrapped in backticks, in the middle of a sentence, is still resolved as a live reference — so writing *about* a reference is indistinguishable, to the engine, from making one.

Found by walking into it. [a prose reference can be human-readable or validated, never both](ISSUE-004.md)^[ISSUE-004](ISSUE-004.md) is a record about how references behave; quoting two example links inside backticks made it fail validation with two errors, both correct and both about links that were never links. The only documented way out is to lift the example into a fenced block, which works — this record does it — but a fenced block is not something a sentence can contain. The escape exists for the case where you were going to break the flow anyway.

The exemption already encodes the right idea: marked-up code is not prose, and a link inside it is a specimen, not a pointer. It just stops at the block level, one character short of the span level, and the reason it stops there is not written down anywhere. Nothing about hard integrity is at stake either way — a mention that resolves is a false red, not a false green, and this project has been explicit that a false red is the safer failure.
