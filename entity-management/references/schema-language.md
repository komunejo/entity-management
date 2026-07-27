# Schema language specification

Schemas are YAML declarations, one file per entity type, living in `schemas/<type>.yaml` (`.yml` is also accepted). The engine (`entity_lint.py`) meta-validates the schemas themselves before validating any record, so a typo in a schema is caught the same way a typo in a record is — including keys that do not apply to a field's type (a `pattern` on an integer, a `max` on an enum), which are rejected rather than silently ignored.

## Contents

- [Project configuration](#project-configuration)
- [Schema file](#schema-file)
- [Field types](#field-types)
- [Aggregate constraints](#aggregate-constraints)
- [Record file format](#record-file-format)
- [Inline references in prose](#inline-references-in-prose)
- [YAML pitfalls the engine catches](#yaml-pitfalls-the-engine-catches)
- [Design guidance](#design-guidance)

## Project configuration

`entity-manager.yaml` at the project root, optional. Defaults:

```yaml
schemas_dir: schemas
entities_dir: entities
policy:
  on_unresolvable: block    # block | relax-and-report
```

`policy.on_unresolvable` is the project's contract policy, chosen explicitly by the user when the first schema is created (see [`SKILL.md`](../SKILL.md), "When the contract cannot be satisfied"). `block` means the contract is untouchable and unresolvable records are surfaced as an honest red; `relax-and-report` allows the agent to loosen the contract, always documented in `schemas/CHANGELOG.md` and reported. When the key is absent, behave as `block`. The engine validates the key's value against exactly those two modes, and `policy`'s key names against its documented namespace — a typo in either, value or key, must never silently behave as `block` (DEC-023). Schema changes are never silent, under either policy.

**One silence to know about**, recorded as ISSUE-016 in the skill's own registry and not yet fixed: the *set* of top-level keys is not validated. A misspelled `schemas_dir:` is neither read nor reported — the engine falls back to the documented default, and if that default folder exists beside the one the project meant to declare, validation comes back green having measured the other one. Since you are usually the one writing this file, check these three key names by eye whenever you create or edit it, and treat a green that arrives right after a layout change as worth one look at this file before you trust it.

The engine finds the project root by walking upward from the current directory until it sees `entity-manager.yaml` or a `schemas/` directory; `--root` overrides.

## Schema file

```yaml
entity: decision            # must match the filename (decision.yaml)
description: >              # optional, for humans and agents
  What this entity type represents.
id:
  prefix: DEC               # unique across ALL schemas in the project
  width: 3                  # MINIMUM digits, zero-padded: DEC-001 (default 3); more digits are accepted as the registry grows past 999
dir: decision               # subdirectory under entities/ (default: entity name; must be unique across schemas)
# path: registry/decision   # optional: project-root-relative home for this type's records,
                            # overriding entities/<dir>. No '..'; locations must be unique
                            # across schemas and must not nest inside one another.
strict: true                # unknown frontmatter fields are errors (default true); with strict: false they are warnings, which never affect the exit code
fields:
  title:    {type: string, required: true}
  status:   {type: enum, values: [proposed, accepted, superseded], required: true}
  date:     {type: date, required: true}
  supersedes: {type: ref, entity: decision}
  addresses:  {type: list, items: {type: ref, entity: requirement}}
  weight:     {type: number, min: 0, max: 1}
```

`id` and `entity` are reserved record fields added automatically — do not declare them in `fields`.

## Field types

| type | accepts | extra keys |
|---|---|---|
| `string` / `text` | YAML string | `pattern` (full-match regex; also read by the pre-parse scan to tell a trailing comment from value content — see YAML pitfalls) |
| `integer` | integer (not boolean) | `min`, `max` |
| `number` | integer or float | `min`, `max` |
| `boolean` | true / false | |
| `date` | unquoted `YYYY-MM-DD` (a YAML date, not a string) | |
| `datetime` | unquoted ISO 8601 date or datetime | |
| `enum` | one of `values` | `values` (required, non-empty list) |
| `ref` | the ID of an existing entity of type `entity` | `entity` (required) |
| `list` | YAML list; every item validated against `items` | `items` (required field spec, may itself be a `ref`) |

Every field also accepts `required: true` (default false), `description`, and `default` (documentation only — the engine does not inject defaults; absent optional fields are simply absent). `required` — like the schema-level `strict` — must be a real YAML boolean: any other value would be read as merely truthy, a decision nobody made, and is rejected as dead configuration (DEC-023).

Refs are checked at two levels: the schema's `entity` target must be a declared entity type (meta-validation), and each record's value must resolve to an existing ID of exactly that type.

## Aggregate constraints

Field specs constrain one record at a time. Some rules live *between* records ("a piece can only be submitted once", "at most 3 submissions per student") — declare those in an optional `constraints` list on the schema, and the engine enforces them project-wide on every `validate` run:

```yaml
constraints:
  - rule: unique
    fields: [piece]
    description: a piece can only be submitted once
  - rule: max_count_per
    group_by: piece.student
    max: 3
    description: at most 3 pieces per student; participation is voluntary
```

`unique` requires the combination of the listed field values to be unique among this entity's records (records where any listed field is absent are skipped, so optional fields work naturally); the reserved field `id` may be included in the combination. `max_count_per` groups this entity's records by the value of `group_by` and errors when a group exceeds `max`; `group_by` may be a dot-path that hops through `ref` fields (`piece.student` = "the student of the referenced piece"), and records whose path does not resolve are skipped — absence never violates an aggregate constraint, which is what makes voluntary participation modelable. Both rules accept an optional `description` that is echoed in the error message.

Constraint declarations are meta-validated like everything else (unknown rules, undeclared fields, paths through non-ref fields are reported), and an invalid constraint is pruned with an error rather than disabling the whole schema. The vocabulary is deliberately small and implemented once in the engine — if a rule cannot be expressed with it, document the rule in the schema `description` and handle it as soft integrity (assisted review), never with a per-project bespoke validator.

## Record file format

`entities/<dir>/<ID>.md` — or `<path>/<ID>.md` when the schema declares `path`. The rule is only that the filename starts with the ID; anything after it is an optional handle (`<ID>-any-handle.md`), chosen by whoever names the file and derived from nothing. `new` suggests the bare ID.

```markdown
---
id: DEC-001
entity: decision
title: One file per entity
status: accepted
date: 2026-07-12
addresses: [REQ-002, REQ-003]
---

Free prose. Everything the schema cannot capture lives here, including
inline links to other entities:
[the requirement (REQ-002)](../requirement/REQ-002.md)
```

Rules enforced by the engine: frontmatter parses and is a mapping; `id` is present, unique project-wide, matches `<prefix>-<at least width digits>` and the filename; `entity` matches the schema implied by the directory; required fields present; all values type-check; refs resolve to the right type; inline references in prose resolve, and their destinations point at the record their ID names; undeclared fields are errors in `strict` schemas and warnings otherwise.

**Folder notes are not records.** A Markdown file named exactly like its own directory (`entities/entities.md`, `registry/skill/skill.md`) is treated as an Obsidian folder note and skipped by record discovery — the natural home for the generated index in vault projects. The exemption is exact-name-only; any other `.md` outside a declared entity location still errors.

## Inline references in prose

Two forms are validated. The default is one ordinary Markdown link whose text carries the ID — free label with the ID in parentheses, or the bare ID when there is no label:

```
[some label (REQ-004)](../requirement/REQ-004.md)
[REQ-004](../requirement/REQ-004.md)
```

The ID-shaped link text under a declared prefix is what marks the link as a reference — a link whose text carries no such ID is an ordinary link and is never touched, so a project can link to a README or to the web without the engine having an opinion (an ID under a prefix no schema declares is likewise left alone). What is checked: the ID resolves, and the destination resolves — relative to the referring record's own directory — to that ID's file. The label is not checked; it is yours, except that a label which merely repeats its own ID (`[REQ-004 (REQ-004)](…)`) is flagged as redundant — write the bare form instead. The pre-DEC-022 caret annotation (`^[ID](path)` riding behind a link) is recognized only to be named as an error: Pandoc reads `^[…]` as an inline footnote and the construct shatters in any footnote-aware renderer (ISSUE-012).

The reason this is the default rather than a wiki-style link is that a wikilink is not Markdown. It renders as literal bracket characters wherever the project is published, so a reference written that way is validated and unreachable at the same time. A destination containing spaces or parentheses must be wrapped in angle brackets — `[label (ID)](<a path with spaces (and parens).md>)`. Two different costs, worth keeping apart. Unwrapped **spaces** validate fine but many renderers stop at the first one and publish the link as literal text: checked and unreachable. Unwrapped **parentheses** are worse in the other direction — the engine does not recognize the construct as a reference at all, so it is neither resolved nor reported (ISSUE-017; the restriction is stricter than CommonMark, which allows balanced parens bare, and lifting it belongs with the reference-grammar work of ISSUE-014). Angle-bracket both and neither cost applies; `md_link_dest` in the engine does exactly that for every destination it writes.

`[[DEC-001]]` and `[[DEC-001|display text]]` remain valid and are checked the same way for the ID, for projects whose readers understand them. A project that prefers them uses them; nothing else changes.

Fenced code blocks (``` … ```) are skipped for both forms, so examples never trigger false errors.

## YAML pitfalls the engine catches

These are the reasons "YAML + Markdown" needs a linter at all:

- `title: yes` → YAML parses a **boolean**, not the word "yes". The engine reports the type mismatch with a hint to quote the value.
- `date: 2026-07-12` unquoted is a date; `date: "2026-07-12"` quoted is a string. `date`-typed fields require the real date, so a quoted one errors.
- `version: 1.0` is a float — declare `string` and quote it if you mean a version label.
- Indentation errors and tab characters produce YAML parse errors, reported per file rather than crashing the run.

A nastier family parses *cleanly* into something the author never typed — the misread destroys its own evidence, so the engine hunts these on the raw text, before the parse (ISSUE-003 in the skill's own registry records the measurements; PROP-006 the design). The writer-side rule that avoids all of them: **when a value holds prose, quote it.** Every message these checks can raise is explained with examples, for humans, in the skill's repository (github.com/komunejo/entity-management) under `docs/`; the agent's triage discipline is [`troubleshooting.md`](troubleshooting.md).

- An unquoted comma inside a flow collection splits the scalar: `{description: applies always, required: true}` is **two keys** — the description truncated and `required` set, silently if the tail spells a legal key. The engine asks for quotes on any multi-word unquoted scalar inside `{...}` or `[...]`.
- A `#` preceded by whitespace in an unquoted value starts a comment: `title: count # per item` parses to `count` — the tail vanishes with no error. Quote the value to keep it (or to make an intended comment unambiguous). The flag fires only where the engine cannot decide for itself (DEC-023): on fields typed `integer`, `number`, `boolean`, `date`, `datetime`, `enum` or `ref`, the tail could never be part of a legal value (`count: 3 # per item` is fine — and a truncation that damages the value leaves an illegal remainder the type check catches), so annotate typed values freely. A `string` field with a declared `pattern` is decided the same way, against the shape: `code: P1 # urgente` under `pattern: "P\d+"` is a value plus a comment, no flag.
- Text glued to the `#` is the treacherous cousin: `tag: #urgent` reads as a *hashtag* to a human and as a comment to YAML — the field parses to null and the text is gone, and no schema declaration can change that: YAML discards it at parse, before the engine looks. Because the null escapes every downstream check, this is the one comment shape flagged on **every** field kind, typed included. Quoting is YAML's own mechanism for such values (`tag: "#urgent"`). A spaced comment after an empty value (`tag: # to fill in`, `tag: ## section`) is the ordinary idiom and stays silent.
- The same key twice in one mapping keeps the **last** value and discards the rest without a word. Never admissible; the engine rejects it with the line number.
- A line reading `---` inside a multi-line frontmatter value would end the frontmatter early if taken as a delimiter. The engine only accepts a closing delimiter at column zero, so an indented `---` stays what it is: a line of your value.

## Design guidance

Model only the facts that must not drift. A field earns its place in the schema when (a) more than one document depends on it, (b) an agent might plausibly get it wrong from memory, or (c) you want to query or index by it. Everything else is prose — that is a feature, not a failure of modeling: the prose is the interface, the schema is the safety net. Prefer `enum` over free `string` wherever the vocabulary is closed; enums are where drift dies. Where a string field's values have a shape (codes, slugs, hashtags), declare it with `pattern`: the engine then validates the shape *and* uses it to read the field's comments correctly — an undeclared shape is knowledge the engine cannot use, and a permissive one (`.*`) is no knowledge at all: the declaration's power is the shape's precision. Prefer one `ref` field per relation over encoding relations in prose alone; you can (and should) still mention the relation in prose with an inline reference, which keeps human readability and gets validated too.
