# entity-manager

Your project's infrastructure is a folder of Markdown documents, written in natural language between humans and artificial narrative entities. This gives that folder the kind of integrity a database engine gives its tables — without taking the prose away from you.

The problem: when humans and agents collaborate in prose instead of structured languages (SQL, Python, HTML), the structured facts embedded in that prose drift — names, statuses, dates, relations. Neither biological nor artificial narrative entities (aka AI agents) have reliable enough memory to keep them consistent.

The approach: keep natural language as the working interface, and make the implicit structure explicit and verifiable *underneath it*. Each entity is a Markdown file whose YAML frontmatter is a typed record validated against a per-project schema; the prose body stays free. A deterministic engine — not the model's discipline — guarantees hard integrity.

## What that looks like

A record is one of your documents, with a small structured block on top and your prose underneath:

```markdown
---
id: DEC-004
entity: decision
title: "Records are named by their ID"
status: accepted
date: 2026-07-12
supersedes: DEC-001
---

We keep the ID in the filename so any record can be found by ID in any file
browser. This replaces [the original naming rule (DEC-001)](DEC-001.md), which
named files after their titles and broke whenever a title was edited.
```

A schema is the rulebook for one type of record, and you write it:

```yaml
entity: decision
id:
  prefix: DEC
fields:
  title:      {type: string, required: true}
  status:     {type: enum, required: true, values: [proposed, accepted, superseded]}
  date:       {type: date, required: true}
  supersedes: {type: ref, entity: decision}
```

From then on, a decision without a date is an error, a status nobody declared is an error, and that link in the prose is checked: `DEC-001` must exist, must be a decision, and must be at that path. Everything else in the document is yours, unjudged.

The schema is **declared**: something somebody authored, that the engine validates against rather than inferring from your files. Drafting it from material you already have is ordinary work, and an assistant can do most of it; what adoption asks is that the declaration exist, and that it be yours. [What adopting asks of you](docs/adopting.md) covers that before you install anything.

## Getting started

No technical setup is required: download [`entity-management.skill`](entity-management.skill) and add it to your Claude environment (the file card offers a *Save skill* button), or simply give your agent the URL of this repository and ask it to install the skill from it.

If you prefer a manual install, copy the [`entity-management/`](entity-management/) folder into your skills directory.

Then, inside any project, just ask in plain language — “I want to track X as consistent records”, “check the structural consistency of the documents” — and the skill will propose schemas, ask for your contract policy, and keep the records validated from then on.

## What it needs to run

The engine is a small Python program: it needs Python 3.8 or newer and [PyYAML](https://pypi.org/project/PyYAML/), and nothing else. An assistant installing the skill takes care of both on its own. On Claude Desktop not even that concerns you — Claude's work runs in an isolated environment of its own (see the [Claude Help Center](https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork)), so your machine needs nothing installed at all.

Working from the CLI or by hand, use whatever you normally use; the engine only needs an interpreter that can `import yaml`. No Python on the machine? [python.org](https://www.python.org/downloads/) has the official installer for every system, and [before the engine can look](docs/troubleshooting/runtime.md) tells both stories calmly, with examples.

## Documentation

**For humans:**

- [`docs/adopting.md`](docs/adopting.md) — what this asks of you, what a record and a schema are, when *not* to adopt, and the two ways in (a project born structured, or an existing corpus converted). Read before installing anything.
- [`docs/quick-guide.md`](docs/quick-guide.md) — the walkthrough: from an empty folder to a validated record, with the real files and the real output, including the first red and where it leads.
- [`docs/schemas.md`](docs/schemas.md) — designing schemas: the whole vocabulary explained for a person writing their own — field types, relations, rules that span records, and what earns a place in a schema at all.
- [`docs/troubleshooting/`](docs/troubleshooting/troubleshooting.md) — every error the engine can raise, explained: what happened, why it matters, how to fix it. Indexed by message, so a red line in your terminal leads straight to its page.

**For agents:** the skill carries its own reference layer — [`SKILL.md`](entity-management/SKILL.md) and [`references/`](entity-management/references/), covering the [schema language](entity-management/references/schema-language.md), [automatic validation](entity-management/references/hooks.md), [flag triage](entity-management/references/troubleshooting.md), and [comment decidability](entity-management/references/comment-decidability.md).

## What is in this repository

- [`entity-management/`](entity-management/) — the skill itself: instructions, the validation engine ([`scripts/entity_lint.py`](entity-management/scripts/entity_lint.py)), its reference documentation, and the skill's own [`CHANGELOG.md`](entity-management/CHANGELOG.md).
- [`entity-management.skill`](entity-management.skill) — the packaged, install-ready build. A duplicate of `entity-management/` by construction, rebuilt on every release; between releases, the folder is the source of truth.
- [`docs/`](docs/) — the human documentation described above.
- [`schemas/`](schemas/), [`entities/`](entities/) — this repository manages itself with its own skill. Its requirements (`REQ-*`), design decisions (`DEC-*`), backlog (`PROP-*`) and reported defects (`ISSUE-*`) are records validated by its own engine, which is also how the method gets tested against a real registry every day. See the generated [`INDEX.md`](INDEX.md).
- [`entity-manager.yaml`](entity-manager.yaml) — project configuration, including the contract policy (`policy.on_unresolvable`).
- [`VERSION.md`](VERSION.md), [`LICENSE.md`](LICENSE.md), [`WAIVER.md`](WAIVER.md) — release version (aligned with the skill changelog), license, and user acknowledgment. Repository infrastructure inspired by [peterkaminski-ai/pkai-agent](https://github.com/peterkaminski-ai/pkai-agent).

## Running the engine directly

Everything the skill does, an assistant does for you. If you want to run it yourself:

```bash
python3 entity-management/scripts/entity_lint.py validate
python3 entity-management/scripts/entity_lint.py index
python3 entity-management/scripts/entity_lint.py new decision --title "Try it"
```

## Releasing

Every release keeps four artifacts telling the same story: an entry in [`entity-management/CHANGELOG.md`](entity-management/CHANGELOG.md) (what changed and why), the version in [`VERSION.md`](VERSION.md), a rebuilt [`entity-management.skill`](entity-management.skill) package, and — once the repository is under git — a `vX.Y.Z` tag. The package is never edited by hand and never left stale: if [`entity-management/`](entity-management/) changed, repackage before tagging.

## License

Copyright © 2026 V. Gracia. Licensed under the Mozilla Public License 2.0 — see [`LICENSE.md`](LICENSE.md). By using this software you acknowledge [`WAIVER.md`](WAIVER.md).
