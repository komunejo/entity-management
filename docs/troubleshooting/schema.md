# When the error is in your schema

Every message so far has been about a record. These are about the file that *declares* what a record of some type must look like — a small YAML file in `schemas/`, one per type, written by you.

If you are meeting these for the first time, you are probably adopting: designing the shape of your registry is the work that adoption asks for, and these messages are the engine checking your design before it checks anything against it. That is deliberate — a schema the engine half-understood would validate your records against a rule you never wrote.

Here is a complete one, so the messages below have something to refer to:

```yaml
entity: decision            # the type's name; must match this file's name
description: A decision taken and its reasoning
id:
  prefix: DEC               # IDs look like DEC-001
  width: 3                  # minimum digits, zero-padded
dir: decision               # its records live in entities/decision/
strict: true                # undeclared fields are errors, not warnings
fields:
  title:    {type: string, required: true}
  status:   {type: enum, required: true, values: [proposed, accepted, superseded]}
  date:     {type: date, required: true}
  supersedes: {type: ref, entity: decision}
  tags:     {type: list, items: {type: string}}
constraints:
  - rule: unique
    fields: [title]
```

What may appear in a schema, and why you would choose one thing over another, is [designing schemas](../schemas.md); the normative specification, written for agents, is [`schema-language.md`](../../entity-management/references/schema-language.md). This page is the other end of both: what the engine says when something does not hold.

## Before anything else: the file has to parse

`schema is not valid YAML: …` and `schema must be a YAML mapping` mean exactly what the equivalents mean for a record — see [the record's frontmatter and its identity](record-structure.md), which explains PyYAML's own messages.

Schema files also go through the same pre-parse scan as records, so a comment that eats half a line is reported here too, with the same remedies: [when the file looks fine and YAML disagrees](quoting.md). The engine knows its own contract, so a comment after `type:`, `required:`, `prefix:` or any other key whose value it controls is provably a comment and stays silent; a comment after a free-text value such as `description:` is ambiguous and is flagged.

## Identity: the type's name and its IDs

### `unknown schema key '<key>' (valid: constraints, description, dir, entity, fields, filename, id, path, strict)`

A top-level key the schema language does not define. Nearly always a spelling slip, and worth catching precisely because the alternative is silence: a key nobody reads is a rule you believe you declared and do not have.

### `schema needs an 'entity' name (string)`

Every schema names the type it declares. This is the name that appears in each record's `entity:` field.

### `schema entity '<name>' should match filename '<name>.yaml'`

The type declared inside the file and the file's own name must agree, so that finding the schema for a type is never a search.

### `duplicate schema for entity '<name>'`

Two schema files declare the same type name. One of them is a leftover — often a copy made during a rename. The second one loaded is dropped entirely, so its rules are not in force at all.

### `schema needs an 'id:' mapping with at least a 'prefix'`

Every type declares how its records are identified:

```yaml
id:
  prefix: DEC
  width: 3
```

`width` is optional and means *minimum* digits, zero-padded — with `width: 3`, IDs run `DEC-001` through `DEC-999` and then simply continue as `DEC-1000`, without a migration.

### `unknown id key '<key>' (valid: prefix, width)`

Only those two.

### `id prefix must be alphanumeric starting with a letter, got …`

A prefix is letters and digits, beginning with a letter: `DEC`, `ISSUE`, `REQ2`. No hyphens (the hyphen is the separator), no spaces, no accented characters.

That last restriction is deliberate rather than an oversight. IDs are the one part of a record that travels through URLs, filenames, shell commands and other people's tools, so they are kept to plain ASCII. Everything else in your registry — titles, labels, prose, and the filenames themselves under a free naming policy — takes accents, `ñ`, `¿`, `¡` and any other character you need. The restriction is on the prefix only, and it fails loudly rather than quietly mangling anything.

### `id prefix '<p>' already used by entity '<e>' — prefixes must be unique`

Two types claim the same prefix, which would make `X-004` ambiguous — and references are resolved by prefix. Choose another; the schema that hit the collision is dropped until you do.

### `id width must be a positive integer, got …`

A count of digits: `3`, not `"3"` and not `0`.

### `'strict' must be a boolean, got … — any other value would be silently dead configuration`

`strict` decides what happens to a field a record carries and the schema never declared: an error (`true`, the default) or a warning (`false`). It takes a real YAML boolean, bare.

Anything else — a quoted `"true"`, a stray word — would previously be read as merely *truthy*, which is a decision nobody made. The engine now asks for the real thing. Left empty beside a comment (`strict:` with nothing after it) it means the documented default, not a falsy accident.

## Location: where the type's records live

### `'dir' must be a non-empty string, got …` / `'path' must be a non-empty string, got …`

`dir` names a folder under the entities directory (the default is the type's own name). `path` is the alternative for a type whose records live somewhere else entirely, given from the project root.

### `'path' must be relative to the project root, without '..': …`

An absolute path or one that climbs out with `..` is refused. A project describes itself in terms of itself; a schema that pointed outside the project would validate files that do not travel with it.

### `records location '<d>' already used by entity '<e>' — entity locations must be unique` / `records location '<d>' is nested with entity '<e>''s location — entity locations must not nest`

Two types cannot claim the same folder, and cannot claim folders where one contains the other. Both would leave files with two owners, and the engine finds records by location — see [where files live](placement.md).

The shape this produces in practice is one folder per type, side by side. If your material genuinely nests — a folder of things, each containing its own things — that is a real limitation of the current model rather than a rule you should work around; the project tracks it against itself as ISSUE-009 and ISSUE-010.

### `'filename' must be one of free, prefixed, got … — falling back to default 'prefixed'`

How this type's records are named. `prefixed` (the default) means the filename starts with the ID; `free` means the project names its files as it likes and the ID lives only in the frontmatter.

Note the tail: the engine falls back to the default rather than dropping the schema. Enforcing a naming rule nobody declared would report violations of a rule that does not exist.

## Fields

### `schema needs a non-empty 'fields' mapping`

A type with no fields declares nothing.

### `field '<name>' is reserved (added automatically)`

`id` and `entity` belong to every record and are checked by the engine itself. Declaring them again would create two sources of truth for the same thing.

### `field spec must be a mapping, got <something>`

Each field is declared as a mapping, even when it has a single key:

```yaml
title: {type: string}       # not: title: string
```

### `unknown or missing type '<t>' (valid: boolean, date, datetime, enum, integer, list, number, ref, string, text)`

Every field declares one of those ten. `string` and `text` differ only in intent (a line versus a paragraph); `ref` points at another record; `list` holds many of something else.

### `unknown field key '<key>'`

A key the field vocabulary does not define: `type`, `required`, `values`, `entity`, `items`, `min`, `max`, `pattern`, `description`, `default`.

`default` is documentation only — it records what a sensible value would be. The engine does not inject defaults: an absent optional field is simply absent.

### `key '<key>' does not apply to type '<t>' — it would be silently dead configuration`

The key is real, but not for this type — `values` on a `string`, `min` on a `date`, `pattern` on an `integer`.

This is the message that most often reads as pedantry and is not. A `pattern` on an integer field is a rule you wrote, believe you have, and do not: nothing would ever check it. The engine refuses to hold configuration it will never read.

Which keys apply where: `pattern` on `string` and `text`; `min` and `max` on `integer` and `number`; `values` on `enum`; `entity` on `ref`; `items` on `list`. `type`, `required`, `description` and `default` apply to everything.

### `'required' must be a boolean, got … — any other value would be silently dead configuration`

`true` or `false`, bare. Not `"true"`, not `yes` in quotation marks, not a word. A quoted string would previously be read as merely *truthy* — a decision nobody made — so it is refused rather than guessed at.

### `'min' must be a number, got …` / `'max' must be a number, got …`

Numeric bounds take numbers. Note that `true` is not one: YAML's booleans are numbers to Python and not to you, so they are excluded explicitly.

### `'pattern' must be a string, got …` / `'pattern' is not a valid regex: …`

A pattern is a regular expression, given as a string, and it is compiled when the schema loads so that a broken one is reported here rather than misfiring later against every record.

Two things worth knowing when you write one: it must match the **whole** value, not a fragment; and declaring it does more than validate. A field with a declared shape becomes decidable for the comment scan too — see [when the file looks fine and YAML disagrees](quoting.md). A pattern is often the durable fix for a whole class of noisy flags.

### `enum needs a non-empty 'values' list` / `ref needs an 'entity' (target entity name)` / `list needs an 'items' field spec`

Three types that are incomplete without one more key: what the allowed values are, what type is being pointed at, and what the list holds.

```yaml
status: {type: enum, values: [open, resolved]}
author: {type: ref, entity: person}
tags:   {type: list, items: {type: string}}
```

The `items` spec is a full field declaration, so a list of references is `{type: list, items: {type: ref, entity: person}}`.

### `ref targets unknown entity '<e>'`

A `ref` field points at a type no schema in this project declares. Either the target type has not been written yet, or the name is misspelled — it is the type's name, not its prefix or its folder.

## Constraints

### `'constraints' must be a list` / `constraint must be a mapping`

Constraints are a list of entries, each a mapping:

```yaml
constraints:
  - rule: unique
    fields: [room, slot]
  - rule: max_count_per
    group_by: supervisor
    max: 3
    description: no supervisor takes more than three concurrent projects
```

### `unknown or missing rule '<r>' (valid: max_count_per, unique)`

Two rules exist today. Each entry names one.

### `unknown constraint key '<key>'`

The keys depend on the rule: `unique` takes `fields`; `max_count_per` takes `group_by` and `max`. Both also take `description`, which is appended to the violation message — worth writing, because a cap without a stated reason invites the next person to raise it.

### `unique needs a non-empty 'fields' list of field names` / `max_count_per needs a 'group_by' field or ref-path` / `max_count_per needs a non-negative integer 'max'`

Each rule's own required keys, missing or malformed.

An invalid constraint is reported and then dropped, and the rest of the schema keeps working. One broken rule never disables validation for the whole type.

### `unique references undeclared field '<f>'`

The uniqueness rule names a field this type does not have. Usually a rename that reached the fields and not the constraint.

### `group_by path '<p>': '<hop>' is not a declared field of entity '<e>'` / `group_by path '<p>': '<hop>' must be a ref to continue the path`

A cap can group by a value reached *through* references — `group_by: student.department` walks from the record to the student it points at, and reads that student's department.

The first message means one step of that walk names a field that does not exist on the type it was read from; the message says which type, so you can see where the walk went wrong. The second means an intermediate step is not a reference — you can only walk *through* a `ref`, since only a reference leads to another record.

## What the engine will not do

There is no message for a schema that is *wrong about your project* — one that requires a field nobody can fill, or forbids a state your material genuinely has. The engine checks that a schema is well-formed and self-consistent, never that it is a good description of the world. That judgment is yours, and revising it is expected: schema evolution is a first-class workflow here, not a repair.
