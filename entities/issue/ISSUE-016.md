---
id: ISSUE-016
entity: issue
title: "a misspelled top-level config key is ignored without a word, and the green that follows can have measured another corpus"
status: open
date: 2026-07-27
channel: self-inspection (0.7.1 documentation pass, while inventorying every message the engine can emit)
tags: [engine, config, integrity, flag-quality]
---

The engine validates the *values* of the two `entity-manager.yaml` keys it recognizes (`schemas_dir`, `entities_dir` must be non-empty strings) and, since [the scan speaks only where the schema leaves it honestly blind (DEC-023)](../decision/DEC-023.md), everything under `policy` — the namespace's key names as well as its one value. What nothing checks is the *set* of top-level keys. A key the engine does not recognize is not read and not reported: the lookup misses, the documented default is used, and the run continues as if the project had declared nothing.

## Measured, not deduced

A project whose config declares

```yaml
schema_dir: registro-schemas
entites_dir: registro
```

— two keys misspelled — and whose tree holds both the directories the owner meant to declare and the default ones, was validated against the defaults and reported `OK: 1 entities across 1 schemas — 0 error(s), 0 warning(s)`, exit 0. The records under `registro/` were never opened. The engine did not validate the corpus the file pointed at, and said everything was well.

This is the family of [a record named like its own directory disappears without a word (ISSUE-007)](ISSUE-007.md): a green whose count is true about something other than what the project declared.

## The two outcomes, and which one is dangerous

- The default directory does not exist → `schemas directory not found`. Red, but naming a path the project never mentioned: confusing, not dangerous, because the run stops.
- The default directory exists → green over the wrong corpus. This is the reportable defect, and it is not exotic: the two conditions meet exactly when a project moves its layout — renames `entities/`, relocates its schemas — and leaves the previous folders in the tree. That is also the moment someone is typing new keys into that file, which is when a key gets mistyped.

A second, milder shape of the same silence: a key the engine never had. Someone writes `filename: free` in the project config believing it sets the mode project-wide — it is a schema key, not a config key — and nothing says so. The owner believes a declaration was made; none was.

## Why this one is not a matter of taste

The config file is the single surface where the contract is the engine's own documentation: no schema describes it, so the engine is the only possible validator of it. Schema files already reject unknown keys (`unknown schema key '<key>'`), and `policy` keys were brought under the same rule by DEC-023 on the grounds that a typo'd contract must never quietly mean `block`. The top level of the config is what the sweep left behind — and the argument that governed the others governs it: everything a project declares, the engine enforces; it must not silence what it knows. The tie between a false red and a silent loss is already decided by the cost principle stated in that decision — a false red is paid once, a silent loss never announces itself — and here the loss does not merely go unannounced, it is announced as `OK`.

While the silence exists it is documented for humans as a known silence, in `docs/troubleshooting/config.md`. Documenting a silence is not registering it, which is why this record exists ([issues are records, registered whatever channel they arrive by (DEC-015)](../decision/DEC-015.md)).

## The shape of the fix, and its price

Validate the top-level key set against the documented names and red on the unknown, mirroring what schema files and `policy` already do. The price is a red on upgrade for any project that had stashed its own keys in that file — the same class of red 0.6.1 and 0.7.0 already produced, and one this project knows how to announce. The softer variant (a warning) is the wrong trade here: the damage in this defect is not that a line goes unread, it is that the closing summary says `OK`.

Engine work, so not part of the release that reports it.

## This repository, checked

Not affected today, and the check is worth recording because the reason is a coincidence rather than a mechanism. This project's config carries exactly the three recognized keys, all spelled correctly, and its declared values are identical to the defaults — so even an ignored key would fall back to the same directories. The green was verified independently of the engine: 54 `.md` files under `entities/`, 4 schemas in `schemas/`, against the engine's 54 entities across 4 schemas.

What stands between this registry and the dangerous outcome is therefore the coincidence above plus the protocol step that checks the count against expectation — that is, the discipline of whoever verifies, which is what [hard integrity must not depend on model discipline (REQ-004)](../requirement/REQ-004.md) says integrity may not rest on. The day this project relocates `entities/` or `schemas/`, it stops being protected by the coincidence and becomes the typical candidate.
