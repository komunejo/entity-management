# Comment decidability: what the engine knows, and your part

How the pre-parse scan treats ` #` since 0.7.1 (DEC-023 in the skill's own registry), and what each flag expects from you. The human-facing explanation of every message lives in the repository's troubleshooting documentation; this page is the agent's reasoning and routes.

## What the engine knows, and from where

The scan is silent wherever a declaration proves a ` #` tail could not be part of a *legal value* — that is the exact guarantee: nothing that could have validated as content is lost without a word. Truncation that leaves an illegal remainder reds downstream with the better message; a tail lost beside a still-legal remainder (`status: open # and also pending`) could only have been a comment or an illegality. One construct is excepted from every silence: a `#`-glued value (`key: #text`) parses to null, so on an optional field nothing downstream ever sees what vanished — it flags on every field kind, with the remedy that exists for the field (quotes where it holds text, the real value where the type could never contain `#…`). The tie-break behind that exception, stated once: a false red is paid once and teaches its fix; a silent loss never announces itself.

- **Typed fields** (`integer`, `number`, `boolean`, `date`, `datetime`): a legal value can never contain ` #`. A tail that damaged the value leaves an illegal remainder the type check rejects; a tail beside a still-legal value could never have been content.
- **`enum`** (declared values without `#`) and **`ref`**: same logic — the enum and ref checks are the backstop.
- **`string`/`text` with a declared `pattern`**: decided per line against the shape. Value matches and the whole line does not → the tail is provably not part of a legal value → silence. The whole line matches → the value legitimately contains ` #` and YAML is about to truncate it → flag. Neither → the truncated value fails the pattern check downstream. A permissive pattern (`.*`, `.+`) declares that anything can be content — the engine gains nothing from it and the flag behaves as on shapeless prose.
- **The engine's own contracts**: schema keys (`type`, `strict`, `required`, …) and `entity-manager.yaml`'s `policy.on_unresolvable` — including its key names — are validated, so comments beside them are free. Contract knowledge never leaks to lookalikes: a nested or foreign key sharing a contract key's name keeps the ambiguous treatment.

Where nothing can decide — prose without a declared shape — the engine does not guess: the flag speaks, and that is the design, not a defect.

## Your three responses to a comment flag

1. **It is prose → quote it.** The golden rule was already yours: when a value holds prose, quote it. One edit, revalidate, the human never hears of it.
2. **The field's values have a shape → declare `pattern`.** Codes, slugs, hashtags: the right fix is not a quote per line but a declaration. When a citable criterion decides it — the human stated the convention, a recorded project criterion covers it, the shape is constitutive of what the field means — make the change yourself: schema evolution is the normal case (see SKILL.md), documented in `schemas/CHANGELOG.md` and reported, never asked as permission for what a criterion already settled. When no criterion is citable, the shape's *intent* is the human's — today's values may all match a pattern the field was never meant to be closed to — and that is your one question, whose answer becomes a recorded criterion.
3. **Genuine doubt about intent → one question.** When the flagged text is not yours and nothing citable decides it, ask about intent, never mechanics ("is `per item` part of the title, or a note about it?"), bundle if there are several, and record the answer as a project criterion so it is never asked twice.

## The frontier no declaration crosses

`tag: #label` unquoted is null to every YAML reader: the format discards everything after the colon at parse time, before any engine looks. The engine will not "recover" it from raw text — it validates YAML, it does not amend it, and a record must mean the same thing to the engine and to every other reader. Values that begin with `#` are written quoted, always: `tag: "#label"   # a comment here is fine`.

## Idioms that stay silent

A spaced comment after an empty value (`key: # to fill in`, `key: ## section`) — the engine's own stubs emit the first form. Comments on their own line, at any indentation. Comments after quoted values, after typed values, after closed collections. Annotate freely there.
