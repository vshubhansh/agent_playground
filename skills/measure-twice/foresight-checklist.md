# Foresight checklist

Two passes for a consequential change: map what it reaches, then imagine how it fails.

## Blast-radius mapping

Find everything the change can affect. Static grep for the symbol is the *start*, not the answer — most under-counting comes from references it can't see.

```
- [ ] Direct callers (grep the symbol/endpoint/table)
- [ ] Dynamic references (string keys, reflection, DI, serialization, config-/env-/route-driven) — see chestertons-fence/suspicious-constructs.md
- [ ] Contracts & external consumers (public API, published package, wire/JSON/proto schema, other repos/teams)
- [ ] Data (rows touched, migrations, backfills, irreversibility, PII)
- [ ] Config & feature flags that select this path
- [ ] Tests covering it (and gaps where none exist)
- [ ] Observability (will you SEE it break? logs/metrics/alerts on this path)
```

Output: a short list of *who/what depends on this*. If the list is long or includes anything outside the repo, treat blast radius as high.

## Pre-mortem failure-mode catalog

Prompt: *"It's three months later and this change caused an incident. What happened?"* Walk the catalog; for each plausible one, write the assumption it depends on and verify it (hand to the `assumption-ledger` skill).

| Failure mode | The question that exposes it |
|--------------|-----------------------------|
| **Data loss / corruption** | Is anything deleted or transformed irreversibly? Is there a backup/restore path? |
| **Broken consumers** | Who calls this that I don't control? Old clients, other services, public API users? |
| **Performance cliff** | What's the cost at 100x the data / load I tested? N+1, full scans, unbounded memory? |
| **Concurrency / races** | Can this run twice at once, out of order, or be retried? Is it idempotent? |
| **Security / authz regression** | Does this widen access, log secrets, or skip a check that was load-bearing? |
| **Rollout & old clients** | During deploy, do new and old versions coexist? Forward/backward compatible? |
| **Observability blind spot** | If this fails silently, how long until anyone notices? Is there an alert? |
| **Cost blowup** | Does this multiply API calls, storage, or compute in a way that bills by usage? |
| **Migration ordering** | Does the schema change need to land before/after the code? What if they interleave? |

A failure mode you can't rule out is not "unlikely" — it's an unverified risk. Verify it, guard against it, or state it explicitly.

## The cheap-conversion move

For most scary findings, the fix isn't "be more careful" — it's to make the change *recoverable*: add a feature flag, a deprecation window, a backup, a dual-write/dual-read, or a canary. Convert one-way doors to two-way doors and most pre-mortem failures become survivable.
