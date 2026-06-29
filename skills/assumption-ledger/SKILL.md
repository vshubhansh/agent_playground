---
name: assumption-ledger
description: >-
  A discipline for not being confidently wrong. Surfaces the hidden assumptions
  a task rests on, tags each Verified / Assumed / Refuted, flags the
  load-bearing ones, and verifies them cheapest-first. Use when starting a
  non-trivial task, debugging, reasoning about how code or a system behaves, or
  before declaring something done, fixed, or working — and whenever reasoning
  leans on "should", "probably", "I assume", "presumably", or "it must".
---

# assumption-ledger

The failure that ruins more of my work than any other is being **confidently wrong**: I build on invisible assumptions — "this API returns null on miss", "the config defaults to X", "it worked before so it still does" — and only find out they were false *after* I've declared the work done. This skill makes my epistemic state explicit and auditable.

**The discipline:** keep a ledger of the claims the work depends on. Tag each `Verified` / `Assumed` / `Refuted`. Mark which are **load-bearing** (if false, the solution breaks). Verify load-bearing assumptions cheapest-first. **Never claim "done / fixed / works" while a load-bearing assumption is still `Assumed`.**

No tools required — verification uses whatever's already at hand: reading the actual source, running one command, or writing a tiny test.

## When this fires

Starting a non-trivial task; debugging; explaining how code or a system behaves; choosing an approach; and — most importantly — **right before claiming completion**. Also any time the reasoning turns to *should / probably / I assume / presumably / obviously / I think / it must / by default*. Those words are the sound of an untested bet. See [assumption-tells.md](assumption-tells.md) for the full list of tells and how to check each.

## Keep the ledger

Copy this checklist for any non-trivial task.

```
Assumption ledger:
- [ ] 1. Surface  — list the claims the approach rests on
- [ ] 2. Classify — tag each Verified / Assumed / Refuted
- [ ] 3. Mark load-bearing — which ones break the solution if false?
- [ ] 4. Verify cheapest-first — kill load-bearing Assumed entries
- [ ] 5. Gate completion — no "done" while a load-bearing entry is Assumed
```

### 1. Surface
Write down the claims your plan silently depends on — about inputs, APIs, the environment, what the code currently does, and what "correct" means here. Use the tells reference to catch the ones hiding in confident-sounding language. If you can't name your assumptions, you can't check them.

### 2. Classify
Tag each entry:
- **Verified** — you actually checked it (and note *how*: read the source, ran it, saw the test pass, read authoritative docs).
- **Assumed** — believed but unchecked. The default state of most beliefs; be honest about it.
- **Refuted** — checked and found false. Keep it in the ledger so you don't re-assume it.

Distinguish "I checked" from "I expect". Memory of an API, a default, or "how this usually works" is `Assumed`, not `Verified`.

### 3. Mark load-bearing
For each `Assumed` entry ask: *if this is false, does the solution break?* If yes, it's **load-bearing**. Incidental assumptions can stay unverified; load-bearing ones cannot. This is where you focus — most assumptions don't matter, a few decide everything.

### 4. Verify cheapest-first
Spend the verification budget on load-bearing `Assumed` entries, cheapest check first. Prefer evidence over more reasoning:
- Read the actual implementation rather than guessing its contract.
- Run one command / a REPL line / a tiny test to observe real behavior.
- Check the real config/env/version rather than the assumed default.

An assumption you can verify in under a minute should never be left `Assumed`. When verified, optionally note evidence strength with skill #1's labels (`Confirmed` / `Likely` / `Speculative`).

### 5. Gate completion
Before saying done / fixed / works, scan the ledger. If any **load-bearing** entry is still `Assumed`:
- verify it now, or
- if it genuinely can't be checked here, **promote it to a stated risk** — tell the human explicitly "this depends on X, which I could not verify" — rather than burying it under a confident claim.

"I can't easily verify this" is a disclosure, never a silent bet.

## Rules

- The burden is on the claim: an unverified load-bearing assumption is a liability, not a detail.
- Absence of doubt is not evidence. Confidence is a feeling; the ledger is the check.
- Keep it lightweight — a few honest lines beat an exhaustive list nobody maintains.

## Worked example

> Task: "Fix the double-charge bug by removing the second `capturePayment` call."

1. Surface: my fix assumes (a) there are exactly two calls, (b) the second is redundant, (c) `capturePayment` is idempotent-safe to drop.
2. Classify: all three `Assumed`.
3. Load-bearing: (a) and (b) — if the "second" call is actually a *retry* guarded by a condition, deleting it breaks legitimate retries.
4. Verify cheapest-first: `rg "capturePayment"` → the call is inside a `if (attempt < maxRetries)` retry loop, not a duplicate. Assumption (b) → **Refuted**.
5. Gate: do not ship. Rework — the real bug is a missing idempotency key, not a duplicate call. No "fixed" was ever emitted on the false premise.

## Reference files
- [assumption-tells.md](assumption-tells.md) — the tells that reveal a hidden assumption, each with its cheapest verification.
- [ledger-template.md](ledger-template.md) — the ledger table format and the completion gate.
