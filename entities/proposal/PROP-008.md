---
id: PROP-008
entity: proposal
title: "state the requirements, don't ship them: install guidance at household scale"
status: implemented
date: 2026-07-26
tags: [runtime, portability, documentation]
---

## Motivation

[The documented install path fails on an externally-managed Python (ISSUE-002)](../issue/ISSUE-002.md) reported the surface: `pip install pyyaml`, the command the engine itself prints, is refused outright on a PEP-668 platform. Working the issue exposed the deeper misdirection: **the actor reading that instruction is almost never the human — it is an agent**, and the instruction assumes permissions and judgment calls that are the human's, not the agent's. The report's own history proves it: the wall was hit not by the reporting human but by his agent, which then had to improvise a venv and a wrapper.

The user casuistry the fix must serve, in order of frequency:

- **An almost-ignorant user on Claude Desktop** who pastes the repo URL and says "install this." Their agent must get to a working engine without one scary word reaching the conversation. This is the case that decides the register — and it is softer than the worst case suggests: observed in use (on macOS, presumably identical on Windows) and documented officially, Claude's desktop environments run the agent's work in an isolated environment of its own, where satisfying the runtime is the agent's ordinary work and no human step exists at all. The details of what that environment provides belong to the platform's documentation (the Claude Help Center), which the skill references instead of replicating — the same stated-not-shipped principle this proposal applies to the dependency itself. The one guided ask is for agents operating directly on a host.
- **A competent user's agent**: assumed to operate under its principal's standing instructions — it has likely handled files and environments like these before, and the human has already told it how such things are done in that house. The engine's job here is to need nothing beyond those conventions: it neither adds steps to them nor fights them.
- **Any human acting alone**, without an agent: must find an honest, calm statement of what is needed and where the authoritative sources are.

Machine casuistry, reduced to the practical: macOS and Windows ship no real Python — macOS has a stub that offers the Command Line Tools (and `git` already forces that install on any machine that clones), Windows has a Store decoy on the factory PATH. Linux ships a real Python but marks it externally-managed (Debian 12 / Ubuntu 23.04 on), so `pip install` outside an environment is refused there too. In every branch, the step that trips is *obtaining the dependency* — never running the script — and in every branch the existing message aims the wrong remedy at the wrong actor.

## Sketch

The engine is a household tool, not a platform. The fix is not to engineer the world's package problems away but to **state the requirements honestly, in a register that never flips the machine into "professional developer aboard" mode**, and to point at the sources that are actually responsible for each piece.

- **Requirements, stated once and gently**: a Python ≥ 3.8 and the PyYAML library. One line in the README, with pointers instead of procedures: python.org for Python, the PyYAML project (pypi.org/project/PyYAML, github.com/yaml/pyyaml) for the library, the OS vendor's support pages for anything machine-specific. No platform matrix to maintain — the world's parts of the problem are documented and pointed at, not fixed. Requirements are stated at their source of truth, not shipped: a vendored or auto-downloaded copy is code nobody here has the job of keeping current.
- **Version floor checked first.** The script verifies its Python floor *before* importing anything and dies plainly when unmet — a version message, not a syntax error.
- **Messages written to be read aloud.** A script error frequently lands verbatim in a novice's conversation, so the engine's user-facing failures carry no `PEP-668`, no `externally-managed`, no tracebacks up front, and no single-command recipe that half the platforms reject. "This tool needs the PyYAML library and can't find it; ask your agent to set it up, or see the requirements section" — technical diagnosis behind, marked as such. A message that frightens the almost-ignorant user is an engine defect of the same rank as a false red.
- **SKILL.md addresses the agent**: verify the interpreter is real before invoking it (the Windows Store decoy answers to `where python` but has no version; on macOS, `xcode-select -p` detects the Command Line Tools without triggering their GUI dialog — invoking `python3` blind would hang a non-interactive agent on a popup). Obtaining PyYAML is the agent's ordinary environment work, done per its house rules — a venv it owns, the distro's packaged `python3-yaml`, whatever the principal has taught it — under two absolutes: **per-user scope and only-if-missing**. Never machine-wide, never elevated, never duplicating something present. When a human step is unavoidable (Windows without Python: Store or python.org, "just for me"), the agent asks for exactly one thing, in plain words, and waits.
- **Humans can act too**: a short section in `docs/troubleshooting.md` (the guide that is already for humans) covering the two honest failure stories — "no usable Python" and "Python but no PyYAML" — each ending in where to look, not in a recipe that pretends one command fits every machine.

## Paths not taken

- **Vendoring PyYAML** (carrying its source inside the repo). Rejected for obsolescence: a bundled copy is code nobody has the job of updating — the promised "re-vendor now and then" is exactly the unowned discipline that does not happen at household scale — and the rot runs both ways (a frozen copy can break on a future Python with no one watching). The library each machine obtains from its source stays maintained by the people whose job that is.
- **A homegrown subset parser** (stdlib-only, dependency deleted). Rejected as reinventing the wheel: today's simple grammar is a snapshot, not a contract, and every future space that needs a little more YAML would turn the engine's maintainers into YAML-parser maintainers — the work PyYAML already did, with twenty years of corners resolved.
- **Download-on-first-use by the engine** (the Composer model: the *script itself* carries a downloader that fetches the library when absent). Easily confused with what this proposal adopts, so the boundary deserves stating: in both designs PyYAML arrives only when missing — the difference is who obtains it. Here it would be *our code*, running inside the engine, talking to the network with no house conventions to lean on; its failures (certificate trust, proxies, half-fetched archives) are the most cryptic kind for the least equipped user, and the downloader is ours to maintain forever. The adopted design leaves the obtaining to the *agent*, which does the same thing with standard tooling, under its principal's conventions, and can explain itself when something goes wrong. The Composer model is sound at another scale — dozens of packages no agent should be improvising — just not at this one.
- **Documentation as platform matrix** (per-OS install tables, venv walkthroughs). Rejected for register: it drags a household tool's README into sysadmin territory, goes stale per platform, and duplicates the vendors' own documentation — which the pointers already reach.

## Open questions

Both were settled by the implementation, [DEC-021](../decision/DEC-021.md):

- Where does the engine's missing-dependency message point? Only at targets that travel with the skill folder: the library's official page. No repo paths — the README and `docs/troubleshooting.md` exist in this repository, not in the consuming spaces the skill is copied into.
- Does [DEC-003](../decision/DEC-003.md) need touching? It stands as written; DEC-021 fixes its reading — "the only dependency" means the only *stated requirement*, never a copy the engine carries or fetches.
