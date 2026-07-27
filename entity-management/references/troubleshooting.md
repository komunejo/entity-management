# Triage: what to do when the validator flags something

This is the agent's side of error handling. The human's side — every message explained with examples, for someone reading raw engine output — does not travel with the skill: it lives in the skill's repository, github.com/komunejo/entity-management, under `docs/`. When a human needs the mechanics of an error, send them that address or quote the passage that covers it, rather than paraphrasing YAML rules at them. This page is about something else: **deciding who a flag belongs to**, fixing your share silently, and asking the human exactly one well-formed question when — and only when — a flag is genuinely theirs.

## The protocol

Every flag gets the same three steps, in order (PROP-006 in the skill's own registry; the firing points are in [`hooks.md`](hooks.md)):

1. **Claim it.** From a green session start, a mid-session flag lands on work this session just did. If the flagged text is your own edit, the likeliest cause is your own slip: fix it, revalidate, and say nothing — the human never hears of a typo you made and unmade.
2. **Resolve it with criterion.** Even when the text is not yours, the session often holds the answer: the human dictated the sentence (so the comma is prose — quote it), the intent was stated, a criterion was recorded in an earlier session. Resolving with criterion is not guessing; if you can cite where the answer comes from, act on it.
3. **Escalate what is honestly yours to escalate.** Only when the flagged text is not yours *and* nothing in the conversation or the project's recorded criteria decides it, ask the human. Ask about **intent, never mechanics**: "is `count # per item` all title, or is `per item` a comment?" — not "how should I quote this?". Bundle: if one validate run raised five flags you cannot resolve, that is one message with five questions ranked, never five messages. And never relay raw engine output as the question — translating the red into the decision it actually needs is your job, not theirs.

A useful mirror for step 3: an agent question that the human can only answer by learning YAML was the wrong question. If they need the mechanics anyway, that is what the human guide linked above is for.

## Criteria accumulate — record them

When the human's answer settles a *rule* rather than an instance — "titles in this project are always literal text, quote them all", "that `#` convention is ours, keep it", "unknown fields in `capture` records are emergent types, not typos" — the answer has become a project criterion, and the same question must never be asked twice. Record it where this project keeps its conventions: its own `CLAUDE.md` or agent-instructions file if it has one, or a decision record if the project tracks decisions as entities. An unwritten criterion is a question in waiting; writing it is what turns this protocol from politeness into memory. (The scope test from the global discipline applies: the criterion belongs to the *project* and travels with its files — not to your session memory, which the next agent never sees.)

## Before flags exist: the runtime family (exit 2)

Exit 2 means the engine could not even look — no project root, a path filter matching nothing, or the runtime itself missing. The runtime cases are **yours, silently, with exactly one exception**:

- **PyYAML missing**: your ordinary housekeeping, per SKILL.md's Runtime section — obtain it per-user, only if missing, never elevated; a virtual environment of your own always works. The human never hears of it. Never relay the engine's message as if it were a question for them.
- **Exit 2 from a hook that works by hand**: the hook is invoking the wrong interpreter (a bare `python3` while PyYAML lives in your environment) — fix the hook command ([`hooks.md`](hooks.md)), not the environment.
- **No usable Python at all** (typically Windows): the one exception, because installing it is genuinely the human's step. One ask, in plain words — install Python from python.org or the Store, "just for me" — and wait. No jargon, no error dumps: to an almost-ignorant user, a scary message is a defect, and that includes the messages you compose.

## Triage by family

What follows sorts the engine's message catalog by who usually owns the fix. "Usually" is a prior, not a verdict — step 2 (criterion) overrides it in both directions.

**Yours to fix silently, almost always:**

- The quoting family (`multi-word unquoted scalar …`, `unquoted value contains ' #' …`, `value looks like '#'-glued …`, `duplicate key …`) — when the text is your own edit. You know what you meant; make the text say it and revalidate. When the text is a hand edit, the same flags become intent questions (see below). The comment flags fire only where no declaration decides them — and when a flagged field's values have a shape (codes, slugs, hashtags), the durable fix is not a quote per line but declaring `pattern`: yourself, when a citable criterion settles it (documented in `schemas/CHANGELOG.md` and reported); as your one intent question, when none does. The full reasoning and routes: [`comment-decidability.md`](comment-decidability.md).
- `filename should be the id …` on a file you created; `frontmatter entity 'x' does not match …` after a move you made; any reference error caused by your own rename or move — the `expected '…'` form of the prose-ref message contains the correct path, ready to apply.
- Type errors whose fix is mechanical and meaning-preserving: `expected string, got bool` on a `yes` you wrote unquoted; a quoted date in a `date` field. Quote or unquote; nothing about the record's meaning is at stake.
- `duplicate id` where the session's own history shows which record was created when — the newer copy takes the next free ID from `new <type>`.

**Resolvable with conversation or recorded criteria, usually:**

- `value 'x' not in enum […]` — if the human said "mark it done" and the enum says `resolved`, the mapping is criterion, not invention; apply it. A genuinely new state is a schema evolution: propose it, don't smuggle it.
- `required field 'x' is missing or null` — mine the conversation and the record's own prose body first; a fact stated anywhere citable is yours to fill. A fact nobody has stated is the human's, and possibly nobody's — see the contract policy in `SKILL.md` ("When the contract cannot be satisfied"): red can be the honest deliverable, and fabricating a value to reach green is the one absolute prohibition.
- `unknown field 'x'` — check for the phantom-field case first (an unquoted comma one line up manufactures fields; the pre-parse scan usually names the real error). A typo you can see, fix. A field that looks deliberate is either a schema evolution to propose or a criterion to ask for — once.

**The human's, almost always:**

- The quoting family **on hand-edited text**, when no stated intent decides it: only the author knows whether the comma was prose or structure. One intent question.
- Aggregate constraint violations (`unique`, `max_count_per`) — the fix is a domain decision (merge, retire, move, or evolve the cap), never mechanical.
- Anything under the contract policy: unresolvable records, loosening a schema, retiring an entity that other records reference.
- `file is not inside any declared entity directory` for a file you did not put there — a hand-placed stray is a placement *intention* you cannot read from the path alone.

**Never yours, under any criterion:**

- Turning red green by inventing a value, deleting a record you were not asked to delete, or changing a schema silently. Every schema change gets its `schemas/CHANGELOG.md` entry and a mention in your report, even when the human made the change and you merely noticed it.

## Session start: the inherited case

The session-start validation ([`hooks.md`](hooks.md), firing point 2) exists so the tier test stays trivial: a flag raised at start is by construction nobody-in-this-session's edit. It should also be rare — every clone runs the same configuration and checks, so inherited red means something upstream skipped its own gate. Apply the same protocol, with one adjustment: at session start the human has the most attention to spare, so an inherited flag you cannot resolve is asked about *now*, before new work piles on top of old red — not parked until it collides with the session's own edits.
