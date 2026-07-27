# When the engine says no

This is the guide for humans: every message the validation engine can raise, explained — what actually happened, why it is worth stopping for, and how to fix it. The [schema language specification](../../entity-management/references/schema-language.md) works from the other end, saying how things *should* be written; these pages start at the moment something was not.

One idea organizes all of it. The registry's whole value is that a green run means the records can be trusted. Every message in this catalog exists because some way of writing a record would quietly make that green a lie. So when the engine blocks you it is not enforcing style — it is refusing to vouch for something it cannot back. The fix is usually one line.

**If you have a message in your hand**, the [index of messages](#index-of-messages) at the foot of this page takes you straight to its explanation. If you are here to understand the shape of things, read on.

## The pages

| Page | What it covers |
|---|---|
| [Before the engine can look](runtime.md) | Python, PyYAML, no project root, unreadable files — exit code 2 |
| [When the file looks fine and YAML disagrees](quoting.md) | The silent-misparse family: comments, commas, quotation marks |
| [The record's frontmatter and its identity](record-structure.md) | The block at the top of a record; ids, filenames, unknown fields |
| [Types and values](values.md) | A value that is not what the schema declared it to be |
| [References between records](references.md) | What makes a link a reference, and every way one can break |
| [Where files live](placement.md) | Location, strays, and the file the engine skips in silence |
| [Rules that cross several records](constraints.md) | Uniqueness and caps — the checks no single file can see |
| [When the error is in your schema](schema.md) | The file that declares a type: identity, fields, constraints |
| [When the error is in the project configuration](config.md) | `entity-manager.yaml`: locations and the contract policy |

If you are new here rather than stuck, three pages come before this one: [what adopting asks of you](../adopting.md) if you are still weighing it, [the walkthrough](../quick-guide.md) if you are ready to start, and [designing schemas](../schemas.md) when you want the full vocabulary. Most of what looks like a wall of errors on a first run is answered in the first of those.

## How an error reaches you

Three doors, and it matters which one you came through.

**You ran the validator yourself**, or a git pre-commit hook blocked your commit. You are holding the raw output: one line per violation, each naming a file, usually a field or a line number, and a message from this catalog. The pre-commit block is the door most hand-edits arrive through, and it exists precisely because editing a record by hand, with no agent watching, is a normal and legitimate thing to do. The hook is what stands between an honest slip and the repository's history.

```
ERROR   entities/decision/DEC-004.md [status]: value 'Open' not in enum ['open', 'resolved']
FAIL: 54 entities across 4 schemas — 1 error(s), 0 warning(s)
```

That last line is worth reading even when the run is green: **the entity count is part of the verdict.** A number lower than you expect means a record was not seen at all, which no error message will tell you.

**An agent brought it to you.** This door has a nuance the others lack. Agents working under this skill are instructed to fix what they can decide on their own — their own typos, anything the conversation already settles — and to bring you *only* the flags they have no standing to resolve. So an agent's question is rarely “how do I fix this?”; it is “what did you mean?”.

A useful answer settles the intent, not the mechanics: “that comma is part of the sentence, keep it as text” resolves the flag. Explaining YAML quoting rules back to the agent adds nothing it does not already know. And when your answer establishes a rule — “in this project, titles are always literal text, quote them all” — a well-behaved agent records the criterion so the same question never comes back.

**Exit codes**, if you are scripting. `0` means integrity holds; `1` means violations were found and listed; `2` means the run itself was invalid — no project root, a missing library, a path filter that matched nothing. Never read a `2` as “no violations”: it means the engine could not even look.

## When red is the honest answer

Sometimes a record violates the schema and no available truth can fix it: a required field nobody knows the value of, a reference to something that was never registered. That is not a formatting problem, and the engine will not be talked out of it.

What happens next is the project's *contract policy*, declared in `entity-manager.yaml` as `policy.on_unresolvable`:

- **`block`** — the red stays, and is itself the deliverable: an honest list of what cannot be managed without a human decision.
- **`relax-and-report`** — the contract may be loosened to keep work moving, and every loosening is logged in `schemas/CHANGELOG.md`.

What is never acceptable, under either policy, is inventing a value to turn red into green. A fabricated green costs more than any red, because it spends the trust that every other green depends on.

And if you believe the engine itself is wrong — the flag looks like a false positive, the message misnames your case — that is worth reporting, not working around. This skill's own repository tracks its defects as `ISSUE-*` records, and more than one rule in this catalog exists because someone reported exactly that.

## Index of messages

Alphabetical by the message's own first words. Where a message begins with a value from your project, it is listed under the word that follows.

| Message begins | Page |
|---|---|
| `' #' inside a flow collection starts a comment` | [quoting](quoting.md#--inside-a-flow-collection-starts-a-comment--the-rest-of-the-line-silently-vanishes-quote-the-scalar) |
| `cannot read config as UTF-8 text` | [runtime](runtime.md#cannot-read-file-as-utf-8-text---cannot-read-config-as-utf-8-text-) |
| `cannot read file as UTF-8 text` | [runtime](runtime.md#cannot-read-file-as-utf-8-text---cannot-read-config-as-utf-8-text-) |
| `config is not valid YAML` | [config](config.md#config-must-be-a-yaml-mapping--config-is-not-valid-yaml---cannot-read-config-as-utf-8-text-) |
| `config key '…' must be a non-empty string` | [config](config.md#config-key-key-must-be-a-non-empty-string-got---falling-back-to-default-default) |
| `config key 'policy' must be a mapping` | [config](config.md#config-key-policy-must-be-a-mapping-got-) |
| `config must be a YAML mapping` | [config](config.md#config-must-be-a-yaml-mapping--config-is-not-valid-yaml---cannot-read-config-as-utf-8-text-) |
| `constraint 'max_count_per …' violated` | [constraints](constraints.md#constraint-max_count_per-g-violated-v-has-n-records-max-m-id-id-) |
| `constraint 'unique' on […] violated` | [constraints](constraints.md#constraint-unique-on-f1-f2-violated-same-values-as-x) |
| `constraint must be a mapping` | [schema](schema.md#constraints-must-be-a-list--constraint-must-be-a-mapping) |
| `'constraints' must be a list` | [schema](schema.md#constraints-must-be-a-list--constraint-must-be-a-mapping) |
| `could not find project root` | [runtime](runtime.md#could-not-find-project-root-no-entity-manageryaml-or-schemas-upwards-from-cwd-use---root) |
| `'dir' must be a non-empty string` | [schema](schema.md#dir-must-be-a-non-empty-string-got---path-must-be-a-non-empty-string-got-) |
| `duplicate id '…'` | [record structure](record-structure.md#duplicate-id-x-also-in-file) |
| `duplicate key '…' in mapping` | [quoting](quoting.md#duplicate-key-x-in-mapping-line-n--the-first-value-would-be-silently-discarded) |
| `duplicate schema for entity '…'` | [schema](schema.md#duplicate-schema-for-entity-name) |
| `entities directory not found` | [placement](placement.md#schemas-directory-not-found--is-this-an-entity-managed-project--entities-directory-not-found) |
| `enum needs a non-empty 'values' list` | [schema](schema.md#enum-needs-a-non-empty-values-list--ref-needs-an-entity-target-entity-name--list-needs-an-items-field-spec) |
| `expected <type>, got …` | [values](values.md#expected-type-got-something-) |
| `field '…' is reserved` | [schema](schema.md#field-name-is-reserved-added-automatically) |
| `field '…' not declared in schema` (warning) | [record structure](record-structure.md#unknown-field-x-not-declared-in-schema-type) |
| `field spec must be a mapping` | [schema](schema.md#field-spec-must-be-a-mapping-got-something) |
| `file is not inside any declared entity directory` | [placement](placement.md#file-is-not-inside-any-declared-entity-directory-known-) |
| `'filename' must be one of free, prefixed` | [schema](schema.md#filename-must-be-one-of-free-prefixed-got---falling-back-to-default-prefixed) |
| `filename should be the id '…'` | [record structure](record-structure.md#filename-should-be-the-id-x-optionally-followed-by---and-a-handle-eg-xmd-x-a-handlemd) |
| `frontmatter entity '…' does not match directory-implied entity` | [record structure](record-structure.md#frontmatter-entity-x-does-not-match-directory-implied-entity-y) |
| `frontmatter is not valid YAML` | [record structure](record-structure.md#frontmatter-is-not-valid-yaml-) |
| `frontmatter must be a YAML mapping` | [record structure](record-structure.md#frontmatter-must-be-a-yaml-mapping) |
| `group_by path '…': '…' is not a declared field` | [schema](schema.md#group_by-path-p-hop-is-not-a-declared-field-of-entity-e--group_by-path-p-hop-must-be-a-ref-to-continue-the-path) |
| `group_by path '…': '…' must be a ref to continue the path` | [schema](schema.md#group_by-path-p-hop-is-not-a-declared-field-of-entity-e--group_by-path-p-hop-must-be-a-ref-to-continue-the-path) |
| `id '…' does not match pattern` | [record structure](record-structure.md#id-x-does-not-match-pattern-prefix-n-digits) |
| `id prefix '…' already used by entity` | [schema](schema.md#id-prefix-p-already-used-by-entity-e--prefixes-must-be-unique) |
| `id prefix must be alphanumeric starting with a letter` | [schema](schema.md#id-prefix-must-be-alphanumeric-starting-with-a-letter-got-) |
| `id width must be a positive integer` | [schema](schema.md#id-width-must-be-a-positive-integer-got-) |
| `inline reference (…) does not resolve to any entity` | [references](references.md#inline-reference-x-does-not-resolve-to-any-entity) |
| `inline reference (…) links to '…', which does not exist` | [references](references.md#inline-reference-x-links-to-p-which-does-not-exist-relative-to-this-records-own-directory) |
| `inline reference (…) links to '…', which is not the file of` | [references](references.md#inline-reference-x-links-to-p-which-is-not-the-file-of-x--expected-q) |
| `inline reference (…) repeats its own ID as the label` | [references](references.md#inline-reference-x-repeats-its-own-id-as-the-label--redundant-write-xpath-alone) |
| `inline reference [[…]] uses unknown prefix` | [references](references.md#inline-reference-x-uses-unknown-prefix-p) |
| `inline reference ^[…] uses the legacy caret annotation` | [references](references.md#inline-reference-x-uses-the-legacy-caret-annotation-which-pandoc-reads-as-an-inline-footnote-issue-012--write-label-xpath-or-xpath-when-unlabeled) |
| `key '…' does not apply to type '…'` | [schema](schema.md#key-key-does-not-apply-to-type-t--it-would-be-silently-dead-configuration) |
| `list needs an 'items' field spec` | [schema](schema.md#enum-needs-a-non-empty-values-list--ref-needs-an-entity-target-entity-name--list-needs-an-items-field-spec) |
| `max_count_per needs a 'group_by' field or ref-path` | [schema](schema.md#unique-needs-a-non-empty-fields-list-of-field-names--max_count_per-needs-a-group_by-field-or-ref-path--max_count_per-needs-a-non-negative-integer-max) |
| `max_count_per needs a non-negative integer 'max'` | [schema](schema.md#unique-needs-a-non-empty-fields-list-of-field-names--max_count_per-needs-a-group_by-field-or-ref-path--max_count_per-needs-a-non-negative-integer-max) |
| `missing or non-string 'id' in frontmatter` | [record structure](record-structure.md#missing-or-non-string-id-in-frontmatter) |
| `missing YAML frontmatter block` | [record structure](record-structure.md#missing-yaml-frontmatter-block----------at-top-of-file) |
| `multi-word unquoted scalar '…' inside a flow collection` | [quoting](quoting.md#multi-word-unquoted-scalar--inside-a-flow-collection--a---or--in-it-is-read-as-structure-not-text-quote-it) |
| `no schema for entity '…'` | [runtime](runtime.md#no-schema-for-entity-x-known-) |
| `'path' must be a non-empty string` | [schema](schema.md#dir-must-be-a-non-empty-string-got---path-must-be-a-non-empty-string-got-) |
| `'path' must be relative to the project root` | [schema](schema.md#path-must-be-relative-to-the-project-root-without--) |
| `'pattern' is not a valid regex` | [schema](schema.md#pattern-must-be-a-string-got---pattern-is-not-a-valid-regex-) |
| `'pattern' must be a string` | [schema](schema.md#pattern-must-be-a-string-got---pattern-is-not-a-valid-regex-) |
| `policy.on_unresolvable must be one of block, relax-and-report` | [config](config.md#policyon_unresolvable-must-be-one-of-block-relax-and-report-got-) |
| `records location '…' already used by entity` | [schema](schema.md#records-location-d-already-used-by-entity-e--entity-locations-must-be-unique--records-location-d-is-nested-with-entity-es-location--entity-locations-must-not-nest) |
| `records location '…' is nested with entity` | [schema](schema.md#records-location-d-already-used-by-entity-e--entity-locations-must-be-unique--records-location-d-is-nested-with-entity-es-location--entity-locations-must-not-nest) |
| `ref needs an 'entity' (target entity name)` | [schema](schema.md#enum-needs-a-non-empty-values-list--ref-needs-an-entity-target-entity-name--list-needs-an-items-field-spec) |
| `ref targets unknown entity '…'` | [schema](schema.md#ref-targets-unknown-entity-e) |
| `reference '…' does not resolve to any entity` | [references](references.md#reference-x-does-not-resolve-to-any-entity) |
| `reference '…' points at a '…', expected '…'` | [references](references.md#reference-x-points-at-a-y-expected-z) |
| `'required' must be a boolean` | [schema](schema.md#required-must-be-a-boolean-got---any-other-value-would-be-silently-dead-configuration) |
| `required field '…' is missing or null` | [values](values.md#required-field-x-is-missing-or-null) |
| `schema entity '…' should match filename` | [schema](schema.md#schema-entity-name-should-match-filename-nameyaml) |
| `schema is not valid YAML` | [schema](schema.md#before-anything-else-the-file-has-to-parse) |
| `schema must be a YAML mapping` | [schema](schema.md#before-anything-else-the-file-has-to-parse) |
| `schema needs a non-empty 'fields' mapping` | [schema](schema.md#schema-needs-a-non-empty-fields-mapping) |
| `schema needs an 'entity' name (string)` | [schema](schema.md#schema-needs-an-entity-name-string) |
| `schema needs an 'id:' mapping with at least a 'prefix'` | [schema](schema.md#schema-needs-an-id-mapping-with-at-least-a-prefix) |
| `schemas directory not found` | [placement](placement.md#schemas-directory-not-found--is-this-an-entity-managed-project--entities-directory-not-found) |
| `'strict' must be a boolean` | [schema](schema.md#strict-must-be-a-boolean-got---any-other-value-would-be-silently-dead-configuration) |
| `these path filters matched no loaded record` | [runtime](runtime.md#these-path-filters-matched-no-loaded-record-) |
| `this tool needs Python 3.8 or newer` | [runtime](runtime.md#this-tool-needs-python-38-or-newer-the-python-running-it-is-) |
| `this tool needs the PyYAML library` | [runtime](runtime.md#this-tool-needs-the-pyyaml-library-and-the-python-running-it-cannot-find-it) |
| `unique needs a non-empty 'fields' list` | [schema](schema.md#unique-needs-a-non-empty-fields-list-of-field-names--max_count_per-needs-a-group_by-field-or-ref-path--max_count_per-needs-a-non-negative-integer-max) |
| `unique references undeclared field '…'` | [schema](schema.md#unique-references-undeclared-field-f) |
| `unknown constraint key '…'` | [schema](schema.md#unknown-constraint-key-key) |
| `unknown field '…' not declared in schema` | [record structure](record-structure.md#unknown-field-x-not-declared-in-schema-type) |
| `unknown field key '…'` | [schema](schema.md#unknown-field-key-key) |
| `unknown id key '…'` | [schema](schema.md#unknown-id-key-key-valid-prefix-width) |
| `unknown or missing rule '…'` | [schema](schema.md#unknown-or-missing-rule-r-valid-max_count_per-unique) |
| `unknown or missing type '…'` | [schema](schema.md#unknown-or-missing-type-t-valid-boolean-date-datetime-enum-integer-list-number-ref-string-text) |
| `unknown policy key '…'` | [config](config.md#unknown-policy-key-key-valid-on_unresolvable) |
| `unknown schema key '…'` | [schema](schema.md#unknown-schema-key-key-valid-constraints-description-dir-entity-fields-filename-id-path-strict) |
| `unquoted value contains ' #'` | [quoting](quoting.md#unquoted-value-contains----from-the--on-yaml-reads-a-comment-and-the-tail-silently-vanishes-from-the-value-quote-the-value-to-keep-it-or-to-make-the-comment-unambiguous) |
| `value '…' does not match pattern` | [values](values.md#value-x-does-not-match-pattern-) |
| `value '…' not in enum` | [values](values.md#value-x-not-in-enum-a-b-c) |
| `value … above max` / `value … below min` | [values](values.md#value-n-below-min-m--value-n-above-max-m) |
| `value looks like '#'-glued content` | [quoting](quoting.md#value-looks-like--glued-content-but-yaml-reads-a-comment-the-value-is-null-and-the-text-is-gone-quote--if-it-was-meant-as-the-value-a-comment-puts-a-space-after-) |
| `value looks like '#'-glued text` | [quoting](quoting.md#value-looks-like--glued-content-but-yaml-reads-a-comment-the-value-is-null-and-the-text-is-gone-quote--if-it-was-meant-as-the-value-a-comment-puts-a-space-after-) |
