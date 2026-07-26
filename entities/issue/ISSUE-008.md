---
id: ISSUE-008
entity: issue
title: the generated index cannot leave a type out
status: open
date: 2026-07-23
channel: use (comroom, correspondence-as-records session)
tags: [index, privacy, publishing]
---

`cmd_index` walks every schema and lists every record. There is no way to declare that a type belongs in the registry — validated, referenced, contract-bound — but not in the generated index.

Where it bit: comroom formalized its inter-estate correspondence as a `letter` type (`path: project/correspondence`, `filename: free`). The letters are deliberately excluded from the published website by the publish layer (`.vitrineignore`), but the entity index is itself a published page — so the moment the type validated, the index leaked the correspondence: titles listed, links pointing at paths the site does not serve. The registry did its job and the index undid it.

The shape of the gap: a type's *validation* and its *exhibition* are two different concerns, and the index currently couples them. Candidate fix, for whoever picks this up: a schema-level key (working name `index: false`) so a type's records stay fully validated but out of `cmd_index` output — the same family as [configurable display field for index generation (PROP-002)](../proposal/PROP-002.md), which already accepts that index generation needs to become declarable per project.

Until then, any space that registers a private or unpublished type must choose between an index that leaks it and no regenerated index at all.
