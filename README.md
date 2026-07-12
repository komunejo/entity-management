# entity-manager

Database-engine-like integrity for projects whose infrastructure is Markdown documents written in natural-language collaboration between humans and AI agents.

The problem: when humans and agents collaborate in prose instead of structured languages (SQL, Python, HTML), the structured facts embedded in that prose drift — names, statuses, dates, relations. Neither biological nor artificial narrative agents have reliable enough memory to keep them consistent.

The approach: keep natural language as the working interface, and make the implicit structure explicit and verifiable *underneath it*. Each entity is a Markdown file whose YAML frontmatter is a typed record validated against a per-project schema; the prose body stays free. A deterministic engine — not the model's discipline — guarantees hard integrity.

## Getting started

No technical setup is required: download [`entity-management.skill`](entity-management.skill) and add it to your Claude environment (the file card offers a *Save skill* button), or simply give your AI assistant the URL of this repository and ask it to install the skill from it.

If you prefer a manual install, copy the [`entity-management/`](entity-management/) folder into your skills directory.

Then, inside any project, just ask in plain language — "I want to track X as consistent records", "check the structural consistency of the documents" — and the skill will propose schemas, ask for your contract policy, and keep the records validated from then on. The full schema language is specified in [`entity-management/references/schema-language.md`](entity-management/references/schema-language.md).

The engine requires Python 3.8+ and PyYAML.

## Contents

- [`entity-management/`](entity-management/) — the skill: [`SKILL.md`](entity-management/SKILL.md), the validation engine ([`scripts/entity_lint.py`](entity-management/scripts/entity_lint.py)), reference documentation ([`references/schema-language.md`](entity-management/references/schema-language.md), [`references/hooks.md`](entity-management/references/hooks.md)), and the skill's own [`CHANGELOG.md`](entity-management/CHANGELOG.md).
- [`schemas/`](schemas/), [`entities/`](entities/) — this repository is itself an entity-managed project (dogfooding): the skill's own requirements (`REQ-*`) and design decisions (`DEC-*`) are records validated by its own engine, including the proposed backlog (decisions in `proposed` status). See the generated [`INDEX.md`](INDEX.md).
- [`entity-manager.yaml`](entity-manager.yaml) — project configuration, including the contract policy (`policy.on_unresolvable`).
- [`VERSION.md`](VERSION.md), [`LICENSE.md`](LICENSE.md), [`WAIVER.md`](WAIVER.md) — release version (aligned with the skill changelog), license, and user acknowledgment. Repository infrastructure inspired by [peterkaminski-ai/pkai-agent](https://github.com/peterkaminski-ai/pkai-agent).
- [`entity-management.skill`](entity-management.skill) — the packaged, install-ready build of the skill. It is a duplicate of [`entity-management/`](entity-management/) by construction and is rebuilt on every release (see Releasing); between releases, the folder is the source of truth.

## Quick taste

```bash
python3 entity-management/scripts/entity_lint.py validate
python3 entity-management/scripts/entity_lint.py index
python3 entity-management/scripts/entity_lint.py new decision --title "Try it"
```

## Releasing

Every release keeps four artifacts telling the same story: an entry in [`entity-management/CHANGELOG.md`](entity-management/CHANGELOG.md) (what changed and why), the version in [`VERSION.md`](VERSION.md), a rebuilt [`entity-management.skill`](entity-management.skill) package, and — once the repository is under git — a `vX.Y.Z` tag. The package is never edited by hand and never left stale: if [`entity-management/`](entity-management/) changed, repackage before tagging.

## License

Copyright © 2026 V. Gracia. Licensed under the Mozilla Public License 2.0 — see [`LICENSE.md`](LICENSE.md). By using this software you acknowledge [`WAIVER.md`](WAIVER.md).
