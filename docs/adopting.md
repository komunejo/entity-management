# What adopting this asks of you

This page is for the moment before anything is installed: deciding whether this method suits your material, and knowing what it will ask of you if it does. It is the page that would have saved the two most instructive failures this project has had from the field, both of which came from capable people who understood the tool and misread the offer.

## What this is, in plain terms

You have a folder of Markdown documents. Some of the facts inside them are structured whether you wrote them that way or not: who owns this, what state it is in, what date it happened, which other document it depends on. Those facts drift. Not through carelessness — through ordinary work, across months, between several people and several assistants, none of whom can hold the whole set in mind.

This method makes those facts explicit and checkable, without taking your prose away from you. Each document keeps a small structured block at the top; a program checks that block against rules you wrote; the prose underneath stays entirely yours.

That is the whole idea. What follows is what it costs.

## What a record looks like

```markdown
---
id: DEC-004
entity: decision
title: "Records are named by their ID"
status: accepted
date: 2026-07-12
supersedes: DEC-001
---

We keep the ID in the filename so that any record can be found by ID in any
file browser, without a search. The cost is that filenames are not sentences.

This replaces [the original naming rule (DEC-001)](DEC-001.md), which named
files after their titles and broke every time a title was edited.
```

Everything above the second `---` is the structured part. Everything below is prose — write it however you write.

Note the link in the last paragraph. Because its visible text carries an ID, the engine checks it: `DEC-001` must exist, must be a decision, and must be at that path. If it stops being true, you hear about it. An ordinary link with no ID in its text is left alone.

## What a schema looks like

A schema is the rulebook for one type of record. It is a small file, and you write it:

```yaml
entity: decision
id:
  prefix: DEC
fields:
  title:      {type: string, required: true}
  status:     {type: enum, required: true, values: [proposed, accepted, superseded]}
  date:       {type: date, required: true}
  supersedes: {type: ref, entity: decision}
  ticket:     {type: string, pattern: '[A-Z]{2,}-\d+'}
```

Reading it aloud: a decision has a title and it is text; it has a status and it is one of these three words; it has a date; it may point at another decision that it supersedes, and that decision must exist; and it may carry a ticket, which looks like `OPS-1421`.

That is the artifact adoption asks you to produce. Not code — a description of what your own material is.

### Say as much as you know

That last field is worth a paragraph of its own, because it is where most first schemas leave value on the table.

A field can declare not only its *type* but its **shape**, as a `pattern`. And declaring the shape buys you two things rather than one. The obvious one: a malformed ticket is caught. The one nobody expects: **the engine stops bothering you about that field.**

That second effect is the substance of this version. The engine reads your raw text before parsing it, hunting for the ways YAML quietly changes what you wrote — most of them involving `#`, which YAML treats as the start of a comment. Where it cannot tell your prose from a comment, it has to ask. But where you have *declared* what the values look like, it no longer has to: it compares the line against your own shape and decides. A field whose values are codes, slugs, hashtags or identifiers goes from a flag on every line to silence, because you told it what it was looking at.

So the principle underneath the whole schema, and the one that repays effort during adoption: **the engine can only check what you declare, and it only pesters you about what you left undeclared.** A first schema that says `{type: string}` about everything is a schema that has told the engine almost nothing — it will validate very little and ask about rather a lot. Every constraint you add — a type instead of plain text, an enum instead of a string, a shape instead of an enum, a `ref` instead of a name — is one more thing checked for you and one less thing you will be asked.

Which is also why it is normal not to get this right on the first pass. You learn what your material's shapes actually are by watching real records meet the schema, and then you say more.

## The one thing adopters get wrong

**The structure gets declared. The engine never infers it.**

Every field report this project has received from a struggling adoption reduces to that sentence. The clearest: someone staged 240 existing notes as a project, ran the validator, counted 171 errors, and read the count as a verdict against the method.

It was not. Those errors measured a corpus against a schema nobody had written — and the engine has nothing else to measure against. It reads your declaration and checks your records against it; it does not look at your files and work out what they seem to be. Until the declaration exists, there is nothing for a record to be right or wrong about.

The comparison the project makes with database engines is exact here, and it cuts both ways. A database gives you integrity guarantees no discipline can match; it also requires the tables to be defined before it gives you anything at all. This is the same bargain, in Markdown.

**Where the declaration comes from is a separate question, and a much more open one.** Reading what you already have and proposing a shape from it is perfectly good work — often the fastest route, and exactly the kind of thing to hand to an assistant. Whether it drafts and you approve, drafts and tells you afterward, or works it out with you field by field, is between you and it; this method has no opinion about how much you delegate. It has an opinion about one thing only: that a declaration exists, and that it is yours.

The single case where reading the shape off the data does not work is worth knowing, because it is what the 240-note report turned out to be. When the material is written by a tool whose format you do not control and which changes without notice, a schema copied from what you observe grants confidence without foundation: when a record fails, you cannot tell whether the record is wrong or the format moved. There, the honest move is to convert into a structure you own rather than validate one nobody wrote.

So a first run against unprepared material producing hundreds of errors is not a failure. It is a measurement — often a useful one, since it tells you where your own material disagrees with itself. What it is not is a verdict.

## The decision this opens with

Before designing anything, the honest question: **is this material worth structuring at all?**

There are real answers of no.

- **Nobody will reference it and nobody will validate it.** Structure exists to be checked and pointed at. A collection nothing points into, and whose facts nobody needs to trust, gains ceremony and loses nothing by staying prose.
- **The facts do not repeat.** This pays off where the same *kinds* of things recur — decisions, issues, clients, sessions, characters, sources. A folder of unrelated one-off documents has no type to declare.
- **The material is still finding its shape.** Declaring a contract over something you are still discovering fights you. Wait, or declare loosely on purpose (there is a setting for it: `strict: false` reports undeclared fields as warnings rather than errors).
- **It is somebody else's format, still moving** — the case above. Not never, but not *in place*: convert into a structure you own, and adopt that.

And the strong yes: things you refer to by name, whose state matters, that outlive the conversation that made them, and that more than one person or assistant will touch.

**Partial adoption is normal.** One type, the one that hurts. Not every folder, not on day one.

## Two ways in

**A project born structured.** You declare the types, and records are created against them from the first day — usually by asking an assistant, which prints a stub with the next free ID already filled in. Nothing to migrate. This is the easy road and it is available more often than people assume.

**An existing corpus, converted.** You have the material already. Then adoption is a migration, in this order:

1. **Look at what you have** and name the recurring kinds. Not exhaustively — the two or three that matter.
2. **Design one schema**, for the kind that would benefit most. Write down the fields you actually rely on, not every field you can imagine.
3. **Convert a handful of documents by hand** and validate. Ten is enough to find out whether the schema you wrote is the schema you needed. It almost never is on the first try, and that is what this step is for.
4. **Revise the schema.** Expect this. Schema evolution is a first-class workflow here, not a repair — the project records it as a requirement of its own design.
5. **Then convert the rest**, which is the part an assistant can do in bulk once the shape is settled.

The order matters because reversing it is what produces the 171-error morning: converting everything against a schema nobody has tested yet.

## Things that surprise people, in a good way

**The layout is yours.** Records of a type live together in a folder you name, and the rest of the project is untouched. The engine only looks where you tell it to look.

**Free files are welcome.** Notes, drafts, indexes, a README in the middle of everything — files that are not records are not errors. They only need to live outside the folders that records live in. This is a designed feature, not a gap: a registry that demanded every file be typed would be unusable in a real vault.

**Filenames can stay yours too.** By default a record's filename starts with its ID, which makes any record findable by ID in a file browser. If your project would rather name files in its own words, one declaration (`filename: free`) turns that off entirely, and the ID lives only in the frontmatter.

**Your language is your own.** Titles, labels, field values, prose, and filenames take accents, `ñ`, `¿`, `¡`, and anything else you write in. The single exception is the ID prefix, which is plain ASCII by design because IDs travel through URLs, filenames and other people's tools — and that restriction fails loudly rather than mangling anything quietly.

**Prose is never validated.** No style rules, no required sections, no vocabulary. The engine reads the block at the top and the links that carry IDs. Everything else, it does not judge.

## What it will not do for you

It will not tell you whether your schema is a *good* description of your material. It checks that a schema is well-formed and that your records honor it; whether the fields you chose are the right fields is a judgment nobody can automate, and revising it later is expected rather than exceptional.

It will not invent a value to make a red go green — and neither should you. When a required fact is genuinely unknown, the red is the honest deliverable: a list of what cannot be settled without a decision. The project's contract policy governs what happens then, and [when the engine says no](troubleshooting/troubleshooting.md#when-red-is-the-honest-answer) explains the two modes.

## Starting

The install is in the [README](../README.md), and it is short. After that, the honest first step is a conversation rather than a command: describe your material to your assistant, ask what types it sees, and push back on the answer. The schema is yours; the assistant drafts it, you decide it.

To see what that conversation produces — the actual files, in order, from an empty folder to a record the engine calls good — follow [the walkthrough](quick-guide.md). When you want a second type, a relation between types, or a rule that spans several records, [designing schemas](schemas.md) is the whole vocabulary in one page.

And when something goes red — it will, on the first day, by design — [when the engine says no](troubleshooting/troubleshooting.md) explains every message the engine can raise, with the fix.
