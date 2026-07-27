---
id: PROP-010
entity: proposal
title: "interactions are entities in their own right, not marginal data in another record's metadata"
status: proposed
date: 2026-07-27
addresses: [REQ-004]
tags: [interactions, modeling, layers, integrity]
---

## Motivation

Sister of [nested metadata: one legitimate client (the compound value), two impostors (embedded entities, interaction history) (PROP-009)](PROP-009.md), and its missing half: what that proposal expels from frontmatter — the events of a document's handling — this one houses. The expulsion is incomplete otherwise; data denied a wrong home and given no right one creeps back.

PROP-009's own identity test decides the question. An interaction has a when, actors, an object, a kind — and, decisively, the vocation of being referenced: "arrived by this letter," "the trial that revealed the mix," "the session that produced this correction." What can be referenced has identity; what has identity is an entity and composes by reference, never by embedding.

The motivating specimen is domestic. [ISSUE-013](../issue/ISSUE-013.md), [ISSUE-014](../issue/ISSUE-014.md) and [ISSUE-015](../issue/ISSUE-015.md) each carry, as free prose in their `channel` string, a description of the same letter — one interaction described three times, invisible to the engine, impossible to reference, drifting apart the first time one copy is edited. The duplication PROP-009 names, already installed in this registry: an interaction with a claim to be a record, flattened into marginal data in three other records' metadata. [issues are records, registered whatever channel they arrive by (DEC-015)](../decision/DEC-015.md) made records out of what the channels deliver; this proposal is the complementary move — the channel event itself as a record.

## The pattern

An `interaction` entity type, declared by a project like any other type: records carrying a date, the actors, a kind, and `ref`s to the records the event touched or produced. Provenance in other records then becomes one labeled reference instead of repeated prose, and the engine validates it like any reference. Nothing in the engine is missing for this — a project can declare the type today; what this proposal fixes is the doctrine (this is the right home for interaction data) and the pattern's shape, so projects do not each reinvent it. The openness DEC-015 built into `channel` — deliberately a string, "the channels are not a closed set" — survives as the openness of the interaction's kind field: the event becomes referenceable without the vocabulary closing.

## The boundary with git

Not a second changelog. In a versioned repository, mechanical file history — who edited which line, when — belongs to git, and duplicating it as records would be PROP-009's impostor pattern run in reverse. The interaction registry models the *semantically meaningful* events: a letter, a working session, a trial run, a review — events that may touch many files or none, that records need to cite, and that exist in worlds with no version control underneath (the memory store [ISSUE-015](../issue/ISSUE-015.md) examines has none; its harness smuggles interaction keys into frontmatter precisely for lack of this layer). The rule of thumb: if a record wants to reference the event, it is an interaction entity; if only the file's history wants it, it is git's.

## Sketch

- Doctrine: interaction events excluded from content metadata (PROP-009) live as records of a declared `interaction` type wherever the project needs to cite them.
- A minimal schema pattern documented with the doctrine: `date`, actors, `kind` (open string, per DEC-015's own reasoning), refs to affected records; ID prefix the project's choice (CORR, INT, RUN).
- This registry's own adoption — collapsing the repeated channel prose of ISSUE-013/014/015 into one correspondence record the three reference — is the candidate first case, decided if and when the proposal is taken up, not now.
- Out of scope: engine changes, and the general theory of the interaction layer — why nearly every current model misfiles it — which has its own life outside this registry.
