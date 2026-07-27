---
id: PROP-011
entity: proposal
title: "the documentation states what adopting costs: use cases and first steps, for humans and agents"
status: proposed
date: 2026-07-27
addresses: [REQ-001, REQ-003]
tags: [documentation, adoption, use-cases, onboarding]
---

## Motivation

A capable field adopter ran this week's most instructive failure: staged a raw corpus as a project, counted 171 errors, and read the count as a verdict on adoption ([the memory-store report asks the engine to validate a format whose contract nobody has written (ISSUE-015)](../issue/ISSUE-015.md)). Nothing the engine lacked caused the misreading. What was missing was stated doctrine: **entity-manager demands schema-design work, exactly as any database engine does.** Adopting is not preserving what exists — it is deciding whether a corpus is worth migrating to a structured format designed for *that concrete case*, and doing the design. Errors thrown by an unprepared corpus measure the corpus's unpreparedness, not the method.

Half of the doctrine is already in the README's first line — "database-engine-like integrity" — but the parallel is stated for *integrity*, not for *cost*: a reader learns what the engine guarantees, not what adoption asks of them. The other half — the schema is declared by the owner, never deduced from the data — lives only inside ISSUE-015, an incident record. Doctrine that lives in incident records is doctrine the next adopter rediscovers by failing.

The documentation today bears this out: `docs/` holds troubleshooting — what to do when something went red — and nothing that comes *before*: no first steps, no statement of use cases, no account of the adoption decision itself.

## Sketch

- **README**: one explicit statement beside the existing integrity line — the parallel completed: like a database engine, the method requires designing the schema for your case; adoption is a migration decision, not a preservation default.
- **`docs/`: a new first-steps file** — not under troubleshooting, before it. What adopting means; the decision it opens with (is this corpus worth structuring, and what structure does *this case* need); the two openings (a project born structured vs. an existing corpus converted — layout freedom, the conversion layer, ID assignment); and the use cases clarified, including the negative ones: a corpus nobody will reference or validate has no reason to adopt, and free files are a designed feature, not a gap.
- **The agent-facing layer says the same thing**: SKILL.md and `references/` teach the operating agent to guide an owner through the adoption decision — including telling an owner *not* to adopt, or to adopt partially — rather than assuming every corpus in reach is a candidate registry. One doctrine, two audiences, phrased for each.
- Out of scope: writing the documents now (they land when the proposal is taken up, as one coherent pass), and any engine change — this proposal touches only what the project says about itself.
