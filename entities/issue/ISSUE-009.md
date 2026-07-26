---
id: ISSUE-009
entity: issue
title: container directories cannot carry their own typed record
status: open
date: 2026-07-25
channel: use (comroom, library-shelves session)
tags: [schema-language, locations, folder-notes]
---

Two engine rules, each correct alone, combine to make one real-world shape unrepresentable: a directory that contains records of one type and should itself be described by a record of another.

The rules:

1. **Folder notes are never records.** The loader exempts `<dir>/<dir>.md` from validation by design (they are Obsidian folder notes — typically the generated index). A record placed as its own directory's folder note therefore exists outside the contract: present, published, never validated.
2. **Entity locations must be unique and must not nest.** A container type cannot declare its records inside (or at the root of) the tree a member type already claims — the loader would claim the container's records for the member type, and the location check rejects the overlap before that.

Where it bit: comroom's library reorganized its `source` records into physical shelves — `library/shelves/<shelf>/SRC-NNN.md`, one directory per shelf, sources inside. The shelf itself deserves a typed record (id, name, what the shelf is for), and the natural place for that record is the shelf directory it describes. No legal placement exists: as `<shelf>/<shelf>.md` it is exempt from validation (rule 1); as a record of a `shelf` type declaring the same tree it collides with the `source` location (rule 2). The workaround — shelf records exiled to `entities/shelf/`, physically apart from the directories they describe — works, but splits a single thing (the shelf) into a room and a card that drift independently.

The shape of the gap: containment is an ordinary relation between entity types (a shelf contains sources; a room contains transcripts), and the schema language cannot express it. The exemption in rule 1 excludes exactly the file that would represent the container.

Candidate fix sketched in [folder-note records](../proposal/PROP-005.md)^[PROP-005](../proposal/PROP-005.md).
