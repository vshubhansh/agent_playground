# Assumption tells

The signals that you're holding a hidden assumption, each paired with its cheapest verification. When you notice a tell, add a ledger entry.

## Linguistic tells

These words are the sound of an untested bet. When one appears in your reasoning, stop and check.

| Phrase | What it usually hides | Cheapest check |
|--------|----------------------|----------------|
| "it should…" / "should be" | unverified behavior | run it / read the impl |
| "probably" / "I think" / "likely" | a guess wearing a confident hat | observe the real value |
| "I assume" / "presumably" | an admission you didn't check | check it now |
| "obviously" / "clearly" / "of course" | the assumption you're least likely to question | question exactly this one |
| "by default" | an assumed default value | read the actual config/signature default |
| "it always" / "never" | an unproven invariant | find one counterexample |
| "just" (as in "just call X") | hidden complexity being waved away | trace what X actually does |

## Behavioral tells

Patterns of action that smuggle in assumptions even without the giveaway words.

- **Reciting an API/signature from memory.** Memory drifts and versions change. → Read the actual definition or type; check the installed version's docs.
- **Trusting a default value.** "It defaults to true / 30s / UTC." → Read the real default in code/config, not the one you remember.
- **"It worked before / on my machine."** Past success ≠ present, here. → Reproduce in *this* environment, now.
- **Pattern-matching from another codebase or language.** "In the other repo this returned a promise." → Verify in *this* code; idioms don't transfer.
- **Assuming where something lives.** "The config is in `config/app.yml`." → Locate it before editing; don't edit the file you imagined.
- **Assuming input shape.** non-null, well-formed, sorted, non-empty, unique, within range. → Check the source of the input and existing guards; test the empty/null/huge case.
- **Assuming a function's call pattern.** called once, called in order, not re-entrant, not concurrent. → Search all call sites (including dynamic/string-keyed ones); check loops and async.
- **Assuming environment/version/locale.** OS, runtime version, timezone, encoding, feature flag state. → Read the actual env/lockfile/flag, don't assume the default profile.
- **Assuming "unused" / "safe to change".** → This is a fence: hand off to the `chestertons-fence` skill before removing.
- **Stating intent recovered from history as fact.** "They added this for performance." → That's an inference (`git-archaeologist` territory); keep it `Assumed` until the commit/PR confirms it.

## The meta-tell

The assumption you feel *most* certain about, and least want to spend time checking, is the one most worth a 30-second check. Strong conviction with no recent evidence is the highest-value ledger entry.
