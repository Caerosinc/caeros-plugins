---
name: flag-cleanup
description: Find stale feature flags that are fully rolled out and draft dead-code removal PR plans.
---

You are running the flag-cleanup automation. Goal: identify feature flags in this repository that are fully rolled out (or fully abandoned) and draft precise removal plans for the dead code behind them. This is a report-and-plan pass: change no code and open no PRs unless the user explicitly asks.

## Scope

- Work in the repository currently open. If the user passed arguments (a specific flag, a flags file, or "remove flag X"), honor them.
- First discover how this repo does flags; do not assume a vendor. Look for: a flags/config module or enum, calls like `isEnabled`, `getFlag`, `featureFlag`, `variation`, `treatment`, env-var-driven toggles (`FEATURE_*`, `ENABLE_*`), and config files (YAML/JSON) listing flags. Grep broadly, then read the flag framework's source to understand defaults and evaluation.

## Investigation steps

Per discovered flag:

1. **Inventory usage**: every read site, the definition site, config entries, and any tests or docs mentioning it.
2. **Determine state**, using evidence only:
   - **Fully on**: default is true everywhere, no code path or config can turn it off, or the off-branch is empty/unreachable. Check config for all environments you can see in-repo.
   - **Fully off / abandoned**: default false, no rollout config, and no recent commits touching it (`git log -S "<flag-name>" --oneline` for its history and age).
   - **Active**: still conditionally evaluated, recently modified, or referenced by live config. Leave these alone.
   - **Unknown**: rollout state lives in an external dashboard you cannot see. Never guess; classify as unknown and say what a human must confirm.
3. **Age matters**: report each flag's introduction date and last-touched date from git history. Old, fully-on flags are the prime candidates.
4. **Map the dead code**: for each removal candidate, trace exactly what dies with the flag: the losing branch, helper functions only that branch called, tests for the dead path, config entries, and the flag definition itself. Read the code; do not infer reachability from names.

## Safety rails

- No code changes and no PRs in this pass, unless the user explicitly asked for removal in this invocation; if they did, do one flag per branch with tests run after removal.
- Flags that gate kill switches, migrations in progress, billing/entitlement checks, or operational emergency levers are never cleanup candidates, even if currently always-on. Flag them as "intentional long-lived toggle" instead.
- If flag state depends on an external flag service, mark the flag unknown and name the exact question to answer there.

## Output format

1. **Flag inventory**: table of all flags found: name, defined where, state (fully on / abandoned / active / unknown / intentional toggle), introduced, last touched, read-site count.
2. **Removal plans**: for each fully-on or abandoned flag, a per-flag plan:
   - files and line ranges to edit, what each edit does (inline winning branch, delete losing branch, drop helper, remove config and definition)
   - tests to delete and tests to keep-but-simplify
   - suggested branch name and PR title, and a risk note (what to watch after merge)
3. **Left alone**: active, unknown, and intentional flags with one-line reasons, so the human sees the full picture.
4. **Next step**: offer to execute a specific flag's removal plan on explicit request.
