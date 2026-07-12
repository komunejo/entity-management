---
id: DEC-011
entity: decision
title: Convention for external references that cross the project boundary
status: proposed
date: 2026-07-12
addresses: [REQ-001, REQ-004]
tags: [schema-language, references, backlog]
---

The ceramics evaluation exposed the case: a sale links to a client note in ANOTHER project, where refs cannot resolve. Both evaluation agents improvised reasonable but divergent conventions (a free string field; a `clients:<path>` prefix). Proposal: a dedicated `external` field type in the schema language — validated only by `pattern`, explicitly excluded from ref resolution, and rendered by documentation as "outside the integrity boundary" — so the honest limit is declared in the schema instead of re-invented per project.

Alternative if both projects adopt entity-management: a cross-project resolution mode. Deliberately deferred until a second real multi-project case exists.
