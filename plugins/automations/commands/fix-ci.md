---
name: fix-ci
description: Investigate the latest failing CI run, root-cause it, and propose or apply a fix on a branch.
aliases: ci-fix
---

You are running the fix-ci automation. Goal: find out why CI is failing on this repository, root-cause it, and prepare a fix. Report first; open a PR only if the user explicitly asks.

## Scope

- Work only in the repository currently open. Detect it with `git remote -v` and `gh repo view`.
- If the user passed arguments (a run ID, a branch, a workflow name, or "open a PR"), honor them. Otherwise target the default branch's most recent failing run.
- If `gh` is unavailable or unauthenticated, say so and stop; do not guess at CI state.

## Investigation steps

1. List recent runs: `gh run list --limit 15` (add `--branch <branch>` if the user named one). Identify the most recent failed or cancelled run relevant to the target branch.
2. Inspect it: `gh run view <run-id>` to see which jobs failed, then `gh run view <run-id> --log-failed` to pull only the failing logs. For long logs, grep for `error`, `FAIL`, `panic`, `Traceback`, and exit codes rather than reading everything.
3. Classify the failure:
   - Code regression: find the commit that introduced it (`git log`, compare against the last green run's SHA, use `gh run list` timestamps).
   - Flaky test or infrastructure issue: check whether the same job passed recently on the same SHA, or whether the failure is a timeout, network error, or runner outage.
   - Config or dependency drift: lockfile conflicts, tool version bumps, expired secrets or tokens (report these; never print secret values).
4. Reproduce locally when feasible: run the failing test or build step exactly as CI does (read the workflow file under `.github/workflows/` to get the real command). Do not spend more than a few attempts on flaky reproduction; note flakiness instead.
5. Root-cause precisely: name the file, line, commit, and mechanism. Read the relevant source before concluding; do not diagnose from log text alone.

## Fixing

- For a clear, low-risk fix: create a branch (`git checkout -b fix/ci-<short-slug>`), apply the minimal change, and rerun the failing test or build locally to verify.
- Keep the diff tightly scoped to the failure. No drive-by refactors, formatting sweeps, or unrelated cleanups.
- For flaky tests: prefer fixing the underlying race or ordering issue. Only suggest a retry or skip as a last resort, clearly labeled as a mitigation, not a fix.
- Never force-push, never rewrite shared history, never edit workflow permissions or secrets.

## Output format

Report back with:

1. **Failing run**: run ID, workflow, branch, trigger commit.
2. **Root cause**: one paragraph, with file paths and the offending commit if identified.
3. **Fix**: what you changed (or propose changing) and how you verified it, including local test output.
4. **Confidence and risks**: anything you could not verify, plus flakiness observations.
5. **Next step**: state that you can open a PR on request. Open one (`gh pr create` from your fix branch with a clear title and body) only if the user explicitly asked for a PR in this invocation.
