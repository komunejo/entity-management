# Rules that cross several records

Most checks look at one record at a time. A few look at all of them together — “no two students share a seat”, “at most three proposals per quarter”. A schema declares those as *constraints*, and they are the only rules whose violation is not visible from inside any single file.

That is what makes their messages different in kind: the fix is never a formatting change. Something in the world the records describe has to move, merge, or retire — or the rule itself has gone stale.

## `constraint 'unique' on [f1, f2] violated: same values as X`

Two records carry identical values in a combination the schema declares must be unique, and the message names the other record.

The declaration might be on one field (`code` must be unique) or several together (`room` plus `slot`: the same room at the same time). Records where any of those fields is empty are skipped — an absent value is not a collision.

To resolve it, decide which record is the truth. Then either correct the duplicate's values, retire it, or merge the two. If they are both legitimately real and the collision is genuine, then the uniqueness rule is describing a world that no longer exists, and the schema is what changes — with a note in `schemas/CHANGELOG.md`.

## `constraint 'max_count_per g' violated: 'v' has N records (max M): ID, ID, …`

A cap declared in the schema is exceeded for one group, and the message lists exactly which records make up the excess so you are not left counting.

A cap can group by a field of the record, or by a field reached *through* a reference — “at most 3 per student's department” walks from the record to the student to the department. Records whose group value is empty are not counted against any group.

The fix is domain sense: something moves out of the group, something is retired, or the cap was set for a season that has passed and the schema should say so.

## When either message carries a trailing explanation

A constraint may declare a `description`, and when it does the engine appends it to the message after an em dash:

```
constraint 'max_count_per supervisor' violated: 'S-004' has 4 records (max 3): PRJ-002, PRJ-007, PRJ-011, PRJ-015 — no supervisor takes more than three concurrent projects
```

That tail is the schema author explaining, to whoever hits the wall, why the wall is there. It is worth writing one for every constraint that is not self-evident: a cap without a reason invites someone to raise it.
