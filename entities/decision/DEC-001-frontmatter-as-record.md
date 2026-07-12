---
id: DEC-001
entity: decision
title: One file per entity — YAML frontmatter is the record, prose is the body
status: accepted
date: 2026-07-12
addresses: [REQ-002, REQ-003]
tags: [architecture]
---

Each entity lives in exactly one Markdown file. The YAML frontmatter is the typed record (validated against the entity's schema); the Markdown body is free prose for everything a schema cannot capture. This keeps documents fully accessible to humans ([[REQ-002]]) while giving agents a deterministic place to read and write structured facts.

Alternatives considered: a central index file (single point of merge conflicts, prose divorced from data) and sidecar data files (two files per entity drift apart). Rejected both.
