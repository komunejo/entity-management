---
id: PROP-002
entity: proposal
title: configurable display field for index generation
status: proposed
date: 2026-07-13
addresses: [REQ-002]
tags: [index, schema-language]
---

## Motivation

The generated index (`entity_lint.py index`) labels each row with `title`, falling back to `name`. A real project (a synchronized-repos registry, 2026-07-13) concluded that its entities do not carry titles at all — hosts, identities, repos and people are known by a *display name*, and the semantic difference matters to its owner — so every schema renamed the field to `display_name`. The engine-generated index now renders an empty label column for all 52 entities: the registry got semantically more truthful and visually less readable at the same time. Readability of the generated views should not depend on entities calling their name field `title` ([documents must remain human-readable and human-editable](../requirement/REQ-002.md)^[REQ-002](../requirement/REQ-002.md)).

## Sketch

- Optional schema-level key, e.g. `display: display_name`, naming the field used to label records in generated output (index today, any future generated view).
- Default when absent: current behavior (`title`, then `name`) — fully backward compatible.
- Meta-validated: the declared field must exist in the schema and be a string.

## Open questions

- Should the fallback chain be declarable as a list (`display: [display_name, title, name]`)?
