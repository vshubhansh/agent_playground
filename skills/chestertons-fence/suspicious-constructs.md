# Suspicious constructs — the catalog of fences

Code that looks removable but is frequently load-bearing. For each: why it's usually there, and how to verify before removing. Pair every check with the git side (find the introducing commit via the `git-archaeologist` skill) — the commit message often names the exact problem the construct solved.

## Defensive checks & "dead" branches
- **Null/undefined/bounds guards, `if (!x) return`, optional chaining.** Usually added after a real production crash. Verify: who calls this with possibly-empty input? Check SSR/first-render, async-not-yet-loaded, empty-collection, and error paths. A guard with no current trigger may still fire in an environment you're not testing.
- **`else` / `default:` / `case` that "can't happen".** Often catches an enum value added later or malformed external data. Verify: is the input type truly closed, or does it come from the network/DB/user? Removing the catch-all can turn a handled case into a silent fall-through.

## Timing & resilience
- **Retries, backoff, `maxRetries`, circuit breakers.** Added for a flaky dependency. Verify: is that dependency still flaky? Removing retries shifts failures onto users. Find the incident/PR that added it.
- **Timeouts, `sleep`, `setTimeout(…, n)`, `await delay()`.** Magic waits usually paper over a race condition or rate limit. Verify: what breaks if it's 0? Often nothing in dev and everything under load. Treat as load-bearing until proven otherwise.
- **`try/catch` that swallows a specific error.** The narrower the caught error, the more deliberate it is. Verify: what throws it, and what happens if it propagates? Swallowing may be hiding an expected, benign failure (e.g., cache miss, optional file).

## Reachability illusions ("unused" code)
- **"Unused" exports / public API / library entry points.** Static grep can't see external consumers. Verify: is this a published package, plugin, or public interface? Search downstream repos if possible; treat exported surface as used-by-default.
- **Dynamically referenced code.** The big source of false "unused" signals. Reachable via:
  - **String keys / registries:** `handlers['foo']`, `formatters[type]` → search for the string, not the symbol.
  - **Reflection / metaprogramming:** `getattr`, `Object.keys`, decorators, `__getattr__`, method dispatch by name.
  - **Dependency injection / service containers:** wired by type or token, never called directly.
  - **Serialization:** field names must match a wire/DB/JSON schema even if "unused" in code.
  - **Config-/env-/route-driven dispatch:** selected at runtime by a value in config, env var, feature flag, or a route table.
  - **Framework lifecycle hooks / conventions:** named methods invoked by a framework (e.g. lifecycle callbacks, test hooks, CLI command auto-discovery).

## Correctness quirks that look like mistakes
- **Locale / timezone / encoding handling, `toUTC`, `normalize()`.** Looks redundant; prevents real cross-environment bugs. Verify against non-default locale/timezone, not just yours.
- **Odd rounding, epsilon comparisons, `+ 0.0001`, integer casts.** Usually fix a floating-point or off-by-one bug. Verify with the edge inputs that motivated them (find them in the commit/tests).
- **Seemingly redundant re-assignments, copies, `.slice()`/`clone()`.** Often defend against shared-mutable-state aliasing bugs. Verify whether the source is mutated elsewhere.

## Explicit "do not touch" signals
- **Comments: `// do not remove`, `// HACK`, `// workaround for <bug>`, `@deprecated`.** Take literally. A linked bug/issue is the verification trail — read it; confirm it's resolved before removing.
- **Lint/type suppressions: `eslint-disable`, `// @ts-ignore`, `# noqa`, `#[allow(...)]`.** Mark a known, accepted exception. Removing the code (or the suppression) may resurface the original error.
- **`@deprecated` but still called.** Deprecated ≠ unused. Verify call sites are actually gone, including dynamic ones, before deleting.
- **Polyfills / shims / browser/version guards.** Support an environment you may not test. Verify the minimum supported runtime still needs them before dropping.

## Verification quick-reference

```bash
rg "exportName"            # direct references
rg "'exportName'|\"exportName\""   # string-key / dynamic references
rg "deprecated|do not remove|workaround|HACK" path/    # explicit signals nearby
git log -S'<code>' --all   # when + why it was introduced (then: git show <sha>)
```

When in doubt, the construct is a fence. Recover its purpose or leave it standing.
