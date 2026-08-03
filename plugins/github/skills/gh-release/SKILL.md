---
name: gh-release
description: Prepare a release: work out what changed since the last release, draft release notes from merged pull requests, and check that the release build is green. Use when the user asks to cut a release, write release notes, or find out what is shipping.
---

# Prepare a Release

## Overview

Answer what is shipping, write notes a user can read, and confirm the build is
green. Publishing the release itself stays with the human.

## Tools

- `github_get_repo` for the default branch.
- `github_search_issues` with `repo:owner/name is:pr is:merged base:main merged:>2026-07-01`
  to find what merged in a window. This is the backbone of the notes.
- `github_get_pull_request` and `github_list_pull_request_files` when a PR title
  is too thin to describe what actually changed.
- `github_list_workflow_runs` filtered to the release branch to confirm the
  build is green.
- `github_list_branches` for existing release branches.
- `git tag --sort=-creatordate` and `git log <last-tag>..HEAD --oneline` in a
  local checkout for the exact commit range. The API cannot see your working
  tree, and tags are the reliable boundary.

## Workflow

1. Establish the boundary: the previous release tag, or a date, or a base
   branch. Ask if it is unclear. A guessed boundary produces notes that are
   confidently wrong about what shipped.
2. List merged PRs in the range with `github_search_issues`.
3. Group by what the change means to a reader, not by the label the repository
   happens to use:
   - New capability
   - Behavior change, including anything that could break an existing user
   - Fix, with the symptom the user would have seen
   - Internal, which usually belongs in a short trailing list or nowhere
4. Read the PRs whose titles do not explain themselves. A note that just
   restates a commit subject is not worth writing.
5. Check the build. `github_list_workflow_runs` on the release branch. Report
   the state rather than assuming it.
6. Produce the notes and the checklist of what remains for the human: tag,
   publish, announce.

## Writing the notes

- Lead with what a user can now do that they could not before.
- Every behavior change gets its own line, stated as what changed and what to
  do about it. Users find breakage here or in production.
- Fixes name the symptom, not the internal cause. "Fixed a crash when opening a
  file with no extension" beats "fixed nil deref in parsePath".
- Credit contributors when the repository does that already.

## Write safety

- Do not create tags, publish releases, or push to a release branch. Produce
  the notes and the exact commands, and let the user run them.
- Do not invent a version number. Ask which one this is.
- Never state a build is green without having read the run status.
