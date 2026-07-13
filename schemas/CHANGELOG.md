# Schema changelog

Every schema modification — by human or agent — gets an entry here. Never in silence (REQ-006).

## 2026-07-13 — proposal schema for desired improvements (Claude (claude-fable-5), with Humano)

- Created `proposal.yaml` (prefix PROP): title, status enum [proposed, accepted, rejected, implemented], date, addresses (list of ref requirement), tags.
- Proposals are the backlog: notes on desired improvements accumulate as records before any design commitment. First record: PROP-001, schema inheritance.

## 2026-07-12 — initial schema set (Claude, with Humano)

- Created `decision.yaml` (prefix DEC): title, status enum [proposed, accepted, superseded, rejected], date, supersedes (ref decision), addresses (list of ref requirement), tags.
- Created `requirement.yaml` (prefix REQ): title, status enum [open, satisfied, dropped], priority enum [must, should, could], date, tags.
