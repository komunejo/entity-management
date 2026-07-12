---
id: DEC-012
entity: decision
title: Index links follow the index’s location; folder notes are never records
status: accepted
date: 2026-07-12
addresses: [REQ-002]
---

Where derived views live is each project owner’s choice, never the engine’s imposition. `index --write` always accepted any path, but links were computed relative to the project root, so an index anywhere but the root shipped broken links — now every link is computed relative to the index file itself (stdout output falls back to the root).

Companion rule: a Markdown file named exactly like its own directory (`entities/entities.md`, `registry/skill/skill.md`) is an Obsidian folder note, and record discovery skips it — which lets the generated index live as the registry’s folder note without tripping stray-file detection. The exemption is deliberately narrow (exact name match only); any other stray `.md` still errors.
