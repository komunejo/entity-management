---
id: PROP-001
entity: proposal
title: "Schema inheritance: abstract parent schemas enforced by the engine"
status: proposed
date: 2026-07-13
addresses: [REQ-004]
tags: [schema-language, inheritance]
---

## Motivation

A real project (a synchronized-repos registry, 2026-07-13) needed an abstract parent type: `artifact`, whose fields (`title`, `host`, `has_local`, `path`, `is_archived`, `last_synced`) must be declared identically by every concrete artifact type (`repo`, `vault`, and whatever comes later). The schema language has no inheritance, so the parent had to be written down as a documented convention (`schemas/ARTIFACT.md` in that project) that child schemas mirror by hand.

That works, but the guarantee is soft: whether a child schema actually conforms to the contract is checked by review, not by the engine. This is exactly the kind of integrity [[REQ-004]] says should not depend on model discipline.

## Sketch

- An abstract schema declares `abstract: true` and defines fields as usual; it has no records of its own (no `id.prefix` needed, or a prefix that no record may use).
- A concrete schema declares `extends: <abstract-name>` and inherits the parent's fields; it may add fields but never redefine an inherited field with a different type or requiredness (meta-validation error).
- Stays fully declarative — schemas remain data, per [[DEC-002]]. No new engine behavior at record-validation time beyond the merged field set.

## Open questions

- Multiple inheritance: probably reject; one parent is enough for the known cases.
- Should the parent be able to declare aggregate constraints that apply per-child or across all children? (The `unique path` rule in the motivating project is per-child today.)
