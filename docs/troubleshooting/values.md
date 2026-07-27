# Types and values

Each field in a schema declares what kind of value it holds — text, a whole number, a date, one of a fixed list, a reference to another record. These messages appear when a value is not of the declared kind, or is outside the range or shape the schema allows.

Most of them are one character away from resolved. The awkward ones are awkward for a single reason worth knowing up front: **YAML converts some unquoted words into things that are not words.**

```yaml
title: yes          # not the word "yes" — the boolean true
title: 2026-07-12   # not that text — a date
title: 7            # not that character — the number seven
```

Quotation marks are what say “this is text”. Everything below is a variation on knowing when you need them and when you must *not* use them.

## `expected <type>, got <something> …`

The value's actual kind does not match the declaration. The message often carries its own diagnosis in parentheses:

- **`(YAML parses yes/no/true/false as boolean — quote it if you meant text)`** — you wrote `yes` in a text field and stored the boolean `true`. Write `"yes"`. This also catches `no`, `on`, `off`, `y`, `n` in some dialects, which is why the habit of quoting prose is worth having.
- **`(YAML parsed this as a date — quote it if you meant text)`** — the mirror case: an unquoted `2026-07-12` in a *string* field became a date object. Quote it.
- **`expected date (YYYY-MM-DD, unquoted), got str …`** — and the mirror of the mirror. A `date` field wants a real date, so it wants the bare `2026-07-12` with no quotation marks. Here the quotation marks are the error.
- **`expected integer, got str '7'`** — quoted numbers are text. Remove the quotation marks; or, if the value really is a label that happens to look numeric (a room number, a version), change the field's type in the schema to `string`.

The rule underneath: **quote what is text, leave bare what is a number, a date, or a yes/no.**

## `required field 'x' is missing or null`

The schema promises that every record of this type carries `x`, and this one does not — either the field is absent, or it is there and empty (`x:` with nothing after it is null, not blank text).

Fill it from what you know. If nobody knows the value, that is not a formatting problem and no amount of editing will make it one; it is the case the project's contract policy exists for, and [when red is the honest answer](troubleshooting.md#when-red-is-the-honest-answer) is what governs it.

What is never acceptable is inventing a plausible value to turn the red green. A fabricated green costs more than any red, because it spends the trust that every other green depends on.

## `value 'x' not in enum ['a', 'b', 'c']`

The field admits only the listed values, and this is not one of them. Two very different causes:

- **A variant of a legal value** — `Open` for `open`, `in progress` for `in-progress`. Case and punctuation both count. Fix the value.
- **A genuinely new state.** Your records have grown a situation the schema did not anticipate. That is schema evolution, not a typo: add the value to the field's `values` list and log the change in `schemas/CHANGELOG.md`.

The distinction matters because the first fix is mechanical and the second is a decision about what your registry now tracks.

## `value 'x' does not match pattern '…'`

The schema declares the *shape* this field's values take, as a regular expression, and the value falls outside it.

Two things to know before you go changing the value. The pattern is a **full match**: the whole value must satisfy it, not some fragment of it. And a pattern is usually there for a reason — an ID format, a code, a slug convention — so a value that violates it is often a real slip rather than an over-strict rule.

If the rule itself has become wrong (the project now uses a code format the pattern predates), the pattern is what changes, in the schema, with a note in `schemas/CHANGELOG.md`.

## `value N below min M` / `value N above max M`

A numeric field with declared bounds, and the value is outside them. Either the value is wrong, or the bound has gone stale — the same question as the pattern above, and the same two places to fix it.

## Values inside lists

A list field validates every item against the same declaration, and reports each one by position:

```
ERROR   entities/course/CRS-002.md [students[2]]: expected string, got int 4
```

`students[2]` is the third item (positions start at zero). Only that item is at fault; the rest of the list is fine.
