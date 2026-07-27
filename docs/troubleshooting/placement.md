# Where files live

The engine finds records by *location*: each type declares a folder, and every Markdown file inside that folder is taken to be a record of that type. So a file's placement is not decoration — it is what tells the engine what the file is.

Two consequences follow, and both produce messages.

## `file is not inside any declared entity directory (known: …)`

A `.md` file is sitting under the entities tree in a spot no type claims. The message lists the folders that *are* claimed.

Three readings, and the right fix depends on which one it is:

- **It is a record in the wrong place.** Move it into its type's folder.
- **It is not a record at all** — a note, a draft, a README. Move it out of the entities tree. Free files are welcome in the project; they just do not live where records live.
- **It is a record of a type this project has not declared yet.** Then the schema is what is missing, not the file. Declaring a new type is ordinary work — see [what adopting asks of you](../adopting.md).

Files elsewhere in the project are never inspected. The engine only claims the entities tree; the rest of your repository is yours.

## `schemas directory not found — is this an entity-managed project?` / `entities directory not found`

The engine found a project root but not the layout it expected underneath.

Usually the run happened from the wrong place, or against the wrong root — see [before the engine can look](runtime.md). If the project genuinely keeps these folders elsewhere, `entity-manager.yaml` is where that is declared, through `schemas_dir` and `entities_dir`; see [when the error is in the project configuration](config.md).

## A file the engine skips without saying so

One placement rule works by silence, and it is the sharpest edge in the project. **A file named exactly like its own folder** — `proposal/proposal.md`, `library/library.md` — is treated as an Obsidian *folder note*: the descriptive note about the folder, usually a generated index, never a record. The engine skips it, and says nothing.

The consequence to watch for: if you name a real record that way, it vanishes from validation. Not reported, not counted — the run comes back green with a total that is quietly one short. A green that means less than it says is the worst failure this project can have, and this one is registered against itself as ISSUE-007, waiting for a fix.

Until then: **do not name a record after its folder.** If you are wondering whether the engine can see a particular file, the count is the check — `validate` prints how many records it loaded, and that number should be the number you expect.

## Two rules the schemas themselves must respect

These fire when the schema is written, not when a record is, so they live with the other schema messages — but they are placement rules, and knowing them explains a great deal:

- **Two types may not claim the same folder** (`records location … already used by entity …`). A file cannot have two owners.
- **Two types may not claim nested folders** (`records location … is nested with entity …`). If one folder sits inside another, the files underneath would belong to both.

The practical shape this produces: one folder per type, side by side. See [when the error is in your schema](schema.md).
