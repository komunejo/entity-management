---
id: DEC-002
entity: decision
title: The validation engine is generic and stable; schemas are data
status: accepted
date: 2026-07-12
addresses: [REQ-001, REQ-004, REQ-005]
tags: [architecture, integrity]
---

Three layers with different life cycles: the **engine** (`entity_lint.py`) is domain-agnostic infrastructure, written once and never touched by schema evolution; **schemas** are YAML declarations — redesigning means editing a declaration, not rewriting a linter; **records** are what gets migrated when schemas change.

Rationale: if the agent regenerated a bespoke linter on every redesign, each regeneration would be an opportunity for silent bugs — and a bug in the validator is worse than a bug in the data, because it corrupts trust in the whole system. A fixed, tested engine makes schema evolution cheap ([[REQ-005]]) and keeps hard integrity deterministic ([[REQ-004]]).
