# ADR template

An Architecture Decision Record captures a significant, hard-to-reverse decision that outlives any single PR. Keep them in-repo (e.g. `docs/adr/NNNN-short-title.md`), numbered and immutable — supersede rather than rewrite.

**When to write one:** the decision sets a pattern others will follow, is expensive to undo, or future-you will ask "why on earth did we do it this way?" If a commit message covers it, you don't need an ADR.

```markdown
# ADR-NNNN: <short decision title>

**Status:** Proposed | Accepted | Superseded by ADR-MMMM
**Date:** YYYY-MM-DD

## Context
The forces at play: the problem, the constraints (perf, deadline, compat,
team, cost), and what makes this decision necessary now. Facts, not opinions.

## Decision
What we are doing, stated plainly and actively: "We will ___."

## Alternatives considered
- **<Option A>** — why rejected (the trade-off that lost).
- **<Option B>** — why rejected.
(This section is the point of the ADR. The decision is obvious in hindsight;
the alternatives and their trade-offs are what's actually worth recording.)

## Consequences
- **Good:** what this buys us.
- **Bad / accepted costs:** what we give up or take on (the honest part).
- **Follow-ups:** anything this now requires or enables.
```

## Rules
- **Immutable record.** Don't edit an Accepted ADR to change the decision; write a new one that supersedes it and update the old one's status. The history of decisions is itself valuable.
- **Alternatives are mandatory.** An ADR with no rejected options isn't recording a decision, just an announcement.
- **Be honest about the costs.** The "Bad" bullet is what makes an ADR trustworthy and saves a future reader from re-litigating a known trade-off.
- **One decision per ADR.** Keep them small and linkable.
