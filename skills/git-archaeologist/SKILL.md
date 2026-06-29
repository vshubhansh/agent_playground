---
name: git-archaeologist
description: >-
  Reconstruct the origin story and intent behind code using git history alone —
  blame, the pickaxe (log -S/-G), rename-following, and commit/PR archaeology —
  producing an evidence-cited timeline. Use when asked why a line, function,
  magic constant, flag, or workaround exists, who wrote it and why, where code
  came from, when something was added or removed, the origin of a bug, or to dig
  through git history / git blame.
---

# git-archaeologist

Code outlives the reasons it was written. Reading *what* code does is easy; recovering *why* it exists is the hard, usually-skipped work — and the record is already buried in git. This skill is the discipline for digging it up properly and reporting it with evidence.

Depends on **git only**. No network, no language runtime, no proprietary tooling. Works on any repo.

## The one mistake this prevents

The lazy move is to read the top `git blame` line and declare the origin. That line is *frequently* a reformat, a code move, or a rename — not the real birth of the code. An archaeologist never dates an artifact by the dust on top of it. Always dig through that noise (`blame -w -C -C -C`, the pickaxe, `--follow`) before concluding anything.

## The Excavation

Copy this checklist and work it in order. Stop early only when the artifact's true origin commit and its intent are both established with evidence.

```
Excavation progress:
- [ ] 1. Define the artifact (what exactly are we dating?)
- [ ] 2. First dig (candidate commits, seeing through noise)
- [ ] 3. Find true origin (pickaxe + rename-following)
- [ ] 4. Read the why (commit message, siblings, PR/issue refs)
- [ ] 5. Synthesize (evidence-cited timeline + conclusion)
```

### 1. Define the artifact
Pin down precisely what you are dating before touching git. One of:
- specific **lines** in a file → use line-scoped blame.
- a **symbol** (function/class/var) → blame its definition lines.
- an exact **string / magic constant / flag name** → use the pickaxe; it beats blame for this.
- **deleted** code (it's gone from HEAD) → only the pickaxe over full history can find it.

### 2. First dig
Find candidate commits while seeing through whitespace, moves, and copies:

```bash
git blame -w -C -C -C -L <start>,<end> -- <file>
```

`-w` ignores whitespace, and the three `-C` flags trace lines back through moves within the file, moves from other files in the same commit, and moves from any file in the commit's history. This is what separates the real author from whoever last reindented the block.

### 3. Find the true origin
Blame shows the *last* touch. To find when code was truly born (or died), use the pickaxe and follow renames:

```bash
git log -S'<exact string>' --oneline --all        # commits that changed the COUNT of this string (added/removed)
git log -G'<regex>' --oneline --all                # commits whose diff matches this regex
git log --follow -p -- <file>                      # full history of a file, across renames
```

Use `-S` for "when did this literal token appear/disappear", `-G` for patterns. Add `-p` to read the introducing diff. The oldest matching commit is usually the real origin.

### 4. Read the why
The commit is the testimony. Open it fully:

```bash
git show <sha>            # message + full diff
git show <sha> --stat     # what else changed alongside (the feature/fix it belonged to)
```

The **message** states intent; the **sibling changes** reveal the larger feature or fix. Harvest any `#1234`-style PR/issue references and merge context — that's the trail to the original discussion:

```bash
git log --merges --ancestry-path <sha>..HEAD --oneline   # the merge/PR that brought it to the main line
```

### 5. Synthesize
Assemble findings into the deliverable in [report-template.md](report-template.md). Lead with a one-sentence answer, back it with a citation-tagged timeline.

## Rules of evidence

- **Cite every claim with a short SHA.** No SHA, no claim.
- **Separate fact from inference.** "The diff adds the retry loop" is fact. "They added it to handle flaky upstream timeouts" is inference — label it.
- **Confidence labels:** mark each conclusion `Confirmed` (diff/message proves it), `Likely` (strong circumstantial evidence), or `Speculative` (plausible, unproven).
- **Never conclude intent from a single blame line.** Corroborate with the pickaxe and the commit message.
- **Report dead ends honestly.** "Introduced in the initial import (`a1b2c3d`), no prior history" is a valid, useful finding — don't invent a backstory.

## Graceful degradation

History isn't always complete. Detect and disclose:
- **Shallow clone** (`git rev-parse --is-shallow-repository` → `true`): history is truncated; suggest `git fetch --unshallow` before concluding origin.
- **Squash-merge workflows:** intermediate commits are gone; the squash commit message and its linked PR are the best available record.
- **Vendored / imported code:** origin may predate this repo entirely; say so rather than dating it to the import.

## Worked example

> Question: "Why is there a `MAX_RETRIES = 7` in `client.py`? Seven is oddly specific."

1. Artifact: an exact constant → pickaxe, not blame.
2. `git log -S'MAX_RETRIES = 7' --oneline --all` → one commit, `9f3a1c`.
3. `git show 9f3a1c --stat` → the same commit adds an exponential-backoff helper and edits `upload.py`.
4. Message: *"fix(client): retry uploads through gateway 504 storms (#812)"*. PR #812 is the discussion trail.

**Answer:** `Confirmed` — `MAX_RETRIES = 7` was introduced in `9f3a1c` as part of a backoff fix for gateway 504 storms (PR #812). `Likely` — 7 was chosen so the cumulative exponential backoff exceeds the gateway's ~2-minute outage window; the exact arithmetic isn't stated in the commit, so treat the specific number's rationale as `Speculative` pending PR #812.

## Reference files
- [excavation-toolkit.md](excavation-toolkit.md) — copy-paste git incantations grouped by the question you're answering, plus gotchas.
- [report-template.md](report-template.md) — the origin-story deliverable format.
