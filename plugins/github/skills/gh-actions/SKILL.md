---
name: gh-actions
description: Inspect and operate GitHub Actions outside the context of one pull request. Use when the user asks about workflow runs on a branch, a scheduled or dispatch workflow, a deploy run, run history, or wants a run rerun or cancelled.
---

# GitHub Actions

## Overview

Workflow-level Actions work: run history on a branch, a deploy that failed, a
scheduled workflow that stopped firing, a run that needs cancelling.

For failing checks on a pull request, use `gh-fix-ci` instead. It starts from
the PR and gets to the failing log line in one call.

## Tools

- `github_list_workflow_runs` for run history, filtered by branch, status, or
  event. `event` is how you separate a scheduled run from a push run from a
  manual dispatch.
- `github_list_workflow_jobs` for the jobs in a run and which step failed.
- `github_get_job_log` for one job's failure snippet. Raise `max_lines` and
  `context_lines` when the default window cuts off the interesting part.
- `github_rerun_workflow_run` and `github_cancel_workflow_run` for run control.
  Both ask for approval.

## Workflow

1. Establish the repository, and the branch or workflow in question.
2. List runs with the narrowest filter that answers the question. Filtering by
   `event` and `branch` beats reading fifty runs and eyeballing them.
3. For a failed run, list its jobs, find the failing one and its failing step,
   then read that job's log. Do not read every job's log.
4. Report the run URL, the failing job and step, and the snippet.
5. Recommend an action. Distinguish clearly between:
   - A real failure in the code or config, which needs a fix.
   - Infrastructure or a dependency outage, which may need a rerun.
   - A flaky test, which needs to be named as flaky rather than quietly rerun.

## Rerun policy

A rerun is only correct when you have evidence the failure was not caused by
the code: a network timeout, a runner that died, a rate limit. Rerunning a real
failure costs minutes and teaches nothing, and rerunning to make a red check
turn green is hiding a problem. Say which of the three cases you believe this
is, and why, before proposing a rerun.

## Scheduled workflows

A schedule that stopped firing usually has a boring cause: GitHub disables
scheduled workflows on repositories with no activity for 60 days, the default
branch changed, or the cron was edited. Check the workflow file and the run
history for the last successful scheduled run before theorizing.

## Write safety

- Rerun and cancel both ask first. Restate the run id, the workflow name, and
  the branch before proceeding.
- Never cancel a run on a branch you were not asked about.
- Workflow dispatch can deploy. Confirm the target environment explicitly, and
  never dispatch a production workflow on your own initiative.
