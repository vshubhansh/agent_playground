# Excavation toolkit

Copy-paste git incantations, grouped by the question you're trying to answer. Every command is portable (standard git, no extensions). Replace `<...>` placeholders.

## "When was this exact string or code added or removed?"

The **pickaxe** is the sharpest tool here — it beats `blame` because it finds the *birth* and *death* of code, not just the last touch, and it works even on code that no longer exists in HEAD.

```bash
git log -S'<exact string>' --oneline --all          # commits that change how many times the string appears
git log -S'<exact string>' -p --all                 # ...with the introducing/removing diff
git log -G'<regex>' --oneline --all                 # commits whose added/removed lines match a regex
git log --pickaxe-regex -S'<regex>' --oneline        # treat the -S argument as a regex
```

- Use `-S` for a literal token (a constant, flag, function name). It triggers only when the *count* of that string changes — precise for "added/removed".
- Use `-G` to match any diff hunk touching a pattern (e.g. every commit that edited a `try/except` near a name). Noisier, but catches modifications too.
- `--all` searches every ref (branches, tags), so origins on since-merged branches aren't missed.
- The **oldest** commit in the output is usually the true origin; the newest may be a later edit or revert.

## "See through reformatting and code movement"

`blame` reports the last commit to touch each line, which is often a reindent or a move. Strip that noise:

```bash
git blame -w -C -C -C -L <start>,<end> -- <file>
```

- `-w` — ignore whitespace-only changes (defeats reformat-pollution).
- `-C` — detect lines moved/copied within the same file.
- `-C -C` — also detect lines moved/copied from other files **modified in the same commit**.
- `-C -C -C` — also detect from **any** file in the commit, even unmodified. Slowest, most thorough.
- `-L <start>,<end>` — restrict to a line range (also accepts `-L :<funcname>:<file>` to blame a function by name).

To watch a single line's history evolve commit by commit:

```bash
git log -L <start>,<end>:<file>        # line-range log: every change to those lines, with diffs
git log -L :<funcname>:<file>          # same, scoped to a named function
```

## "Trace a file across renames and moves"

```bash
git log --follow -p -- <file>          # full history including the file's life under former names
git log --follow --stat -- <file>      # lighter: just what changed and when
```

`--follow` is essential — without it, history stops at the rename and you'll wrongly date the file to its move.

## "Which PR or merge brought this in?"

```bash
git log --merges --ancestry-path <sha>..HEAD --oneline    # merge commits on the path from origin to now
git show <merge-sha>                                       # merge message often names the PR (#1234)
git log --oneline <sha>~1..<sha>                           # the commit itself, to scan its message for refs
```

Most workflows put `#<number>` (GitHub/GitLab) or a tracker key (e.g. `PROJ-123`) in commit or merge messages. Grep for them:

```bash
git show <sha> | rg -o '#[0-9]+|[A-Z]+-[0-9]+'    # harvest PR/issue references from a commit
```

## "What changed alongside it?" (the surrounding story)

A commit is rarely about one line. Read its whole footprint to understand the feature/fix it served:

```bash
git show <sha> --stat                  # files touched + churn — the shape of the change
git show <sha>                         # the full diff + message
git show <sha>:<file>                  # the file's full contents AS OF that commit
```

## Quick repo reconnaissance

```bash
git rev-parse --is-shallow-repository  # "true" => history is truncated (see gotchas)
git log --oneline -5                   # is this a real history or a single squashed import?
git log --format='%h %ad %an %s' --date=short -- <file>   # compact, citation-ready file history
```

## Gotchas

- **The top blame line lies.** It's the last edit, not the origin. Always corroborate with the pickaxe before stating who/why.
- **Shallow clones** (`--is-shallow-repository` is `true`) have truncated history; run `git fetch --unshallow` (or note the limitation) before claiming an origin.
- **Squash-merge workflows** collapse a branch into one commit — intermediate reasoning is gone. The squash message + linked PR are the best surviving record.
- **`-S` counts occurrences**, so a commit that both adds and removes the same string equally won't show up; fall back to `-G`.
- **Rebases/force-pushes** rewrite SHAs; a SHA referenced in an old ticket may no longer exist — search by message or content instead.
- **Vendored/generated code** may have no meaningful authorship; date it to its import, and say the true origin is upstream.
