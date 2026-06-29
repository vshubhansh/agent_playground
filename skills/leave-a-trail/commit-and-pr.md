# Commit messages & PR descriptions

Both answer the same question — *why this change?* — at different scopes. Neither should restate the diff.

## Commit messages

Structure:

```
<imperative subject, <=50 chars, no trailing period>

<body, wrapped ~72 cols: why this change is needed and why done THIS way.
Describe the problem and the rejected alternatives, not the mechanics of
the diff. The reader can see the diff; they cannot see your reasoning.>

Refs: #<issue>
```

- **Subject in the imperative** ("Fix race in checkout", not "Fixed" / "Fixes"): it completes "If applied, this commit will ___".
- **Body explains why now and why this way.** Skip it only for genuinely trivial changes.
- **Link the issue/PR**, don't paste it.

### Good vs bad

Bad (restates the diff):
```
Update upload.py

Changed maxRetries from 5 to 7 and added a sleep in the loop.
```

Good (captures intent):
```
Harden uploads against gateway 504 storms

Gateway returns sporadic 504s for ~90s during failover. 5 retries with
no backoff gave up too early and surfaced as user-facing upload errors.
Raise to 7 with exponential backoff so total wait covers the failover
window. Considered a longer fixed delay but it slowed the common path.

Refs: #812
```

## PR descriptions

A reviewer needs the story, not a file list. Cover:

```markdown
## Why
<the problem / motivation; link the issue>

## What & approach
<the approach at a high level — not a per-file narration>

## Alternatives considered
<what else you weighed and why you rejected it>

## Risk & rollback
<blast radius, what could break, how to revert/flag off>

## Test plan
<how you verified it; what a reviewer should check>
```

- **"Alternatives considered" is the highest-value section** — it's the reasoning that's otherwise lost forever.
- **Risk & rollback** pairs with the `blast-radius` skill; **Test plan** pairs with `assumption-ledger`'s completion gate (state what you verified vs. assumed).

## The one anti-pattern

If a sentence would still be true by reading the diff alone, cut it. Trails carry what the diff *can't*: motivation, rejected paths, and constraints.
