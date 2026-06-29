---
name: chestertons-fence
description: >-
  A guardrail against removing code whose purpose you don't yet understand.
  Use when about to delete, remove, simplify, refactor, "clean up", inline,
  disable, or replace code that looks unused, redundant, pointless, legacy, or
  dead — especially defensive null/bounds checks, retries, timeouts, sleeps,
  try/catch, feature flags, workarounds, or "unused" exports. Recovers why the
  code exists before letting it be removed.
---

# chestertons-fence

> "Don't ever take a fence down until you know the reason why it was put up." — G.K. Chesterton

The most expensive bugs I cause are deletions. Code that *looks* pointless — a stray null-check, a retry, a magic `sleep`, an "unused" export, a defensive `else` — is often quietly load-bearing. This skill is the guardrail.

**The one rule:** the burden of proof is on *removal*, not retention. "I can't find a use" is **not** proof of safety. You must recover *why* the code exists before it earns the right to be removed.

Depends on git + codebase search only. No network, no proprietary tooling.

## When this fires

Any time the plan involves making code *go away* or look simpler: delete, remove, drop, inline, collapse, "clean up", "simplify", "modernize", de-duplicate, disable, comment out, or replace a construct that appears unused / redundant / legacy / dead. The more pointless it looks, the more this skill applies — apparent pointlessness is the signal that someone solved a problem you can't currently see.

## Inspect the Fence

Copy this checklist and work it before removing anything non-trivial.

```
Fence inspection:
- [ ] 1. Spot the fence (what exactly am I about to remove?)
- [ ] 2. State the naive hypothesis (why it LOOKS safe — that's an assumption)
- [ ] 3. Recover the purpose (git history + usage search)
- [ ] 4. Render a verdict (Load-bearing / Vestigial / Unknown)
- [ ] 5. Preserve the knowledge (if removing, record why it was safe)
```

### 1. Spot the fence
Name the construct precisely and classify it. Many innocent-looking constructs are common fences — see [suspicious-constructs.md](suspicious-constructs.md) for the catalog and per-type verification tips.

### 2. State the naive hypothesis
Write it down explicitly: *"This looks safe to remove because ____."* This sentence is your **assumption**, not a fact — naming it stops you from smuggling it in as proven. The rest of the workflow exists to test it.

### 3. Recover the purpose
Two independent lines of evidence — do both:

- **Why was it added?** Use the `git-archaeologist` skill's techniques (in [../git-archaeologist/SKILL.md](../git-archaeologist/SKILL.md)): `git blame -w -C -C -C` to see past reformats/moves, the pickaxe (`git log -S'<code>' --all`) to find the introducing commit, and `git show <sha>` to read the message + the PR/issue it belonged to. The original commit often *states* the problem it solved.
- **Is it used now?** Search the codebase for references — and remember that grep for the literal symbol is not enough. Check **dynamic** references too: string keys, reflection, dependency injection, serialization field names, env-/config-driven dispatch, route tables, and public API consumed outside this repo. See per-type guidance in [suspicious-constructs.md](suspicious-constructs.md).

Then ask: would removing it change observable behavior in any path, including error paths, edge cases, and other environments (locale, timezone, browser, prod-only config)?

### 4. Render a verdict
Pick exactly one and back it with evidence (cite SHAs / file:line, like skill #1):

- **Load-bearing** — keep it. State what it protects ("guards against null `user` during SSR, added in `a1b2c3d` for #412").
- **Vestigial** — safe to remove. You must cite positive evidence that the original reason is *gone*: the dependency it guarded was deleted, the upstream bug it worked around is fixed, the feature flag fully rolled out, the caller no longer exists. Absence of a reason is not the same as a reason being absent.
- **Unknown** — purpose could not be recovered. Do **not** silently delete. Surface the risk to the human, or keep it with a note. An unrecoverable fence is a reason to stop, not to bulldoze.

### 5. Preserve the knowledge
If you remove it, leave a trail so the next archaeologist isn't confused: record what was removed, its recovered purpose, and the evidence it was obsolete — in the commit message and/or per [removal-record.md](removal-record.md). Removing a fence *and* its rationale forces someone to rediscover it the hard way.

## Rules of evidence

- **Burden of proof is on removal.** Default to keeping when unsure.
- **Absence of evidence ≠ evidence of absence.** Especially for dynamic references and external API consumers.
- **Distinguish two very different statements:** "I proved the original reason is gone" (→ Vestigial, safe) vs. "I don't see a reason" (→ Unknown, stop).
- **Confidence labels** (shared with skill #1): `Confirmed` / `Likely` / `Speculative`. A removal should be `Confirmed`-safe or it doesn't ship.

## Worked example

> Task: "Delete `exportLegacyReport` — nothing calls it."

1. Fence: an exported function with zero direct call sites.
2. Naive hypothesis: "Unused export, grep finds no callers, safe to delete."
3. Recover:
   - `git log -S'exportLegacyReport' --all` → added in `c4d5e6f`, message *"register report formats via plugin map (#640)"*.
   - Usage search: no direct calls, **but** `rg "'legacy'"` finds `formatters['legacy'] = exportLegacyReport` — it's dispatched by **string key** from config.
4. Verdict: **Load-bearing** (`Confirmed`). It's reachable whenever `report.format=legacy` is set; the "no callers" signal was a false negative from static grep.
5. Outcome: keep it. (Had config shown `legacy` was retired everywhere, the verdict would flip to Vestigial — but only with that positive evidence.)

## Reference files
- [suspicious-constructs.md](suspicious-constructs.md) — catalog of fences that look removable but usually aren't, with how to verify each.
- [removal-record.md](removal-record.md) — the deliverable when a fence is safely removed.
