# Ledger template

A lightweight, living table — a few honest lines, updated as you verify. Keep it visible while working; run the completion gate before any "done".

```markdown
## Assumption ledger — <task>

| # | Claim the work depends on | Status | Load-bearing? | How verified / how to verify |
|---|---------------------------|--------|---------------|------------------------------|
| 1 | `getUser` returns null on cache miss | Verified | yes | read src/cache.ts:42 — returns null |
| 2 | input list is already sorted | Assumed | yes | TODO: check the caller / add a test |
| 3 | this runs only in Node, not the browser | Assumed | no | incidental; skipping |
| 4 | the second capturePayment call is a duplicate | Refuted | yes | rg shows it's inside a retry loop |
```

## Completion gate

Before claiming done / fixed / works, run this:

```
- [ ] Every load-bearing claim is Verified or Refuted (none left Assumed)
- [ ] Each Verified claim notes HOW it was verified (not "I expect")
- [ ] Refuted claims have been designed around, not ignored
- [ ] Any load-bearing claim that truly can't be verified here is stated to the human as an explicit risk
```

If a box can't be checked, the work is not done — it's done *pending a stated, unverified dependency*. Say so plainly.

## Rules for keeping it
- **Honesty over optics:** a claim is `Assumed` until you actually checked it; do not upgrade to `Verified` on the strength of confidence.
- **Focus the effort:** verify load-bearing entries; let incidental ones stay `Assumed` on purpose.
- **Keep Refuted entries** so the same false belief isn't quietly re-adopted later.
- **Stay light:** the ledger earns its place only if it's cheap enough to actually maintain.
