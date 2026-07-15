---
id: DEC-015
entity: decision
title: Issues are records, registered whatever channel they arrive by
status: accepted
date: 2026-07-15
tags: [backlog, governance]
---

An `issue` type joins requirement, decision and proposal in this project's own registry: what the project is told about itself, recorded like everything else and validated by the same engine.

The tracker on GitHub has been open since the repository was published and has never held an entry. Not for want of material — every proposal in the backlog came out of one real project, and the line-endings defect fixed in 0.4.1 came out of another. Everything this project knows about itself arrived through somebody using it, and none of it arrived through GitHub. A tracker only ever holds what walks in through its own door; GitHub carries one channel, and issues arrive by many — use, conversation, a report by mail, an agent running into the engine inside some other project.

So the register cannot be the tracker. It is the repository, where the engine validates it and the index lists it, and GitHub's tracker becomes one channel feeding it rather than the place issues live. Whoever receives an issue, by whatever channel, is the one who registers it.

The cost of not having had this is already on the record. The 0.4.1 defect went from noticed to fixed in a single step, because there was no issue type to open an issue with; the fix exists as [[DEC-014]] and the problem it fixed existed nowhere until [[ISSUE-001]] registered it after the fact. A defect with no entry cannot be found twice, cannot be pointed at, and cannot be answered with "already known" — and the project that argues structured facts should not be left loose in prose had let its own most recent one do exactly that.

`channel` is a free string and not an enum, against the schema language's own guidance to close a vocabulary wherever it can be closed. The guidance is right and does not apply: the claim this decision rests on is that the channels are *not* a closed set, and an enum would quietly re-impose the assumption the register exists to refuse.
