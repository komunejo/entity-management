---
id: PROP-005
entity: proposal
title: "folder-note records: typed records for container directories"
status: proposed
date: 2026-07-25
addresses: [REQ-001]
tags: [schema-language, locations, folder-notes]
---

## Motivation

[Container directories cannot carry their own typed record (ISSUE-009)](../issue/ISSUE-009.md): the folder-note exemption keeps the container's natural record out of the contract, and the location-uniqueness rule keeps a container type out of its members' tree. A domain whose entities *are* containers (shelves holding sources, rooms holding transcripts) is therefore not fully definable, which is what [schemas must be fully definable per project (REQ-001)](../requirement/REQ-001.md) promises. The engine already accepts that a record's filename is a naming concern, not an identity concern (0.5.0 separated ID, filename, and title); this proposal extends the same move to *placement*: being a directory's folder note is a placement, and placement should not decide whether a record is validated.

## Sketch

- Optional schema-level key, working name `folder_note: true`, valid only together with `path`. It declares: the records of this type are the folder notes `<path>/<dir>/<dir>.md` of the subdirectories of `path`. The id lives in frontmatter only (as with `filename: free`); the directory name is the filename's counterpart — a label, not the identity.
- **Loader**: for a folder-note type, claim exactly the `<dir>/<dir>.md` files under its `path`; the general folder-note exemption stays for every other schema (their generated indexes and room notes remain non-records).
- **Location check**: a folder-note type and at most one ordinary type may declare the same `path` — the sanctioned container/member pair. The member's loader already skips folder notes, so each file has exactly one claimant; the uniqueness rule stays intact for every other overlap.
- **Stray detection**: unchanged in principle — every `.md` under a shared tree is either the member type's record, the container type's folder-note record, or a stray.
- **`new <type>`**: for a folder-note type, ask for (or derive from the title) the directory label and suggest `<path>/<label>/<label>.md`, creating the directory alongside.
- Default when the key is absent: current behavior, fully backward compatible.

## Open questions

- Depth: immediate subdirectories of `path` only (the shelf case), or any depth?
- May a folder-note type exist with no member type sharing the tree (a registry of rooms with free-form contents)?
- The same file is the engine's record and the publish layer's room door (Vitrine appends generated contents to folder notes). Is that double duty a feature — one file, both faces — or does it want a boundary?
