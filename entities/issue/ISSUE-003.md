---
id: ISSUE-003
entity: issue
title: the unquoted comma is one of a class — silent misparses the parse itself hides
status: resolved
date: 2026-07-15
channel: report
resolved_by: [DEC-020]
tags: [engine, schema-language, integrity]
---

An unquoted comma inside an inline flow mapping — a `description` ending in `, if any` — makes YAML read the tail as a second key. Meta-validation then rejects `unknown field key 'if any'`, which is entirely correct and nearly useless: it names what YAML produced, not what the author typed. The author is looking for a field they never wrote. A hint in the shape of *this looks like an unquoted comma in a flow scalar* would close the distance between a true message and a usable one.

The pitfall is also absent from the YAML-pitfalls list in `references/schema-language.md` — the section that exists precisely to name the reasons YAML and Markdown need a linter at all. It covers `title: yes`, quoted dates, `version: 1.0` and tabs; it says nothing about commas.

Corrected 2026-07-26, and per [a record that is wrong is corrected in place, and the correction names the error](../decision/DEC-019.md)^[DEC-019](../decision/DEC-019.md), the correction names what the first version got wrong: it read the failure as a diagnostic-quality problem and closed with "nothing about hard integrity is at stake." That sentence was true of the case reported and false of the class the case belongs to. The reported comma was the benign member — the phantom key happened to be unknown, so meta-validation fired. The malignant members parse clean and validate green.

Measured on 2026-07-26, the class has four verified members. A comma in a flow scalar whose tail spells a *legal* key — `{description: applies always, required: true}` — truncates the description and sets `required` without anyone having written it, no error anywhere. A `#` preceded by a space in an unquoted scalar, block style included — `title: count # per item` — silently drops the tail as a comment. Duplicate keys in one mapping keep the last value and discard the rest, silently. And a line reading `---` inside a multi-line frontmatter value — legal YAML in a block scalar, even indented, since the splitter compares stripped lines — ends the frontmatter right there, in the engine's own `split_frontmatter`, before PyYAML ever runs: the value loses its tail and every field below the line falls silently into the document body. That member widens the class's definition: the destroyer is not only the YAML parse but any layer that reads the raw text ahead of the checks — the engine's frontmatter split included. None of these can be caught downstream even in principle: by the time the checks run, the typed text — the evidence — is gone. The list is open; membership is what has been measured to misparse silently, not a claim of completeness.

The hint this record originally asked for covers the benign member only. The class wants the pre-parse raw-text scan proposed in [raw-text scan: pre-parse checks for what the parse destroys](../proposal/PROP-006.md)^[PROP-006](../proposal/PROP-006.md); the missing entries in the pitfalls list remain the writer-side half of the fix.
