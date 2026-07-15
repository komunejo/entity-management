---
id: ISSUE-001
entity: issue
title: regenerating the index on Windows rewrote every line to CRLF
status: resolved
date: 2026-07-15
channel: use
resolved_by: [DEC-014]
tags: [io, engine, bug]
---

`index --write` returned an index whose content was identical and whose every line had changed. No error, no warning, nothing an editor shows — visible only in `git diff` or a byte count. A real change, when one came, would arrive buried in hundreds of false ones.

It surfaced in another project built with the engine, whose own prose had been rewritten to CRLF by a separate utility; fixing that utility exposed the same defect here. It arrived, like everything else this project knows about itself, through somebody using it.

Registered after the fact: when it was found there was no issue type to register it with, so it went from noticed to fixed in one step and only the fix was written down. That gap is what [issues are records, registered whatever channel they arrive by](../decision/DEC-015.md)^[DEC-015](../decision/DEC-015.md) exists to close, and this record is the first thing to fall through it.
