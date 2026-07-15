---
id: ISSUE-004
entity: issue
title: a prose reference can be human-readable or validated, never both
status: resolved
date: 2026-07-15
channel: report
resolved_by: [DEC-017, DEC-018]
tags: [architecture, references]
---

A record store keyed by human-readable slugs was deliberately left outside the engine: adopting it would have meant renaming every record and rewriting every ref. The same project's decision log went in and is in daily use, so this is not the engine failing — it is the engine not being applicable, which is the more expensive kind of no. A defect gets fixed; a boundary just quietly keeps things out.

Where it actually bites is the prose, because prose is where a person writes. Measured on the real engine, with two links that both point at nothing:

```
[[DEC-999]]                     -> error: does not resolve to any entity
[[how we remember provenance]]  -> accepted in silence
```

The silence is not a bug: in a vault most wikilinks target notes that are not entities, and erroring on them would be absurd. But the consequence is exact — the only links that get integrity are the ones nobody wants to write. The price of a checked reference is a number in the middle of a sentence.

So the trade in [entities are referenced by stable IDs with a unique prefix per type](../decision/DEC-004.md)^[DEC-004](../decision/DEC-004.md) — references use IDs and never names, because renames are the great destroyer of coherence — is real, argued and right, and it is paid by the writer, sentence by sentence. That is a defensible price. It has never been named as one.

The filename rule is a separate matter and a weaker one. `id` is declared in the frontmatter, and then the engine additionally requires the file to be called `<ID>-slug.md` — the name must repeat an identity it does not carry. When this was written nothing in the registry said why: no decision mentioned filenames, no requirement did, and DEC-004 is about refs and silent on names. The rule lived in one line of `references/schema-language.md` and one line of the engine, explained in neither. [a filename is an ID and an optional handle; the name is the title](../decision/DEC-018.md)^[DEC-018](../decision/DEC-018.md) is the answer that arrived: the pattern was never the engine's to fix, and a store can keep `provenance.md` while carrying `id: MEM-042` inside, losing nothing DEC-004 protects.

Two questions, in this order. Is the filename rule load-bearing or incidental — because if incidental, part of what kept this store out was never necessary, and if load-bearing the reason has earned the right to be written down. And then the hard one: can a reference be human-readable in the prose and stable underneath it at the same time, or is the number in the sentence the actual, irreducible cost of the guarantee? The first is answerable. The second is what the report is really asking, and answering it either way lands as a decision — including a no, which would at last put a price tag on something this engine has been charging silently.
