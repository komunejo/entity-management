---
id: DEC-005
entity: decision
title: Validation fires via Claude Code hooks and explicit workflow steps
status: accepted
date: 2026-07-12
addresses: [REQ-004]
tags: [hooks, workflow]
---

A linter that exists but is never fired is documentation, not integrity. Two firing points: a Claude Code `PostToolUse` hook runs the validator whenever the agent writes or edits a Markdown file inside an entity-managed project, and the skill's workflows instruct the agent to validate at defined moments (after creating records, after migrations, on request — "check the structural consistency of the documents"). Git pre-commit hooks remain a documented option for teams that want them, but are not installed by default.
