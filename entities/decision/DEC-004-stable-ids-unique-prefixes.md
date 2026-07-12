---
id: DEC-004
entity: decision
title: Entities are referenced by stable IDs with a unique prefix per type
status: accepted
date: 2026-07-12
addresses: [REQ-004]
tags: [architecture, references]
---

References use IDs like `DEC-002`, never names or titles: renames are the great destroyer of coherence in Markdown projects. Each entity type declares a unique prefix, so any ID is self-describing about its type. Typed frontmatter refs (`supersedes: DEC-001`) and inline prose refs (`[[DEC-001]]`) are both validated by the engine — see [[DEC-002]].
