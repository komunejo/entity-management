---
id: ISSUE-003
entity: issue
title: the unquoted-comma diagnostic names the phantom key, not the comma
status: open
date: 2026-07-15
channel: report
tags: [engine, schema-language]
---

An unquoted comma inside an inline flow mapping — a `description` ending in `, if any` — makes YAML read the tail as a second key. Meta-validation then rejects `unknown field key 'if any'`, which is entirely correct and nearly useless: it names what YAML produced, not what the author typed. The author is looking for a field they never wrote. A hint in the shape of *this looks like an unquoted comma in a flow scalar* would close the distance between a true message and a usable one.

The pitfall is also absent from the YAML-pitfalls list in `references/schema-language.md` — the section that exists precisely to name the reasons YAML and Markdown need a linter at all. It covers `title: yes`, quoted dates, `version: 1.0` and tabs; it says nothing about commas.

Worth being exact about what failed and what did not: the engine caught the problem, reported it honestly, and refused to guess. Nothing about hard integrity is at stake. This is the softer edge — a diagnostic that is right without being helpful, which costs a reader the minute the checker was supposed to save.
