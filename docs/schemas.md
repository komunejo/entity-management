# Designing schemas

A schema is the rulebook for one type of record: what fields it has, what kinds of value they hold, and what must be true across all records of that type. You write one file per type, in YAML, and the engine checks your records against it.

This page is the whole vocabulary, explained for a person who wants to write their own. It is not the normative text — that is [`schema-language.md`](../entity-management/references/schema-language.md), which travels inside the skill and is what the engine is held to. Where the two ever disagree, the specification is right and this page has a bug. Read that one when you need the letter of something; read this one to understand what you are choosing between.

If you have not written a first schema yet, [the walkthrough](quick-guide.md) gets you to one in about fifteen minutes, and you can come back here for the second.

## The shape of a schema file

It lives at `schemas/<type>.yaml` — the filename must match the type it declares — and every key below except `entity`, `id` and `fields` is optional.

```yaml
entity: decision            # the type's name; matches the filename
description: >              # what this type is, for whoever reads the schema later
  A decision taken, and the reasoning that produced it.
id:
  prefix: DEC               # IDs look like DEC-001; unique across the whole project
  width: 3                  # minimum digits, zero-padded
dir: decision               # where its records live, under entities/
strict: true                # an undeclared field is an error, not a warning
fields:
  title:      {type: string, required: true}
  status:     {type: enum, required: true, values: [proposed, accepted, superseded]}
  date:       {type: date, required: true}
  supersedes: {type: ref, entity: decision}
  addresses:  {type: list, items: {type: ref, entity: requirement}}
constraints:
  - rule: unique
    fields: [title]
```

`id` and `entity` are added to every record automatically. Do not declare them as fields; the engine will tell you they are reserved.

## Identity: `id`

Every record carries an ID, and the ID is what everything else points at. Two keys shape it:

- **`prefix`** — letters and digits, starting with a letter (`DEC`, `ISSUE`, `REQ2`). It must be unique across every type in the project, because references are resolved by prefix: `DEC-004` has to name exactly one kind of thing. It is deliberately plain ASCII — IDs travel through filenames, URLs and other people's tools — and a prefix with an accent fails loudly rather than being quietly mangled. Everything else in your registry takes any character you write in.
- **`width`** — how many digits, zero-padded, and it is a **minimum** rather than a cap. With `width: 3` you get `DEC-001`, and when the registry passes 999 it simply continues as `DEC-1000`. Nothing has to be migrated.

The reason identity lives here rather than in the filename is that filenames get renamed, copied and dragged between folders, and renames are the great destroyer of coherence in a folder of documents. An ID does not change.

## Where the records live: `dir`, `path`, `filename`

The engine finds records by location: every `.md` file inside a type's folder is taken to be a record of that type.

- **`dir`** — the folder under `entities/`. Defaults to the type's own name, which is usually what you want.
- **`path`** — for a type whose records belong somewhere else entirely, given from the project root (`registry/decision`). No climbing out with `..`.

Two types may not claim the same folder, and may not claim folders where one contains the other — both would leave a file with two owners. In practice: one folder per type, side by side.

- **`filename`** — how record files are named. `prefixed` (the default) means the name starts with the ID: `DEC-004.md`, or `DEC-004-any-handle.md` if you want words in there too. `free` means the project names its files however it likes and the ID lives only in the frontmatter.

The choice is a real one. `prefixed` makes any record findable by ID in any file browser, with no search. `free` keeps filenames readable as language, which is what a vault of prose notes usually wants. Decide once, per type: with `free` the engine stops checking names altogether, and nothing you write in them is wrong.

One thing a name does affect, in either mode: if it contains parentheses or spaces, a link *pointing at* that file has to wrap the destination in angle brackets — `[label (DEC-004)](<some note (draft).md>)` — or the engine will not see the reference at all. It is not reported broken; it is invisible, which is worse. That is a limitation on this side, recorded as ISSUE-017 and explained in [references between records](troubleshooting/references.md#writing-the-destination-when-the-filename-has-spaces-or-parentheses); until it is lifted, the angle brackets are the whole remedy.

## Fields

Each field declares a type, and may add constraints on top of it.

| type | holds | extra keys |
|---|---|---|
| `string` | a line of text | `pattern` |
| `text` | a paragraph of text (same checks; the difference is intent) | `pattern` |
| `integer` | a whole number | `min`, `max` |
| `number` | a whole number or a decimal | `min`, `max` |
| `boolean` | `true` or `false` | |
| `date` | an unquoted `2026-07-12` | |
| `datetime` | an unquoted ISO 8601 date or date-and-time | |
| `enum` | one of a fixed list | `values` (required) |
| `ref` | the ID of another record, of a stated type | `entity` (required) |
| `list` | several of something else | `items` (required) |

Every field also takes:

- **`required`** — `true` or `false` (the default). It must be a real YAML boolean: a quoted `"true"` is refused rather than read as vaguely true, because that would be a decision nobody made.
- **`description`** — for whoever reads the schema in a year. Worth writing on any field whose purpose is not obvious from its name.
- **`default`** — documentation only. It records what a sensible value would be; the engine does not inject it, and an absent optional field is simply absent.

### Two field types that are doing more than they look

**`enum` is where drift dies.** Wherever the vocabulary is closed — statuses, categories, kinds — declaring the list is what stops a registry from accumulating `read`, `Read`, `finished` and `done` as four spellings of one state. It is the single highest-return declaration in the language, and the moment a genuinely new state appears, adding it to the list is a deliberate act with a date on it rather than a drift.

**`ref` makes a relation checkable.** A field declared `{type: ref, entity: requirement}` holds the ID of a requirement, and the engine verifies both that the ID exists and that it names a requirement rather than something else. Prefer one `ref` per relation over describing the relation only in prose — and then describe it in prose as well, with an inline reference, which stays readable *and* gets validated.

For several of something, wrap it: `{type: list, items: {type: ref, entity: requirement}}`. The `items` spec is a full field declaration, so a list can hold anything a field can.

### `pattern`, which pays twice

On a `string` or `text` field, `pattern` declares the *shape* its values take, as a regular expression that must match the whole value:

```yaml
ticket: {type: string, pattern: '[A-Z]{2,}-\d+'}
```

The obvious return is that a malformed value is caught. The one that surprises people: **the engine stops asking you about that field.** Before parsing your files it scans the raw text for the ways YAML silently changes what you wrote — most of them involving `#`, which starts a comment. Where it cannot tell your prose from a comment it has to flag; where you have declared the shape, it compares the line against your shape and decides for itself. A field of codes, slugs or hashtags goes from a flag on every line to silence.

Which is the general principle of the whole language, stated once: **the engine can only check what you declare, and it only pesters you about what you left undeclared.** A schema that says `{type: string}` about everything has told it almost nothing.

One caveat: a permissive pattern (`.*`) buys no silence, because it is no knowledge. The power of the declaration is the precision of the shape.

## Rules that span several records: `constraints`

Field declarations constrain one record at a time. Some rules live between records — “a piece can only be submitted once”, “at most three per student” — and those go in an optional `constraints` list, enforced across the whole project on every run.

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

**`unique`** requires the combination of the listed fields to be unique among this type's records. One field, or several together (`[room, slot]`: the same room at the same time). Records where any listed field is empty are skipped, so optional fields behave sensibly.

**`max_count_per`** groups the records by a value and errors when a group exceeds `max`. The value can be a field of the record, or one reached *through* references: `group_by: piece.student` means “the student of the referenced piece”, hopping from this record to the piece to its student. Records whose path does not resolve are skipped — absence never violates an aggregate rule, which is what makes voluntary participation expressible.

Write the `description`. It is echoed in the error message, and a cap without a stated reason invites the next person to raise it.

The vocabulary is deliberately two rules. If a rule you need cannot be expressed with them, write it into the schema's `description` and treat it as a review question rather than building a bespoke validator for one project — a rule enforced by a script nobody else runs is a rule that will be quietly wrong within a year.

## `strict`

`strict: true`, the default, means a field a record carries and the schema never declared is an error. With `strict: false` it is a warning instead, which never affects whether the run passes.

Loose is the right setting for a corpus still finding its shape: you get to see what your records are actually growing without being blocked by it. Strict is the right setting for anything other work depends on. Moving from loose to strict, once the shape has settled, is a normal step rather than a tightening you should feel bad about postponing.

## Changing a schema later

You will. The first version of a schema is a hypothesis about your own material, and real records are what test it.

Changing it is an ordinary workflow, not a repair: edit the declaration, run `validate`, fix what the change surfaces, and record what you did and why in `schemas/CHANGELOG.md`. That log is the difference between a schema that evolved and a schema that drifted — in a year, the question is never *what* the field used to be but *why* it changed.

The engine will not tell you your schema is a poor description of your material. It checks that the schema is well-formed and that your records honor it; whether the fields you chose are the right fields is a judgment nobody can automate.

## What earns a place in a schema

The useful test, applied field by field: model only the facts that must not drift. A field earns its place when

- more than one document depends on it,
- an assistant might plausibly get it wrong from memory, or
- you want to search, sort or count by it.

Everything else stays prose, and that is a feature rather than a failure of modeling. The prose is the interface; the schema is the safety net underneath it. A schema that tries to capture everything a document says has reinvented a form, and forms are what people stop filling in.
