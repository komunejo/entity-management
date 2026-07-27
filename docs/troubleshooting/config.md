# When the error is in the project configuration

One optional file at the project root, `entity-manager.yaml`, says where things are and how the project behaves when the contract cannot be met. A project without it works fine on the defaults; the file exists for when the defaults are not what you want.

```yaml
schemas_dir: schemas
entities_dir: entities
policy:
  on_unresolvable: block
```

That file is also what marks the folder as an entity-managed project — the engine finds a project by walking upward looking for it, or for a `schemas/` directory.

## `config must be a YAML mapping` / `config is not valid YAML: …` / `cannot read config as UTF-8 text: …`

The same three failures a record can have, in this file. See [the record's frontmatter and its identity](record-structure.md) for PyYAML's own messages and [before the engine can look](runtime.md) for the encoding one.

The config goes through the pre-parse comment scan as well, and when the scan speaks, the engine stops reading the file rather than proceeding on a value it may have misread.

## `config key '<key>' must be a non-empty string, got … — falling back to default '<default>'`

`schemas_dir` and `entities_dir` name folders, relative to the project root. Anything else in their place — a number, a list, an empty value — is not a folder name.

The engine falls back to the documented default and carries on, because a project that cannot say where its schemas are is better served by looking in the usual place than by refusing to run.

**One silence to know about**, registered against the project itself as ISSUE-016 and waiting for a fix. Unknown keys at the *top level* of this file are not reported: a misspelled `schema_dir:` is ignored, the engine uses the default, and nothing says so. That matters most when the default folder still exists beside the one you meant to declare — after moving your layout, typically — because then the run comes back green having validated the other folder. If a setting you declared appears to have no effect, or a green looks too easy after a reorganization, check the spelling against the three names above — `schemas_dir`, `entities_dir`, `policy` — before looking anywhere else. (Keys under `policy` *are* checked — see below.)

## `config key 'policy' must be a mapping, got …`

`policy` holds settings, so it takes keys beneath it, not a value beside it:

```yaml
policy:
  on_unresolvable: block
```

## `unknown policy key '<key>' (valid: on_unresolvable)`

A misspelled key under `policy` — `on_unresolveable`, `unresolvable` — would leave the project silently running on the default policy while its file appears to declare another. Since the default is the strict one, the failure would be quiet in the dangerous direction: you would believe you had loosened the contract and would not have, or the reverse.

So the key names are checked, not only their values. Fix the spelling.

## `policy.on_unresolvable must be one of block, relax-and-report, got …`

Those are the two modes, and what they mean is on the [index page](troubleshooting.md#when-red-is-the-honest-answer).

A typo here — or a comment that swallowed half the value — must never quietly behave as `block`, which is why this one is an error rather than a fallback. The two modes lead to genuinely different work, and the project must say which it is in.
