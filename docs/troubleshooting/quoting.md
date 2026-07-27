# When the file looks fine and YAML disagrees

This is the least intuitive family in the catalog, and it deserves the first explanation rather than a message list. Everything here has the same shape: **the file looks perfectly correct to a human eye**, and YAML reads it as something other than what you wrote — cleanly, without any error, into data that is not what you meant.

That is why these checks run *before* the file is parsed, on the raw text you typed. Once the parse has happened the evidence is gone: nothing downstream can tell that your sentence lost its second half, because by then the second half was never there. (The project's own registry keeps the measured cases as ISSUE-003 and the design as PROP-006 and DEC-023.)

One writer-side habit avoids the whole family: **when a value holds prose, quote it.**

## The two YAML rules underneath all of this

You do not need to learn YAML to use this. You need two facts, and they explain every message on this page.

**One.** A `#` that has a space before it starts a comment, and everything after it on that line is discarded. Not just at the start of a line — anywhere.

```yaml
title: count # per item        # YAML stores: "count". The rest is gone.
```

**Two.** Inside `{…}` and `[…]` — YAML calls this *flow* style — commas and colons are punctuation of the data structure, not of your sentence. A comma in your prose splits it into two entries.

```yaml
# Written, meaning one description:
meta: {description: applies always, required: true}

# Read by YAML — two keys, the description truncated, 'required' set:
meta:
  description: applies always
  required: true
```

Quotation marks are what tell YAML “this is text, all of it”. That is the whole remedy, and it is why so many of these messages end in *quote it*.

## When the engine speaks, and when it stays quiet

The engine does not flag every comment it sees. Comments in YAML are legitimate and useful, and a validator that shouted at all of them would be wrong. It speaks only where it genuinely cannot decide, and it works that out from your own schema:

- **A field the schema types as `integer`, `number`, `boolean`, `date`, `datetime`, `enum` or `ref`**: silence. A number can never contain ` # per item`, so the tail is provably a comment. And if a comment ever did swallow real content, the value left behind fails the type check, which reports it with a better message anyway. `count: 3  # per item` is simply fine. (This is what the field report ISSUE-013 taught the engine — before it, correct configurations were going red.)
- **A `string` field whose schema declares a `pattern`**: decided line by line against that shape. If the value matches the declared shape and the whole line does not, the tail is a comment — silence. If the *whole line* matches, then your value legitimately contains a ` #` and YAML is about to cut it in half — so the engine speaks.
- **A `string` field with no declared shape**: nobody can tell your prose from your comment, including you six months from now. The engine says so and names the two ways out.

The practical consequence: if a whole class of your values has a shape — codes, slugs, hashtags, identifiers — the durable fix is not a quotation mark on every line. It is declaring that shape as a `pattern` in the schema, once. The engine then knows what you know.

Every message in this family names the field it fired on, in square brackets: `[count]`.

## `unquoted value contains ' #' — from the '#' on, YAML reads a comment and the tail silently vanishes from the value; quote the value to keep it (or to make the comment unambiguous)`

Rule one, in a field where the engine cannot decide. Two honest ways out, depending on what you meant:

```yaml
title: "count # per item"      # the # was part of the title
title: "count"  # per item     # the # really was a comment
```

Either one settles it, for the engine and for the next reader.

## `value looks like '#'-glued content, but YAML reads a comment: the value is null and the text is gone; quote "#…" if it was meant as the value (a comment puts a space after '#')`

Text glued straight onto the `#` — a hashtag, `#TODO`, `#1` — looks like content to a person, but YAML starts the comment at the `#` regardless. The field ends up with **no value at all**, and every character after the colon is discarded:

```yaml
tag: #urgent                   # YAML stores: null. "#urgent" is gone.
tag: "#urgent"                 # YAML stores: "#urgent"
```

This one fires on every kind of field, including the typed ones where the engine is otherwise silent, and for a precise reason: the field parses to *null*, so no downstream check ever sees what vanished. There is nothing left to catch it later.

On a typed field the message is the same story with the remedy that actually exists there:

> `value looks like '#'-glued text, but YAML reads a comment: the field is silently empty and the text is gone; write the value you meant — a '#…' text can never satisfy this field's declared type (a comment puts a space after '#')`

Quoting would not help you there — `"#3"` is still not an integer. What is needed is the value you meant.

**What does not fire:** a comment with a space after the `#`. `tag: # to be filled` is the ordinary way to annotate a deliberately empty field — it is the form the engine's own `new` command emits — and it stays silent, as does a `## …` run.

## `multi-word unquoted scalar '…' inside a flow collection — a ',' ':' or '#' in it is read as structure, not text; quote it`

Rule two. An unquoted multi-word value inside `{…}` or `[…]` is standing in a minefield: the moment it grows a comma or a colon, YAML splits it and tells nobody.

The engine cannot know whether the comma was your sentence or your structure, so it asks for the two characters that settle it:

```yaml
meta: {description: "applies always, required: true"}   # one value, protected
```

If you genuinely meant two entries, quoting the words in each also clears the flag. Either way the text now says what you mean where YAML can see it.

## `' #' inside a flow collection starts a comment — the rest of the line silently vanishes; quote the scalar`

Rule one caught inside `{…}`/`[…]`, where the vanishing tail usually takes the closing bracket with it — so the damage is not one value but the whole structure. Same remedy: quote the scalar the `#` belongs to.

## `duplicate key 'x' in mapping (line N) — the first value would be silently discarded`

YAML's default behavior with a repeated key is to keep the last one and throw the earlier away without a word.

This is the one message in the family that is never a question of intent: there is no reading of a record that names the same key twice in which both were meant. Delete the wrong line, keep the right one. The line number points at the *second* occurrence, and the usual cause is an editing slip — a field added at the bottom of a frontmatter that already had it at the top.

## What never fires, so you can stop worrying about it

The rules above are narrower than they sound. All of these are read as content and pass in silence:

| What you wrote | Why it is fine |
|---|---|
| `url: http://example.org/page#section` | no space before the `#`, so no comment |
| `title: "a # inside quotes"` | quoted text is text, always |
| `notes: \|` then indented lines with `#` | inside a block of literal text, `#` is just a character |
| `# a whole comment line` | there is no value for it to truncate |
| `tags: [a, b] # a note` | the comment starts after the collection closed |
| `count: 3 # per item` | typed field: the tail cannot be content |

## The frontier no declaration crosses

One case cannot be rescued by any schema, and it is worth knowing where the line is: a value glued to the `#` with no quotation marks (`tag: #label`) is discarded by YAML during the parse, before the engine ever sees the file. Recovering it would mean the engine reading YAML differently from every other YAML reader in the world, which would make your files lie to everything else that opens them.

So the answer there is quotation marks, and the message says so precisely.
