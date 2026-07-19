---
name: daily-digest
description: Summarize notable repo changes and risks from the last 24 hours into a concise digest.
aliases: digest
---

You are running the daily-digest automation. Goal: produce a concise, skimmable digest of what changed in this repository over the last 24 hours and what deserves attention. This is a read-only report: make no changes, open no PRs.

## Scope

- Work only in the repository currently open. If the user passed arguments (a different time window like "last 3 days", a specific branch, or an area of the codebase), honor them; otherwise cover the last 24 hours on the default branch plus repo-wide PR and issue activity.
- If `gh` is unavailable or unauthenticated, fall back to a git-only digest and say so.

## Gathering steps

1. Commits: `git log --since="24 hours ago" --stat --no-merges` on the default branch (fetch first if the local clone may be stale: `git fetch origin`). Note merge commits separately with `git log --since="24 hours ago" --merges --oneline`.
2. Pull requests: `gh pr list --state all --search "updated:>=<ISO-date>"` to capture opened, merged, and closed PRs. For merged PRs, skim titles and sizes; open the diff of anything large or touching sensitive areas (auth, payments, migrations, CI config, dependency manifests).
3. Issues: `gh issue list --state all --search "updated:>=<ISO-date>"` for new and closed issues. Flag anything labeled bug, security, or regression.
4. Releases and tags: `git tag --sort=-creatordate | head -5` and `gh release list --limit 3` to catch anything shipped in the window.
5. Risk pass: for the changes you saw, look specifically for schema or data migrations, dependency bumps (especially major versions or security-sensitive packages), changes to CI/CD workflows, deleted tests, and public API changes. Read the actual diffs for anything you plan to flag; do not speculate from filenames.

## Safety rails

- Read-only. No commits, no branch creation, no PR or issue comments, no labels.
- Do not quote secrets, tokens, or credentials even if they appear in diffs; flag their presence as a finding instead.

## Output format

A digest with these sections (omit empty ones):

1. **Headline** (2-3 sentences): the shape of the day in this repo.
2. **Merged and shipped**: bullet list of merged PRs and releases, one line each, with PR numbers.
3. **In flight**: notable open PRs updated in the window, with state (draft, review, blocked).
4. **Issues**: new bugs and closed issues worth knowing about.
5. **Risks and follow-ups**: migrations, risky diffs, dependency bumps, deleted tests, CI changes, possible secret exposure. Each with a file path or PR number and a one-line why-it-matters.

Keep the whole digest under roughly 40 lines. Prefer dropping low-signal items over compressing everything into mush.
