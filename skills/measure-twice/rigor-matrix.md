# Rigor matrix & rollback plan

How much process a change earns, by where it sits on the two axes — and the rollback template for the changes that need one.

## The matrix

Locate the change, apply the recipe. The goal is *proportion*: do everything the quadrant calls for, and nothing it doesn't.

### Low blast x Two-way door — "just do it"
The common case. Make the change; rely on the fact that a revert restores everything.
- Tests: existing suite.
- Review: normal.
- Rollout: direct.
- Rollback: `git revert`.

### Low blast x One-way door — "confirm, then do"
Small surface but irreversible. The risk is mis-rating reversibility.
- First: double-check it's truly one-way and truly low-blast.
- Add: a backup/snapshot if any data is touched.
- Rollback: the restore path, written down.

### High blast x Two-way door — "test, review, revert-ready"
Wide reach but recoverable. Lean on reversibility, invest in catching it fast.
- Tests: add coverage for the newly-reachable paths and edge cases.
- Review: a reviewer who knows the affected area.
- Rollout: behind a flag or staged if it touches a hot path.
- Rollback: revert + (if flagged) flip off; ensure observability so you notice quickly.

### High blast x One-way door — "the full treatment"
The dangerous quadrant. Slow down deliberately.
- **Convert to two-way first** if at all possible (flag, deprecation window, backup, dual-write). This is almost always worth it.
- Pre-mortem: mandatory; verify each load-bearing assumption (`assumption-ledger`).
- Tests: new coverage + a staging/canary trial on real-ish data.
- Review: explicit sign-off; an ADR (`leave-a-trail`) for the decision.
- Rollout: staged / canary with abort criteria.
- Rollback: a written plan (below), rehearsed if data is involved.

## Rollback plan template

Required for any one-way-door or high-blast change. Write it *before* cutting.

```markdown
## Rollback plan — <change>

**Door:** one-way | two-way    **Blast radius:** low | high

**Abort criteria:** <the signal that says stop/roll back — error rate, data check,
report mismatch, alert>

**Undo steps:**
1. <e.g. flip feature flag `x` off>
2. <e.g. revert commits N..M>
3. <e.g. restore `users.legacy_score` from snapshot `snap-2026-06-23`>

**Data restore:** <backup location + retention window + how to verify integrity>

**Comms:** <who to notify; where to post status>

**Point of no return:** <when rollback stops being possible — e.g. after backup
expires / after clients adopt the new API — and what guards that boundary>
```

## The one rule

If you cannot write the abort criteria and the undo steps for a one-way, high-blast change, you are not ready to make it. The missing rollback plan is the finding — stop and convert the door to two-way, or get the decision reviewed.
