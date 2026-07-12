# Schema changelog

Every schema modification — by human or agent — gets an entry here. Never in silence (REQ-006).

## 2026-07-12 — initial schema set (Claude, with Humano)

- Created `decision.yaml` (prefix DEC): title, status enum [proposed, accepted, superseded, rejected], date, supersedes (ref decision), addresses (list of ref requirement), tags.
- Created `requirement.yaml` (prefix REQ): title, status enum [open, satisfied, dropped], priority enum [must, should, could], date, tags.
