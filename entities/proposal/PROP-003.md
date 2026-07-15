---
id: PROP-003
entity: proposal
title: "generated views inside records: reverse-reference tables"
status: proposed
date: 2026-07-13
addresses: [REQ-004]
tags: [refs, generated-views]
---

## Motivation

Composite entities want their record to show what points at them. In a synchronized-repos registry (2026-07-13), a `project` record should include the table of artifacts whose `projects` field references it — automatically updated — alongside the hand-written prose that explains the project. Today the only options are a hand-maintained table (which drifts, exactly what [hard integrity must not depend on model discipline](../requirement/REQ-004.md)^[REQ-004](../requirement/REQ-004.md) says must not depend on discipline) or asking readers to run a query themselves. The engine already knows every inbound reference at validation time; it just has no way to materialize that knowledge inside a record.

## Sketch

- A marked block inside the record body (delimiters in comments, e.g. `<!-- generated:backrefs -->` ... `<!-- /generated -->`) that a new engine command regenerates deterministically, the same way `index` regenerates INDEX.md. Prose outside the block is never touched.
- Per-schema declaration of which inbound refs to tabulate, e.g. `views: {backrefs: {from: repo.projects, columns: [id, display]}}`.
- Regeneration is idempotent and safe to run from hooks after validate passes.

## Open questions

- One command (`index`) growing a `--views` mode, or a dedicated `views` subcommand?
- Should the block content count as record prose for inline-ref validation? (Probably skip it, like fenced code.)
