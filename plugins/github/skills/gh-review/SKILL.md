---
name: gh-review
description: Review a GitHub pull request for correctness, security, and design problems, and report the findings in chat. Use when the user asks to review a PR, asks what is wrong with a diff, or wants a second opinion before approving. Nothing is posted to GitHub unless the user explicitly asks.
---

# Review a Pull Request

## Overview

Read a PR properly and report what is actually wrong with it. The default
output is a review IN CHAT. Nothing is posted to GitHub unless the user asks
for that in so many words.

For the current local diff, the builtin `/review` command is the better path.
Use this skill when the target is a PR on the remote, possibly one you have no
checkout for.

## Tools

- `github_get_pull_request` for the description and state. Read the intent
  before judging the code.
- `github_list_pull_request_files` with `include_patch: true` for the change,
  file by file.
- `github_get_pull_request_diff` when you want the whole thing at once. Large
  diffs come back truncated and flagged, so check the flag before concluding
  you have seen everything.
- `github_list_pr_checks` for CI state. Do not review code whose tests you have
  not looked at.
- `github_pull_request_feedback` to see what other reviewers already said, so
  you are not the third person to raise the same point.
- `github_get_file` to read the surrounding code the diff does not show.

Only on an explicit request: `github_submit_review`, `github_create_pr_comment`.

## Workflow

1. Read the intent: PR title, description, and linked issues.
2. Read the check state. A red PR often has its answer in the log already.
3. Read the diff file by file, and pull surrounding context with
   `github_get_file` when a hunk does not stand on its own. Most real bugs live
   in the interaction between the change and the code around it, which the
   patch does not show.
4. Read the existing review threads before writing anything.
5. Work through the dimensions deliberately rather than reading once for
   everything:
   - Correctness: the failure case the author did not consider. Nil, empty,
     concurrent, retried, partially applied.
   - Security: input that crosses a trust boundary, credentials in logs, path
     traversal, authorization checked at the wrong layer.
   - Interface: does the change fit how the rest of this codebase already does
     the same thing.
   - Tests: is the risky path covered, or only the easy one.
6. Verify each finding before reporting it. State the concrete input and the
   wrong result. A finding you cannot make concrete is a question, so ask it as
   one.

## Output

- Order by severity. A correctness bug first, a naming preference last or not
  at all.
- Each finding: file and line, what breaks, and the input that breaks it.
- Say plainly when you found nothing serious. Padding a review with nits to
  look thorough wastes the author's time.
- Note what you did not review: truncated diff, generated files, a subsystem
  you lack context for.

## Write safety

- The review is for the chat. Do not post it to GitHub unless asked.
- Never submit an APPROVE review on the user's behalf without an explicit
  instruction. An approval is attributable to them and can unblock a merge.
- Never resolve someone else's review thread as part of a review.
