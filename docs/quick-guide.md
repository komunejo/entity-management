# From an empty folder to a validated record

This is the walkthrough: what actually happens, in order, with the real files and the real output. It assumes you have decided to adopt — if you are still weighing that, [what adopting asks of you](adopting.md) is the page for it — and that the skill is installed, which the [README](../README.md) covers in three lines.

Everything below can be done by asking your assistant in plain language. It is written out so you can see what the asking produces, and so you can do it yourself if you would rather.

The example is deliberately small: a person who wants to keep track of what they read.

## 1. Pick one type. One.

Not every kind of document you own — the one whose facts you are tired of losing. Here: sources. A source has a title, an author, a state (to read, reading, read) and a date I added it.

Starting with one type is not a cautious half-measure; it is how this goes well. You will change your mind about the fields once real documents meet them, and changing your mind about one type is an afternoon.

## 2. Make the skeleton

Three things at the project root:

```
my-library/
├── entity-manager.yaml
├── schemas/
└── entities/
```

`entity-manager.yaml` is short, and one line of it is a decision rather than a setting:

```yaml
schemas_dir: schemas
entities_dir: entities
policy:
  on_unresolvable: block
```

That last key is the project's **contract policy**, and it answers a question you will meet for real within the week: what happens when a record cannot satisfy the schema and nobody knows the missing truth. `block` means the red stays and is itself the answer — an honest list of what needs a decision. `relax-and-report` means the contract may be loosened to keep work moving, with every loosening written down. Neither is the safe choice in general; `block` is the right default for a registry other things depend on. It is explained in full under [when red is the honest answer](troubleshooting/troubleshooting.md#when-red-is-the-honest-answer).

## 3. Write the schema

One file, `schemas/source.yaml`, named after the type it declares:

```yaml
entity: source
description: A text I read and want to be able to cite later.
id:
  prefix: SRC
fields:
  title:  {type: string, required: true}
  author: {type: string, required: true}
  status: {type: enum, required: true, values: [to-read, reading, read]}
  added:  {type: date, required: true}
```

Four fields, because four facts are the ones I actually rely on. Everything else I might want to say about a book belongs in the prose, where it is free.

Two choices in there worth noticing, because they are the ones that pay:

- `status` is an **enum**, not a string. That single decision is what stops a registry from accumulating `read`, `Read`, `finished` and `done` as four states meaning one. Wherever your vocabulary is closed, close it.
- `added` is a **date**, not text. Now it sorts, filters and groups; and a typo in the format is caught rather than discovered a year later.

The full vocabulary — every field type, the constraints that work across records, how to relocate a type's folder — is in [designing schemas](schemas.md). You do not need it today.

## 4. Validate before you have anything

```bash
python3 <skill-path>/scripts/entity_lint.py validate --root .
```

```
OK: 0 entities across 1 schemas — 0 error(s), 0 warning(s)
```

This is not a ceremony. The engine checks *schemas* before it checks records, so this run tells you your schema itself is well-formed — before you write twenty records against a rule that was malformed all along. If something is wrong here, [when the error is in your schema](troubleshooting/schema.md) is the page.

Read the count while you are here: `0 entities across 1 schemas`. That number is part of every verdict, and it is the only thing that catches a record the engine cannot see at all.

## 5. Create the first record

Never invent IDs by hand — ask the engine:

```bash
python3 <skill-path>/scripts/entity_lint.py new source --title "The Social Construction of Reality" --root .
```

```markdown
---
id: SRC-001
entity: source
title: "The Social Construction of Reality"
author:
status:  # one of: to-read, reading, read
added:  # YYYY-MM-DD
---

(prose body — free-form Markdown; reference entities inline as [label (ID)](path/to/ID.md), or [ID](path/to/ID.md) when unlabeled)
```

It also tells you where the file goes: `entities/source/SRC-001.md`.

Notice what the stub does. It has taken the next free ID; it has quoted the title for you; and every field it could not fill carries the hint it needs — the enum lists its legal values, the date states its format. You fill in the blanks and delete the hints:

```markdown
---
id: SRC-001
entity: source
title: "The Social Construction of Reality"
author: Berger and Luckmann
status: reading
added: 2026-07-27
---

The book that gave me the phrase I keep using.
```

## 6. Validate again

```
OK: 1 entities across 1 schemas — 0 error(s), 0 warning(s)
```

That is the whole loop. Everything after this is more of it.

## 7. Your first red, which arrives on day one

It will look something like this — here, a second record where the status was capitalized and the date was quoted:

```
ERROR   entities/source/SRC-002.md [status]: value 'Reading' not in enum ['to-read', 'reading', 'read']
ERROR   entities/source/SRC-002.md [added]: expected date (YYYY-MM-DD, unquoted), got str '2026-07-27'
FAIL: 2 entities across 1 schemas — 2 error(s), 0 warning(s)
```

Read it as three pieces: the file, the field in square brackets, and the message. The first is a capital letter. The second is subtler and worth learning once — quotation marks around a date make it *text*, so a `date` field wants the bare `2026-07-27`. Both are one keystroke.

Two red lines on day one is the system working. [When the engine says no](troubleshooting/troubleshooting.md) explains every message it can produce, indexed so you can go straight from the line in your terminal to its explanation.

## 8. Now convert ten, not two hundred

If you are bringing an existing collection in, this is the step where it is won or lost. Convert about ten documents by hand and validate. Ten is enough to discover that the schema you wrote is not quite the schema you needed — which is normal, and is exactly what this step is for.

Then **change the schema**, and log the change in `schemas/CHANGELOG.md`. Evolving a schema is a first-class workflow here, not an admission of failure. Only after that does bulk conversion make sense, and bulk conversion is the part to hand to your assistant.

Reversing this order — converting everything against an untested schema — is how a first run produces hundreds of errors and reads like a verdict on the method. It is not one. It is a corpus meeting a rule nobody has tried yet.

## 9. Make it automatic, and make it visible

Two things worth doing once you have more than a handful of records, both of which your assistant sets up on request:

- **Automatic validation** — after every file edit, at the start of a session, and as a git pre-commit hook. The pre-commit one matters most: it is what stands between a hand-edited slip and your repository's history.
- **A generated index** — `entity_lint.py index --write INDEX.md` writes a table of every record, by type, with links. It is generated, so it never drifts; do not edit it by hand.

## What you have now

A folder where the facts you care about are checked instead of remembered, and where a green run is a claim you can lean on. Prose is still prose; nothing about how you write has changed.

Where to go next: [designing schemas](schemas.md) when you want a second type, a relation between types, or a rule that spans several records; [when the engine says no](troubleshooting/troubleshooting.md) when something goes red and the message is not obvious.
