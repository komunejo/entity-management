---
id: PROP-009
entity: proposal
title: "nested metadata: one legitimate client (the compound value), two impostors (embedded entities, interaction history)"
status: proposed
date: 2026-07-27
addresses: [REQ-002]
tags: [schema-language, frontmatter, composition, layers]
---

## Motivation

The question arrived through [the memory-store report asks the engine to validate a format whose contract nobody has written (ISSUE-015)](../issue/ISSUE-015.md) — a foreign dialect that nests fields under a `metadata:` wrapper — and the first analysis asked whether the schema language should learn nesting to accommodate it. Working the cases turned up a question worth settling on its own terms, independent of any one dialect: when is a nested mapping in frontmatter ever the right design? The answer decides whether the schema language wants a mapping type at all, and for exactly what.

## The discriminator: identity

Whether nesting is right is decided by whether the nested thing has identity — an ID, a lifecycle, the possibility of being referenced. An entity composed of other entities (X composed of Y and Z) never nests their metadata: composition is expressed by `ref` fields carrying IDs, Y's metadata lives in Y's record and nowhere else, and following the reference is what [entities are referenced by stable IDs with a unique prefix per type (DEC-004)](../decision/DEC-004.md) exists to make cheap. Nesting one entity's metadata inside another's is duplication — two sources of truth, diverging from the first edit. This is the first impostor: what looks like structure is denormalization.

## The one legitimate client: the compound value

A value with internal structure and no identity — `geo: {lat: 40.4, lon: -3.7}` — is one value with parts. Nesting there is semantics, not taste: it tells the schema what flattened names cannot — that `lat` without `lon` is invalid, that the parts are required jointly, that "coordinate" is a declarable, reusable type. `geo_lat` and `geo_lon` at the top level push that unity into a naming convention the schema cannot see or enforce. This is the single case a mapping type in the schema language would serve, and it is a real one.

The cost must be stated with it: nested mappings are not editable in Obsidian's Properties — the second tier of the project's support criterion — so a project declaring compound values chooses raw-text editing for those fields. [documents must remain human-readable and human-editable (REQ-002)](../requirement/REQ-002.md) is the requirement this trade touches; raw legibility survives (a two-key indented block reads fine), the Properties panel does not.

## The second impostor: interaction history

Keys that record who touched the document, when, and from where (`modified`, `originSessionId` and kin) are not metadata of the content — they say nothing about what the document affirms. They are events of its handling: another layer. Nearly every current model misfiles that layer into frontmatter because it has never been properly described anywhere; the memory-store envelope ISSUE-015 examines is the confusion made syntax — a harness with no versioned layer beneath it, quarantining its bookkeeping in a bag inside the document because it has nowhere else to put it. In this project's world the layer exists and is git: mechanical history lives in commits. What a record legitimately carries is its *semantic* history — a supersession notice, a correction that names its error ([a record that was wrong is corrected in place, and the correction names the error (DEC-019)](../decision/DEC-019.md)) — because that is part of what the record means, curated as content. The rule this proposal fixes: interaction events never enter a document's metadata; where a source dialect embeds them, stripping them is the conversion layer's job, not the schema's to describe.

## Sketch

- Doctrine, documentable now, no engine change: entities compose by reference; interaction events stay out of frontmatter; nesting is reserved to compound values.
- If and when compound values are wanted: a `mapping` field type declaring sub-fields with their own types and joint requiredness, validated as one value — opt-in per field, with the Properties-editability trade named in the docs where it is declared.
- Out of scope: accommodating foreign envelopes (`metadata:` wrappers and their kind) — that is conversion-layer work, and it waits on ISSUE-015's pending questions.
