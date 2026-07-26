# Firing the validator automatically

A linter that exists but is never fired is documentation, not integrity. This project uses four firing points (see DEC-005 in the skill's own decision log): a post-edit Claude Code hook, a session-start Claude Code hook, explicit workflow instructions, and git pre-commit — the last being the only one that fires on a hand edit made with no agent at the keyboard.

## 1. Claude Code hook (PostToolUse)

Add to the project's `.claude/settings.json` (create the file if absent). This runs the validator every time the agent writes or edits a file in the project, surfacing violations while the offending edit is still in working memory — the cheapest possible moment to fix it. The redirection and the exit code matter: Claude Code only feeds hook output back to the agent when the hook writes to **stderr** and exits with code **2** — a plain exit 1 with the report on stdout is invisible to the agent, which would defeat the whole point:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "python3 entity-management/scripts/entity_lint.py validate --root . --json 1>&2 || exit 2"
          }
        ]
      }
    ]
  }
}
```

Adjust the script path to wherever the skill/engine actually lives (skill installs are usually outside the project — use an absolute path then). On failure the agent receives the machine-readable violation list on the feedback channel; fix the violations before continuing, and never work around the hook.

The `--json` payload the hook (or any script) can parse: `{"engine_version", "ok", "errors", "warnings", "entities", "schemas", "issues": [{"level", "file", "message", "field"?}]}`. Engine exit codes: 0 = integrity holds, 1 = violations found, 2 = the run itself was invalid (no project root, missing PyYAML, a path filter matching nothing) — never read 2 as "no violations".

If the project only wants validation when entity files change, pass the changed file as a positional argument (`validate <file> --root .` restricts record checks to that file while aggregate constraints still run project-wide), or simply accept the cheap full run: on projects of hundreds of entities the validator takes well under a second.

## 2. Session-start validation (SessionStart hook)

Open the session with a full validation, and the question "whose edit broke this?" never needs asking again: a flag raised at session start is by construction nobody-in-this-session's edit — inherited state, which should be rare, since every clone runs the same configuration and the same checks — while from a green start every later flag belongs to the session's own work. Add alongside the PostToolUse hook:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 entity-management/scripts/entity_lint.py validate --root . --json 1>&2 || exit 2"
          }
        ]
      }
    ]
  }
}
```

What the agent does with a flag — at session start or any other firing point — is tiered (PROP-006): resolve it yourself first, because the likeliest cause is your own involuntary slip and the fix is one edit and a green revalidate; the human never hears of it. Criterion includes the conversation — if the session's context explains the flag, resolve with that. Only when you honestly cannot — the flagged text is not yours and you have neither knowledge nor standing to decide what its author meant — does the flag become one question to the human. The per-family triage is [`troubleshooting.md`](troubleshooting.md).

## 3. Conversational triggers (always available)

The skill's workflows already instruct the agent to validate after creating records, after migrations, and before reporting completion. In addition, treat phrases like these from the user as a direct instruction to run `validate` and report the results:

- "check the structural consistency of the documents"
- "is the registry consistent?" / "validate the entities"
- "is everything in the records still in a valid state?"

The same applies to equivalent requests in any other language. Run the engine, then relay its findings — never answer from memory.

## 4. Git pre-commit

The layer that stands between a hand edit and the history: the two hooks above fire only when an agent is at the keyboard, and a human editing records directly bypasses both. Any repo whose owner has hands wants this one, not only teams:

```bash
#!/bin/sh
# .git/hooks/pre-commit  (chmod +x)
# Adjust the engine path if the skill is installed outside the repository.
python3 entity-management/scripts/entity_lint.py validate --root . || {
  echo "entity_lint: commit blocked — fix integrity errors first." >&2
  exit 1
}
```

Not installed by default; offer it when the project lives in git and the user wants nothing unverified entering the history.
