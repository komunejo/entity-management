---
id: ISSUE-010
entity: issue
title: folder notes as first-class typed records across a whole space
status: open
date: 2026-07-26
channel: use (keikoba space, constitution session)
tags: [schema-language, locations, folder-notes]
---

Second occurrence of the gap behind [container directories cannot carry their own typed record (ISSUE-009)](ISSUE-009.md), reported from a different shape of use — one that stresses all three open questions of [folder-note records (PROP-005)](../proposal/PROP-005.md).

## The case

The keikoba space adopted a convention: **every folder of Markdown content carries a folder note** (`<folder>.md`) as its quick access door — a short function description plus a contents-index table (ID where the content carries IDs, filename, minimal description), regenerated on every content change. The convention is registered in the space itself as a document type (a composition with two sections), so the *recipe* is under contract — but the folder notes themselves are not. The owner wants them typed: a `folder-note` (or `content-index`) record type where the filename convention `<folder>.md` is respected while the id lives in frontmatter — making explicit what these files are, and putting the space's actual doors under validation instead of leaving them the only untyped documents in it.

## What blocks it today

1. **The folder-note exemption**: the loader skips `<dir>/<dir>.md` unconditionally — the records this type wants are exactly the files the engine refuses to see (ISSUE-009's rule 1).
2. **Single-path claim**: a type's records are discovered only under its declared location, and locations must not overlap another type's tree. Folder notes live scattered — one per content folder, across trees other types already claim (ISSUE-009's rule 2).
3. Filename-starts-with-id is *not* a blocker if the type can declare its filename policy (the id/filename/title separation already exists) — noted so the fix is not over-scoped.

## The separation underneath

The id/filename/title separation cost real effort and settled that a record's name is not its identity. This issue asks for its counterpart in the placement dimension, and it is the crucial one: **type is not folder.** A record's type is what its frontmatter declares; the folder it sits in is an organizational choice. Today the loader conflates the two — placement is how records are discovered and claimed, so placement effectively decides what a record is (and whether it exists at all: the folder-note exemption is placement deciding *non-existence*). Folder notes are the shape of use that exposes the conflation, because their placement is dictated by what they describe — one per folder, everywhere — and no location-as-type scheme can claim them. Whatever form the fix takes, the test is that discovery follows the record's own declaration, not the tree it happens to stand in.

The declaration field already exists — `entity:` in every record's frontmatter — but today it is a checksum, not a declaration: the folder decides what a file is, and the field is merely required to agree. The fix is an inversion of authority: `entity:` decides; folder and id prefix remain as convention — useful redundancy whose drift is worth flagging, never the identity itself. The inversion buys a second capability almost for free: a file may declare a type **before any schema exists** for it. A declared-but-uncontracted type is an honest, recognizable state — the engine can report it as such (a visible inventory of candidate types emerging from use) instead of erroring, and the type graduates to a schema when its shape settles. That is how capture works in real notebooks: the kind is named at the moment of capture; the contract comes later.

## What this case adds to PROP-005's open questions

- **Depth**: immediate-subdirectories-only does not cover it. This space has folder notes at nested depths (`space/notebooks/<developer>/` — a folder note per developer notebook inside a folder that has its own). The need is recursive: wherever a content folder exists.
- **Member-less trees**: yes, required. The notebooks tree holds deliberately schema-less content (pre-structure thinking, no records at all), and its folders still carry folder notes that deserve the contract. A folder-note type must be able to claim folder notes in a tree no member type shares.
- **Whole-space scope**: the shelf case wants folder notes of one container's subdirectories; this case wants them across the entire space — which suggests the key's `path` should be able to scope wide (the space's content root) rather than one container tree.
- **Mixed-content folders**: the same conflation bites beyond folder notes. The space's notebooks tree hosts one strict type (`notebook-note`) and is meant to also host captures of kinds not yet defined — a related repository card, a source, whatever the moment of capture needs, each as its own future type. Today the loader claims *every* `.md` under the type's `path` for that type, so a strict type forces a homogeneous folder: the only ways out are loosening the type (wrong — a capture with other keys is another type, not a sloppy note) or keeping captures out of Markdown. A type should claim the files that declare it (`entity:` in frontmatter), not every file in the tree it stands in.

## Acceptance, from the reporting space's side

A schema can declare a type whose records are folder notes: named after their folder, id in frontmatter only, discovered recursively wherever the declared scope has content folders, validated like any record — while generated indexes outside the declared scope stay exempt as today.

One integrity property specific to this type, for the design to weigh: a folder note's reason to exist is its contents index — a table meant to mirror the folder it describes. Head validation alone leaves the index free to go stale, which in this document type is the failure that matters most (a stale quick-access door misleads more than a missing one). If the engine can check that the index's rows correspond to the folder's actual content — records and files present, nothing listed that is gone — then "derived view, never stale" stops being discipline and becomes contract. If that is out of the engine's scope, the issue asks the design to say so explicitly, so spaces know that freshness stays on the agents' side.
