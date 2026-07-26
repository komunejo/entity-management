---
id: ISSUE-002
entity: issue
title: the documented install path fails on an externally-managed Python
status: resolved
date: 2026-07-15
channel: report
resolved_by: [DEC-021]
tags: [runtime, portability]
---

On a current macOS with a Homebrew Python 3.14, `pip install pyyaml` is refused outright: PEP-668 marks the interpreter externally-managed, and `--user` is blocked with it. The engine hands out that exact command itself — `entity_lint: PyYAML is required. Install with: pip install pyyaml` — so the program tells the user to run something the platform rejects. `SKILL.md` and `references/hooks.md` compound it by invoking a bare `python3 .../entity_lint.py`, which assumes PyYAML is importable from the system interpreter. Out of the box, on that platform, none of it works.

The workaround reached independently was a dedicated venv plus a small wrapper. Suggested: a documentation note at least, or a shipped venv/uv path.

This does not put [engine runs on Python 3.8+ with PyYAML as the only dependency (DEC-003)](../decision/DEC-003.md) in question — one dependency, PyYAML, is not what broke. What broke is the unstated assumption underneath it: that a dependency can be installed into the interpreter the documentation then tells you to invoke. On a growing share of systems it cannot.

Resolved by retargeting the remedy, not fighting the platforms: the actor reading the engine's instruction is almost always an agent, so the message and the docs now address it as such — obtain PyYAML as ordinary environment work (per-user, only if missing, a venv of its own always works: exactly the workaround this report reached by improvisation, now the documented path) — while humans get requirements stated once with pointers to the official sources. [DEC-021](../decision/DEC-021.md) records the design.
