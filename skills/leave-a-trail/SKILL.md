---
name: leave-a-trail
description: >-
  Capture the why behind a change at the moment it's made, and route it to the
  right durable place — code comment, commit message, PR description, or ADR —
  so future readers never have to excavate it. Use when committing, writing a
  commit message, opening a PR, adding a workaround / guard / magic value /
  feature flag, choosing one approach over alternatives, making a non-obvious
  trade-off, or finishing a task.
---

# leave-a-trail

"Why" has a short half-life. The instant a change is made I know the alternatives I rejected, the constraint that forced the awkward bit, the bug a guard prevents — an hour later it's faded, and a year later someone runs `git-archaeologist` to recover it. This skill is the inverse of that dig: externalize intent **at creation time**, where a future reader will actually look.

**The discipline:** record the *why* — the problem, the rejected alternatives, the constraint — never the *what* the diff already shows. Match the record's durability to the decision's significance.

Depends on git + markdown only. No proprietary tooling.

## When this fires

Committing or writing a commit message; opening a PR; adding a workaround, guard, magic value, or feature flag; choosing one approach over a viable alternative; making a non-obvious trade-off; or finishing a task. The signal is a **fork**: a moment where a reasonable person would later ask "why did they do it *this* way?"

## Lay the trail

```
Trail checklist:
- [ ] 1. Notice the fork (a decision/trade-off/workaround just happened)
- [ ] 2. Capture the why (problem, rejected alternatives, constraint)
- [ ] 3. Pick the medium (route by significance + audience)
- [ ] 4. Right-size (one honest sentence > a paragraph of ceremony)
```

### 1. Notice the fork
Most lines need no trail. The ones that do are forks: a choice between viable options, a workaround, an unobvious constraint, a surprising value. If you'd have to explain it in review, it needs a trail.

### 2. Capture the why
Write down, while it's fresh:
- the **problem** this solves (or the bug it prevents),
- the **alternatives you rejected and why** — this is the single most-lost piece of information, the thing the archaeologist can almost never recover,
- the **constraint** that forced the shape (deadline, API limit, compat, perf).

Do **not** describe what the code does; the diff already shows that.

### 3. Pick the medium
Route by who needs it and how long it must survive:

| Medium | Use for | Lifespan |
|--------|---------|----------|
| **Code comment** | intent a reader *at this spot* needs and the code can't convey — why a guard, magic value, or workaround exists | as long as the line |
| **Commit message** | why this change as a unit; link the issue | forever, per-change |
| **PR description** | the change's story for reviewers: motivation, approach, alternatives, risk, test plan | forever, per-change-set |
| **ADR** | a significant, hard-to-reverse architecture/technology decision that outlives any single PR | forever, project-level |

See [commit-and-pr.md](commit-and-pr.md) for commit/PR craft and [adr-template.md](adr-template.md) for the ADR format.

### 4. Right-size
The trail must be cheaper to write than the future dig it saves. One honest sentence on a `// HACK` usually beats a formal record. Escalate to an ADR only when the decision is significant and hard to reverse — over-documenting trivia trains readers to ignore trails.

## Rules

- **Why, not what.** A comment that restates the code is noise; delete it. (If the code needs explaining, prefer making the code clearer first.)
- **Capture rejected alternatives.** "Used X" is weak; "used X instead of Y because Y deadlocks under N" is a trail.
- **Link, don't duplicate.** Point to the issue/ADR/PR rather than copying it; one source of truth.
- **Every workaround carries its reason and a link.** `// HACK`/`// workaround` without a why is a future mystery — exactly the fence `chestertons-fence` will refuse to touch.

## Worked example

> Change: replace a mutex around inventory updates with optimistic locking + retry.

- **Code comment** (at the retry): `// retry on version conflict; optimistic locking chosen over a mutex to avoid the cross-service deadlock in #931`.
- **Commit body:** states the problem (lock contention stalling checkout), the rejected alternative (a mutex — reintroduces the #931 deadlock across services), and the constraint (must not hold locks across the payment call).
- **ADR:** because it sets a concurrency *pattern* others will copy — records context, the decision, mutex-vs-optimistic trade-offs, and consequences.

Three trails, each sized to its audience — and no future archaeology required.

## Reference files
- [commit-and-pr.md](commit-and-pr.md) — intent-rich commit messages and PR descriptions, with good-vs-bad examples.
- [adr-template.md](adr-template.md) — the lightweight ADR format for significant decisions.
