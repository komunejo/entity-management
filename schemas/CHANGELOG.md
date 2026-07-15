# Schema changelog

Every schema modification — by human or agent — gets an entry here. Never in silence (REQ-006).

## 2026-07-15 — issue schema for what the project is told about itself (Claude (claude-opus-4-8), with Humano)

- Created `issue.yaml` (prefix ISSUE): title, status enum [open, resolved, rejected], date, channel (string, required), resolved_by (ref decision), tags.
- Issues are the register: a defect reaching the project by any channel is recorded here, since the GitHub tracker only ever sees what arrives through GitHub (DEC-015). `channel` is a string rather than an enum because the set of channels is deliberately open. First record: ISSUE-001, the line-endings defect resolved by DEC-014.

## 2026-07-13 — proposal schema for desired improvements (Claude (claude-fable-5), with Humano)

- Created `proposal.yaml` (prefix PROP): title, status enum [proposed, accepted, rejected, implemented], date, addresses (list of ref requirement), tags.
- Proposals are the backlog: notes on desired improvements accumulate as records before any design commitment. First record: PROP-001, schema inheritance.

## 2026-07-12 — initial schema set (Claude, with Humano)

- Created `decision.yaml` (prefix DEC): title, status enum [proposed, accepted, superseded, rejected], date, supersedes (ref decision), addresses (list of ref requirement), tags.
- Created `requirement.yaml` (prefix REQ): title, status enum [open, satisfied, dropped], priority enum [must, should, could], date, tags.
