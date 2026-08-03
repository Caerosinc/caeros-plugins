---
name: github
description: Orient and triage GitHub repository, pull request, and issue work through the native GitHub app, then route to the right specialist workflow. Use when the user asks for general GitHub help, wants a PR or issue summary, or needs repository context before a more specific GitHub task.
---

# GitHub

## Overview

This is the entry point for GitHub work. Its job is to establish which
repository and item you are operating on, answer the request directly when it is
plain triage, and hand off to a specialist skill the moment the task has a
shape.

Everything runs through the native `github_*` tools. They call the GitHub API
with the token Caeros holds: no CLI login, no local checkout, no connector
broker. Use `git` and `gh` only for the local checkout, for pushing, and for
cross-repository or fork PR heads.

Do not linger here. Once the category is clear, route and stop.

## Resolve context first

1. A repository, PR number, issue number, or URL from the user always wins.
2. For "this branch" or "the current PR", resolve the local git context, then
   find the PR with `github_list_pull_requests`, or `gh pr view --json number,url`.
3. If the repository is still ambiguous, ask for it. There is no repository
   search here, so guessing produces confident nonsense.

## Route

| The user wants | Go to |
| --- | --- |
| Failing checks, red CI, "why is the build broken" | `gh-fix-ci` |
| Review comments, requested changes, unresolved threads | `gh-address-comments` |
| A review OF a PR, "review this PR", find problems in a diff | `gh-review` |
| Commit, push, open a PR | `gh-publish` |
| Which PRs need attention, PR backlog, stale PRs | `gh-triage-prs` |
| Issue backlog, labeling, duplicates | `gh-triage-issues` |
| Workflow runs, reruns, dispatches outside one PR | `gh-actions` |
| Cutting a release, release notes | `gh-release` |
| Branch protection, labels, workflow scaffolding | `gh-repo-setup` |

Two builtin commands overlap and are often the better answer. `/pr` runs the
commit, push, and open flow with its command approvals already armed. `/review`
runs an in-chat review of the current diff. Prefer them when the user is
working from a local checkout and say that is what you are doing.

## Handle directly

Stay here only for orientation and light triage:

- Repository state: `github_get_repo`, `github_list_branches`.
- One PR at a glance: `github_get_pull_request`, plus
  `github_list_pull_request_files` for the shape of the change.
- One issue: `github_get_issue`.
- A quick cross-repository question: `github_search_issues` with GitHub search
  syntax, for example `repo:owner/name is:pr is:open review-requested:@me`.

If answering means reading logs, walking review threads, or writing code, that
is a specialist skill. Route.

## Output

- For triage, lead with what needs attention and why, not with a table dump.
- When routing, say which path you are taking in one line before you take it.
- Before any write, restate the exact target: repository, number, and what will
  change.
- Never claim GitHub Actions logs, review-thread state, or check results
  without having read them.

## Not available here

- Merging and closing. Caeros suggests, the human acts. Say what you would
  merge, then let the user do it.
- Repository search by topic or description.
- Any read of an external check provider's logs. Those are report-only.
