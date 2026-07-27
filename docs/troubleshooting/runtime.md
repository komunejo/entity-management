# Before the engine can look

These are not integrity verdicts. They all mean the same thing: the engine never got as far as reading your records, so it has no opinion yet about whether they are sound. Every one of them ends calmly.

They are also the ones that report **exit code 2**. If you are scripting anything around the validator, that distinction is the important one: `0` means integrity holds, `1` means violations were found and listed, `2` means the run itself could not happen. Never read a `2` as “no violations” — it means the engine could not even look.

## `this tool needs Python 3.8 or newer; the Python running it is …`

The engine is a Python program, so a Python has to run it, and the one that answered is too old.

Where machines stand, by default: Linux ships one. macOS gets one with the Command Line Tools, which installing `git` already brings. Windows ships none — and its `python` command is a decoy that opens the Microsoft Store rather than telling you it is absent.

If you work with an assistant, it sorts all of this out and, at most, asks you once to install Python, because that step is genuinely yours. The official installer at [python.org](https://www.python.org/downloads/) is the short road: choose “install just for me”, which needs no administrator. If you manage your own machine, install it however you prefer — the engine does not care which Python it gets, only that it is 3.8 or newer.

On Claude Desktop this message should never reach you at all: Claude's work runs in an environment of its own, and your machine needs nothing installed.

## `this tool needs the PyYAML library, and the Python running it cannot find it`

The engine reads YAML — the small structured language at the top of every record — through a library called PyYAML. It is the engine's only dependency, and the Python that just ran is not seeing it.

An assistant treats this as its own housekeeping and fixes it without bothering you: in its own per-user environment, only if actually missing, never with elevated permissions. On your own, install it any way you like — your distribution's package, a virtual environment, `pip` where your platform allows it. The official page is [pypi.org/project/PyYAML](https://pypi.org/project/PyYAML/).

One thing worth knowing before you go looking: on many current systems, the system Python refuses direct installs by design. That is a protection, not a fault, and a virtual environment is the standard answer; your operating system's own documentation is the authority on its particulars.

**The variant that confuses people.** Validation works when you run it by hand, and then a hook fails with this exact message. Nothing is broken: the hook is calling a *different* Python than your shell is. Point the hook's command at the same interpreter, full path and all.

Neither of these two stories ever requires touching the system Python, elevated permissions, or anything machine-wide.

## `could not find project root (no entity-manager.yaml or schemas/ upwards from cwd). Use --root.`

The engine locates a project by walking upward from wherever it was run, looking for either an `entity-manager.yaml` file or a `schemas/` directory. It reached the top of the tree without finding either.

Almost always this means the command ran from somewhere outside the project — a home directory, a parent folder, a different repository. Run it from inside the project, or point it explicitly:

```bash
python3 entity_lint.py validate --root /path/to/the/project
```

If you *are* inside what you believe is the project and this still fires, then the project has neither of the two landmarks, and what you actually have is a folder of Markdown files that nobody has declared a registry yet. That is a beginning, not an error — see [what adopting asks of you](../adopting.md).

## `these path filters matched no loaded record: …`

You asked the validator to check specific files, and one of the paths you named is not a record the engine loaded. Rather than validate the rest and report a cheerful green, it stops: a typo in a filter would otherwise produce a pass that means nothing.

The path may be misspelled, or it may name a file that is not a record at all — a file outside every declared location, or one the engine skipped. Check the spelling first; if the spelling is right, the file's own placement is the question, and [where files live](placement.md) is the page for it.

## `no schema for entity 'x' (known: …)`

You asked for a stub of a type this project does not declare — `new proposals` where the type is `proposal`, or a type you have not written a schema for yet. The message lists the types that do exist.

If the type is genuinely missing, writing its schema is the next step: [when the error is in your schema](schema.md) shows a complete one.

## `cannot read file as UTF-8 text: …` / `cannot read config as UTF-8 text: …`

The file exists but its bytes are not valid UTF-8, so the engine will not guess at them.

In practice this is an encoding left over from another tool — a file saved as Latin-1 or Windows-1252, usually recognizable because it contains accented characters or `ñ`. Re-save it as UTF-8 in your editor and the content comes back intact. (A UTF-8 file that begins with a byte-order mark, as some Windows editors write, is read without complaint — that case is handled.)

The engine reports this and keeps going rather than crashing: a violation report you can act on is always better than a stack trace.
