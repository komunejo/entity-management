---
id: REQ-006
entity: requirement
title: Contract changes are never silent
status: open
priority: must
date: 2026-07-12
tags: [core, integrity, governance]
---

Whether the human or the agent modifies a schema, the change is documented — an entry in `schemas/CHANGELOG.md` and a mention in the report. A silently negotiated contract is worse than a violated one, because it corrupts trust in every green validation that follows it. NEVER in silence.
