---
id: REQ-005
entity: requirement
title: Schema evolution is a first-class workflow
status: open
priority: must
date: 2026-07-12
tags: [core, evolution]
---

Schemas are always evolving — that rigidity is the classic weakness of SQL. After every (re)design phase, existing records must be migrated to the new schema and re-verified. Evolving a schema must be cheap: edit a declaration, migrate the affected records, re-run the validator until green.
