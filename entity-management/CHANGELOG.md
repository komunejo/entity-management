# entity-management skill changelog

The engine and skill evolve by versioned maintenance, never by per-project regeneration (DEC-002). Every engine or skill change gets an entry here, mirroring the discipline that schemas/CHANGELOG.md imposes on schemas (REQ-006).

## 0.7.0 — 2026-07-26

- Engine, breaking: the prose reference form is now one Markdown link whose text carries the ID — `[label (ID)](path)`, bare `[ID](path)` when unlabeled. The 0.5.0 caret construct (`[label](path)^[ID](path)`) imitated a footnote mark no dialect owns; Pandoc reads `^[…]` as an inline footnote and the construct shatters in any footnote-aware renderer (ISSUE-012, DEC-022 correcting DEC-017). The legacy caret is recognized outside fenced code only to be named as an error, so it cannot creep back. An ID-shaped link text under a prefix no schema declares stays an ordinary link; a label that merely repeats its own ID is flagged as redundant (the bare form is what it collapses to).
- Engine: `new` stubs hint the corrected reference form.
- Engine fix: the reference pattern now reads angle-bracketed destinations containing parentheses — `[label (ID)](<a name (draft).md>)` — the very form `index --write` emits for free filenames. Before, such a link matched nothing and was silently not validated: a false green in every `filename: free` project whose names carry parentheses, latent since 0.5.0. Measured in a scratch project with prose filenames (spaces, commas, parentheses), varied labels (accents, internal parentheses, cross-type), and Pandoc HTML/docx transformation of the results.
- Docs: the reference form updated in SKILL.md, the schema language reference, and `docs/troubleshooting.md`, which gains entries for the redundant-label and legacy-caret errors and loses the two-destinations one (the corrected form has a single destination).
- Registry: all records migrated in one pass — 34 files, with the 7 label-equals-ID references collapsed to the bare form; fenced historical specimens left as written.

## 0.6.1 — 2026-07-26

- Engine: the missing-PyYAML message no longer prescribes `pip install pyyaml` — the command half the current platforms reject, aimed at an actor (the agent) that neither has nor should have the permissions it assumes. It now addresses both possible readers in plain words: the agent (obtain the library as ordinary environment work — per-user, only if missing, never elevated) and the human (your assistant can arrange it; official page as the only pointer). The Python floor (≥ 3.8) is checked before any import and fails as a sentence, not a stack trace (ISSUE-002, PROP-008, DEC-021).
- Docs: SKILL.md gains a Runtime section addressed to the agent — verify the interpreter is real before invoking it (Windows Store decoy, macOS Command Line Tools dialog), obtain PyYAML per the machine's own conventions (a platform-provided agent environment, e.g. Claude Cowork's, satisfies the runtime inside itself — the platform's documentation, referenced not replicated, states what it provides), and only when operating directly on a host with no Python ask the human for exactly one thing in plain words. `references/troubleshooting.md` files exit-2 runtime failures as a triage family owned by the agent (missing PyYAML is silent housekeeping; the no-Python machine is the protocol's one plain ask). `references/hooks.md`: hooks must invoke the interpreter that satisfies the runtime, not a bare `python3`. README states the two requirements once, gently, with pointers to the official sources; `docs/troubleshooting.md` opens with the two pre-engine failure messages, quoted as the engine emits them, explained for humans.

## 0.6.0 — 2026-07-26

- Engine: pre-parse raw-text scan over every YAML the engine reads (config, schema files, record frontmatter), run before PyYAML — catching the family of misparses that destroy their own evidence: a multi-word unquoted scalar inside a flow collection (an unnoticed comma splits it into extra keys, silently when the tail spells a legal one), and an unquoted value containing ` #` (the tail vanishes as a comment). Scan findings replace the downstream symptom, so the author reads about the comma they typed, not the phantom key they didn't (ISSUE-003, PROP-006).
- Engine: duplicate keys in any mapping are rejected with the line number, at any depth — PyYAML's default keeps the last value and silently discards the rest (PROP-006: never admissible).
- Engine fix: the frontmatter closing delimiter is only accepted at column zero. An indented `---` inside a multi-line value was previously taken as the close, truncating the frontmatter before the parse — fields below the cut fell silently into the document body (ISSUE-003's fourth member).
- Engine fix: `new` emits the title double-quoted — the engine's own emissions meet the scan's rules (an unquoted title carrying ` #` or a comma was born misparsing).
- Hooks: session-start validation added as a firing point (SessionStart hook), making flag ownership trivial — from a green start, every later flag belongs to the session's own work. Pre-commit reframed as the layer covering hand edits made with no agent at the keyboard.
- Docs: `references/troubleshooting.md` (new) — the agent's flag-triage discipline: claim, resolve with criterion, escalate at most one intent question; criteria the human's answers establish are recorded so no question is asked twice. The human-facing companion, `docs/troubleshooting.md` in the repository, explains every engine message with examples (repo documentation, not packaged).
- Docs: the YAML-pitfalls list gains the silent-misparse family and the writer-side rule that avoids it — when a value holds prose, quote it — in the schema language reference, SKILL.md's record-creation and bulk-import workflows, and the golden rule (which now also carries the escalation protocol in brief).

## 0.5.0 — 2026-07-15

- Engine: prose references in the form `[label](path)^[ID](path)` are validated — the ID resolves, the two destinations agree, and they point at that ID's file relative to the record making the reference. The label is never read; it is free, and a stale one is soft integrity (DEC-017). A Markdown link without the `^[ID](path)` annotation is an ordinary link and is untouched. `[[ID]]` remains valid and is still checked for its ID.
- Engine: optional per-schema `filename` — `prefixed` (the default, `<ID>.md` or `<ID>-<handle>.md`) or `free` (unconstrained; the id lives in the frontmatter alone). A project whose records are named in prose declares `free` and keeps its names (DEC-018).
- Engine fix: the `prefixed` check tested `startswith`, so `PROP-0011.md` passed as a name for `PROP-001`. It now requires the ID exactly, or the ID followed by `-`.
- Engine fix: `index --write` wraps destinations containing whitespace or parentheses in angle brackets. Unescaped, they rendered as plain text — a generated index of dead links, which `free` filenames would have hit immediately.
- Engine: `new` suggests `<ID>.md` and no longer slugifies the title into a filename. A handle is chosen, never derived (DEC-018).
- Docs: the Markdown reference form documented as the default in SKILL.md and the schema language reference; the filename described as an ID plus an optional handle; the rename workflow no longer offers to leave a slug stale.

## 0.4.1 — 2026-07-15

- Engine fix (DEC-014): `index --write` no longer forces the interpreter's platform line ending on the file it writes. It now preserves the line endings of an existing index and defaults to LF for a new one; the endings are detected from the raw bytes, since universal-newline translation on read would hide them. On Windows the previous behaviour rewrote every line of an LF index as CRLF, so each regeneration produced a whole-file diff and silently converted the file. `index --write` is the engine's only file write; `new` prints to stdout and is unaffected. Validation, schema handling and the CLI surface are untouched.

## 0.4.0 — 2026-07-12

- Engine: `index --write` computes every link relative to the index file's own location (stdout falls back to the project root), so the index works at any path the project prefers (DEC-012).
- Engine: folder notes are never records — a Markdown file named exactly like its own directory (`entities/entities.md`, `registry/skill/skill.md`) is skipped by record discovery, letting the generated index live as the registry's folder note in vault projects; the exemption is exact-name-only and stray detection is otherwise unchanged (DEC-012).
- Engine: optional per-schema `path` (project-root-relative, no `..`, unique, non-nesting) places a type's records anywhere in the project, overriding `<entities_dir>/<dir>`; record discovery scans declared locations, `new` suggests the declared home, and the default layout needs no declaration (DEC-013).
- Docs: `path`, the folder-note exemption, and free index placement documented in the schema language reference and SKILL.md.

## 0.3.0 — 2026-07-12

- SKILL.md description compacted to the packager's 1024-char limit; trigger performance re-measured intact (26/26 on the bilingual eval set).
- Engine hardening after adversarial review (12 reproduced defects fixed, zero remaining crashes across the review fixture suite): UTF-8 BOM tolerated everywhere (`utf-8-sig`); non-UTF-8 files, invalid `pattern` regexes, non-numeric `min`/`max`, non-string config dirs and non-string `dir` now report errors instead of crashing; aggregate constraints no longer crash on list/dict values; duplicate `dir` across schemas rejected; `index` escapes `|` in titles and creates parent directories on `--write`; path filters resolve against the project root and a filter matching no record exits 2 instead of producing a false green.
- Stricter meta-validation: field keys that do not apply to the declared type (a `pattern` on an integer) are rejected rather than silently ignored.
- Docs: PostToolUse hook snippet corrected — feedback reaches the agent only via stderr + exit 2, the previously documented form was invisible to it; `--json` payload shape and exit codes (0/1/2) documented; `width` documented as minimum zero-padding; warnings semantics for non-strict schemas documented.
- SKILL.md: three workflows the description promised but the body never taught, now taught — bulk-create from CSV/spreadsheet rows, index/summary/CSV export generation, and computed reports/queries over the records.

## 0.2.1 — 2026-07-12

- SKILL.md description replaced after measured trigger optimization: 26/26 on a 26-query bilingual eval set (15 positive / 11 adversarial negative), vs 17/26 for the original. Leads with the registry-of-typed-records condition, enumerates lifecycle workflows including bulk CSV import, exports, and computed reports/queries over records, and carries explicit non-triggers (SQL/ORM, JSON Schema, blog frontmatter rendering, one-off prose).

## 0.2.0 — 2026-07-12

- Engine: aggregate constraints in the schema language — `unique` and `max_count_per` (with dot-paths through refs), enforced project-wide on every validate; meta-validated and pruned-with-error when misdeclared (DEC-009).
- Engine: `engine_version` included in `--json` output.
- SKILL.md: per-project contract policy `policy.on_unresolvable` (block | relax-and-report), asked explicitly at first-schema creation, default block (DEC-007); mixed rule defining "unresolvable" under block (DEC-008); schema changes always logged in schemas/CHANGELOG.md, never silent (REQ-006).

## 0.1.0 — 2026-07-12

- Initial release: generic validation engine (types, required, enums, unique prefixed IDs, typed refs, inline `[[ID]]` refs outside code fences), `validate` / `index` / `new` commands, schema language specification, hooks reference, workflows for init, record creation, migration, rename and soft-integrity review.
