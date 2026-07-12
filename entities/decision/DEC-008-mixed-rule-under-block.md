---
id: DEC-008
entity: decision
title: Mixed rule defines "unresolvable" under the block policy
status: accepted
date: 2026-07-12
addresses: [REQ-004, REQ-006]
tags: [integrity, governance, workflow]
---

Under `block`, a violation whose repair requires no new fact — a broken ref in an optional field, a dangling inline `[[ID]]` — is repaired by demoting the claim to prose with a note and a pending question (green). Red is reserved for violations that require a fact nobody has, such as a missing required field. This keeps red meaningful: every red item is a real question for a human, not a typo.

Origin: iteration-2 evaluation. The iteration-1 repair agent demoted the ghost mentor and the dangling tool link to prose (green); the iteration-2 agent left both red awaiting a human. Both were defensible because [[DEC-007]] had not defined the boundary; this decision defines it.
