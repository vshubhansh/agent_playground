# Removal record

The deliverable when a fence is removed. Its job: make sure the *next* archaeologist never has to wonder why this code vanished. Put it in the commit message (and/or a PR description). Only fill this out for a **Vestigial** verdict — `Confirmed`-safe removals.

```markdown
# Removed: <the construct — e.g. `exportLegacyReport` / the retry loop in upload.py>

**Original purpose:** <why it existed, quoting the introducing commit> (`a1b2c3d`, PR #640)

**Why it is now obsolete:** <positive evidence the original reason is gone>
- e.g. "The `legacy` report format was removed from config in `9f3a1c0`; no environment sets `report.format=legacy` (verified across config/ and env templates)."
- e.g. "The upstream 504 storms this retried were fixed by the gateway migration (`d4e5f6a`); error rate for this call has been 0 since."

**Confidence:** Confirmed — <what proves it; the removal does not ship below Confirmed>

**Residual risk / how to revert:** <honest statement of any path not fully ruled out, and the SHA to restore from if it bites>
```

## Rules for filling it in
- **Lead with the original purpose**, recovered from git — not your guess. Quote the commit.
- **Obsolescence needs positive evidence.** "Couldn't find a caller" is not enough; state what changed that made the fence unnecessary (dependency removed, bug fixed, flag retired, caller deleted), with the SHA.
- **Only `Confirmed` ships.** `Likely` or `Speculative` → verdict is Unknown; keep the fence and surface it to the human instead.
- **Always give the revert path** (the SHA the code can be restored from). A documented, reversible removal is cheap to undo; a silent one is expensive to rediscover.
