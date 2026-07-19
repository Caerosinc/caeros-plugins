---
name: add-tests
description: Find recently changed high-risk logic lacking coverage, add focused tests, run them, and report results.
---

You are running the add-tests automation. Goal: locate recently changed, high-risk logic that lacks test coverage, write focused tests for it, run them, and report. Test files are the only thing you add; do not modify production code, and open a PR only if the user explicitly asks.

## Scope

- Work in the repository currently open. If the user passed arguments (a module, a diff range, "just report, no writes"), honor them. Default window: code changed in the last 14 days.
- Match the repo's existing test framework, layout, and naming exactly. Find a good existing test file first and use it as the template. If the repo has no test infrastructure at all, report that and propose a minimal setup instead of inventing one unasked.

## Investigation steps

1. **Find recent changes**: `git log --since="14 days ago" --name-only --no-merges`, ranked by churn. Include currently uncommitted changes (`git status`, `git diff`).
2. **Rank by risk**: prioritize logic where a bug is expensive or likely:
   - money, quantities, and rounding; date/time and timezone handling
   - parsing, validation, and sanitization of external input
   - concurrency, caching, and retry/timeout logic
   - branchy conditionals, error paths, and boundary arithmetic
   Deprioritize generated code, plumbing, and pure configuration.
3. **Check existing coverage**: for each candidate, look for tests that already exercise it (search for the symbol name in test files; use the repo's coverage tooling if configured and cheap to run). Skip anything adequately covered.
4. **Read before writing**: understand the function's contract, edge cases, and failure modes from the source and its call sites. Tests must assert real intended behavior, not merely mirror the implementation.

## Writing tests

- Small and focused: each test targets one behavior, named descriptively per repo convention.
- Cover the risk that made the code a candidate: boundaries, error paths, and the specific recent change, not just the happy path.
- Deterministic only: no real network, no wall-clock sleeps, no ordering dependence. Use the repo's existing fixtures and fakes.
- If a test you write fails because the production code appears genuinely buggy, do not "fix" the test to pass or patch production code. Keep the failing test (marked per repo convention if needed), and report the suspected bug prominently.

## Run and verify

- Run the new tests plus the touched packages' existing tests, using the repo's canonical runner (check CI workflow files or the Makefile for the real invocation).
- If anything is flaky, rerun to confirm and report the flake rather than papering over it.

## Output format

1. **Coverage gaps found**: ranked list with file, function, risk reason, and existing-coverage verdict.
2. **Tests added**: per file, which behaviors are now covered, plus the test run output summary (pass/fail counts, runtime).
3. **Suspected bugs**: any production defects your tests exposed, with evidence.
4. **Skipped**: candidates you left alone and why.
5. **Next step**: note the tests are uncommitted local changes; offer to commit or open a PR on explicit request.
