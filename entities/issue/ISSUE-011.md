---
id: ISSUE-011
entity: issue
title: entity scope — machine-private types and the no-inbound-references guarantee
status: open
date: 2026-07-26
channel: use (keikoba space, constitution session)
tags: [schema-language, scope, privacy, references]
---

Sibling of [ISSUE-010](ISSUE-010.md): both stem from the engine assuming one shared view of all records. That issue is about *placement* (type is not folder); this one is about *scope* — which records exist on every clone, and which exist only on the machine that wrote them.

## The case

The keikoba space keeps developer notebooks as machine-private content: notebook-note records are contracted and validated like any record, but by the project's decision they never travel — the repository ships the folders' doors, not their content. Today that scope exists only in `.gitignore`, a layer the engine knows nothing about. The engine validates the author's clone, sees every ref resolve, and reports green — while the records it validated are invisible to every other clone.

## What scope must mean, if the schema can declare it

A schema-level scope declaration (working name: `scope: repo | machine`, or `local: true`), from which the engine derives three guarantees:

1. **No inbound references.** No record outside the private scope may reference a private record — frontmatter refs and prose refs alike. Without this rule the failure is perverse: on the author's machine the ref resolves and validates green; the red fires on every *other* clone, after the offending record has traveled. The one machine that could prevent the break is the one where the validator sees nothing wrong. (Private-to-private and private-to-shared refs stay legal; only the crossing from shared into private is broken by construction.)
2. **Scope-aware derived views.** A generated view that travels must not enumerate private records. The reporting space hit this concretely: the generated index listed every notebook note's title and path — thinking leaked by a derived view — and the interim fix (the index itself made per-clone, regenerated locally) works but is convention, not contract.
3. **ID allocation scoped accordingly.** Private types allocate IDs from the clone's own view, so the same ID may exist on different machines. That is harmless precisely and only because of guarantee 1 — no shared record can name a private ID, so no shared namespace ever forms. The design should state this dependency: per-clone sequences are sound *given* the no-inbound-references rule, unsound without it.

## Scope as the source of the ignore configuration

Today the scope lives twice: declared nowhere, enforced by hand-written `.gitignore` patterns. Once the schema declares it, the ignore configuration becomes *derived* — and the derivation has a precise limit worth designing around. Path patterns are non-deterministic under mixed folders: a folder can host different Markdown types distinguishable only by their YAML (the [ISSUE-010](ISSUE-010.md) mixed-content case), and no pattern language reads frontmatter. But per-file resolution *is* deterministic: the engine knows each record's declared type, hence its scope, hence exactly which files must not travel. The design space: an engine-maintained exclusion block of exact paths (regenerated like any derived view), or a travel audit that flags both mismatch directions — a machine-scoped record that is not excluded (it would leak), and a repo-scoped record sitting in an excluded tree (it would silently not exist on any other clone, with no red anywhere until a ref dangles).

## Acceptance, from the reporting space's side

A schema can declare its records machine-private; the validator then rejects, on the author's machine and before anything travels, any shared-scope record that references a private one; and whatever views the engine generates for traveling consumption exclude private records. The space's own closing discipline (a pre-commit review check) covers this today; the point of the issue is to move it from discipline to contract, where it fails on the machine that can still fix it.
