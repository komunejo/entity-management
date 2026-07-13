---
id: PROP-004
entity: proposal
title: "Polymorphic refs: a field that references one of several entity types"
status: proposed
date: 2026-07-13
addresses: [REQ-004]
tags: [schema-language, refs]
---

## Motivation

A synchronized-repos registry (2026-07-13) models `copy` — a local copy, on a concrete machine, of a synchronized artifact that is either a `repo` or a `vault`. The natural declaration is one field referencing *an artifact*; the schema language only supports refs typed to exactly one entity. The workaround is two optional fields (`repo`, `vault`) plus a prose rule "exactly one must be set" — checked by a reconciliation pass, i.e. by discipline, which is what [[REQ-004]] says integrity must not depend on. The same shape will recur wherever a family of sibling types shares a relation (any "artifact contract" implementer, cf. [[PROP-001]]).

## Sketch

Two orthogonal pieces, useful separately and complete together:

1. **Multi-target ref**: allow `entity` to be a list — `{type: ref, entity: [repo, vault]}`. A value must resolve to an existing ID of *any* listed type. Meta-validation: every listed type must be declared; ID prefixes make the resolution unambiguous.
2. **Exactly-one aggregate rule**: a per-record constraint `{rule: exactly_one, fields: [repo, vault]}` erroring when zero or more than one of the listed fields is present. Also expressible today as nothing at all — hence the proposal.

With (1) the two-field workaround collapses into one field; (2) alone would at least harden the current workaround.

## Open questions

- Does `unique` over a multi-target ref field need per-type semantics (one copy per repo per machine) or is whole-value uniqueness enough?
- Interaction with [[PROP-001]]: if abstract parents exist, `entity: artifact` (the parent) may be sugar for the list of its children.
