---
id: ISSUE-013
entity: issue
title: "the ' #' rule fires on benign comments, and for typed fields its remedy is impossible"
status: resolved
date: 2026-07-26
channel: report (correspondence, 2026-07-26 reply to the 0.6.1 release letter)
resolved_by: [DEC-023]
tags: [engine, scan, false-red, flag-quality]
---

Reported from the field, in correspondence, by the same reporter whose July 14 report produced [the documented install path fails on an externally-managed Python (ISSUE-002)](ISSUE-002.md) and [the unquoted comma is one of a class — silent misparses the parse itself hides (ISSUE-003)](ISSUE-003.md) — a space that upgraded to 0.6.1 the day it shipped. The formulations quoted below are the reporter's.

The pre-parse scan's ` #` rule ([the engine reads raw text before the parse, and flags interrupt the agent before the human (DEC-020)](../decision/DEC-020.md)) fires on *every* trailing comment on an unquoted scalar, not only the ones that truncate something. Correct YAML that had sat in the reporting space's config since July goes red on upgrade:

```yaml
policy:
  on_unresolvable: block   # honest red; never silently loosen the contract
```

The value parses as `block` either way — "the comment was doing exactly what a comment does." A false red is this project's chosen safe failure, so the firing alone would be a calibration question. What makes it a defect is the remedy. DEC-020's design rests on the claim that the remedy the scan asks for "is always harmless"; on a typed non-string field that claim is false, and the message's own advice moves the project from one red to another:

```yaml
count: 3  # how many      ->  ERROR: unquoted value contains ' #' ... quote the value to keep it
count: "3"  # how many    ->  ERROR: [count] expected integer, got str '3'
```

"Taking the advice literally moves a project from one red to another; the only real fix is deleting the comment." The reporter's proposed narrowing, recorded as the candidate design: for `integer`, `boolean` and `date` fields the scan can know that quoting is not available to the author; and the reporter suspects the genuinely ambiguous case is narrower still — a plain scalar that already contains whitespace, where the tail could plausibly be content.

Why the measurements missed it: this repository's house style puts comments on their own line above the key, so the trailing-comment pattern never entered the sample — the reporting space's style puts them after the value, "which is common enough that I doubt we're the last to hit it." One style measured, presented as the class: the same shape of error [the reference annotation imitates a footnote mark, and footnote-aware renderers take it at its word (ISSUE-012)](ISSUE-012.md) records against DEC-017's "measured rather than assumed," arriving here through the scan's own evidence base.

**Resolved by [the scan speaks only where the schema leaves it honestly blind (DEC-023)](../decision/DEC-023.md), which took the criterion further than the report's candidate.** The reporter's typed-field narrowing became the general rule — the scan is silent wherever the schema (or the engine's own documented contract, `policy.on_unresolvable` included) makes a comment tail decidable, with the typed check downstream as the honest red — while the report's suspected single-word narrowing was declined as the engine guessing what it cannot know. The review's own inventory then surfaced the `#`-glued value reading as null — flagged now on every field kind, since null escapes every downstream check — while its second candidate (indented comments swallowing a plain-scalar continuation) was implemented and withdrawn when adversarial measurement showed it red on idiomatic YAML; the false premise the report exposed ("the remedy is always harmless") is corrected in place in [PROP-006](../proposal/PROP-006.md) and [DEC-020](../decision/DEC-020.md). The map closes without new schema language: string fields whose values have shape declare it with the `pattern` the language always had — which the scan now reads, deciding per instance — and shapeless prose is the case the golden rule already orders quoted.
