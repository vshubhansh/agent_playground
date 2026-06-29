# Origin-story report template

The deliverable for an excavation. Lead with the answer; back every claim with a short SHA. Keep it as long as the evidence warrants and no longer.

```markdown
# Origin of: <the artifact — e.g. `MAX_RETRIES = 7` in client.py>

**TL;DR:** <one sentence answering "why does this exist?", tagged with confidence and the key SHA>

## Timeline
| SHA | Date | Author | What changed | Why (inferred unless quoted) |
|-----|------|--------|--------------|------------------------------|
| `a1b2c3d` | 2023-04-11 | A. Dev | Introduced the constant alongside a backoff helper | Quoted from message: "retry through gateway 504 storms" |
| `9f3a1c0` | 2024-01-02 | B. Dev | Bumped 5 → 7 | Inferred: align with longer gateway outage window |

## Smoking gun
- **Origin commit:** `a1b2c3d` — <what it proves>.
- **Discussion trail:** PR #812 / TICKET-123 (from the commit message).

## Conclusion
- `Confirmed`: <claims the diff or message directly proves>
- `Likely`: <claims with strong circumstantial support>
- `Speculative`: <plausible but unproven; say what would confirm it>

## Open questions / dead ends
- <e.g. "Exact rationale for the number 7 isn't in any commit; would need PR #812 comments.">
- <e.g. "History is shallow before 2023-01; earlier origin can't be ruled out.">
```

## Rules for filling it in
- **TL;DR first.** The reader wants the answer, then the evidence.
- **Every row needs a SHA.** A timeline row without a citation is a guess, not a finding.
- **Quote messages verbatim** when stating intent from a commit; paraphrase only your own inferences and label them.
- **Three confidence tiers**, never silently mix them: `Confirmed`, `Likely`, `Speculative`.
- **List the dead ends.** Honestly stating what you couldn't recover is more useful than a confident fabrication, and tells the reader where to look next.
- Keep the table to the commits that actually move the story — omit unrelated touches.
