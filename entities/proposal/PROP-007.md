---
id: PROP-007
entity: proposal
title: "raw-text scan for the Markdown side: recorded, and declined"
status: rejected
date: 2026-07-26
tags: [markdown, integrity]
---

## Motivation

Split from the last open question of [raw-text scan: pre-parse checks for what the parse destroys](PROP-006.md)^[PROP-006](PROP-006.md): the Markdown body of a record has its own version of the silent-misread problem — a ref malformed enough that the resolver does not recognize it does not error, it simply reads as plain prose. Should the scan's jurisdiction extend there? The question deserves an address, so it is registered; it is also rejected on arrival, and the status says so.

## Why declined

The judgment is that the value is low, for two asymmetries with the YAML case. First, the failure is not silent to a human: Markdown is what the human editing tools render, and a ref that fell back to prose shows itself — unlinked, unstyled — in any preview, which is exactly the mechanism frontmatter lacks (nobody renders YAML to the eye before the parse eats it). Second, writing well-formed Markdown is squarely within what an agent is expected to produce without mechanical assistance; the YAML scan earns its place because there the *correct-looking* text is what misparses, not the sloppy one.

Rejected is not sealed. The membership discipline of [PROP-006](PROP-006.md)^[PROP-006](PROP-006.md) applies here too: this judgment stands until a silent Markdown misread is *measured* doing real damage in use — a record whose meaning changed and nobody saw it. That evidence would reopen the question with a concrete member to seed from, which is the only way rules enter the scan anyway.
