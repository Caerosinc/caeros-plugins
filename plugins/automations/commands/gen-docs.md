---
name: gen-docs
description: Find recently changed or under-documented modules and draft documentation updates for them.
---

You are running the gen-docs automation. Goal: find the modules in this repository where documentation is most stale or missing relative to the code, and draft the updates. Present drafts for review; only write files into the repo when the user has asked you to apply changes, and never open a PR unless explicitly told.

## Scope

- Work in the repository currently open. If the user passed arguments (a module, "apply the changes", a doc style, or a target like README vs API docs), honor them.
- Documentation means whatever this repo actually uses: README files, docs/ trees, doc comments (godoc, JSDoc, docstrings), CHANGELOG, configuration reference. Detect the convention before writing anything.

## Investigation steps

1. **Map the doc landscape**: locate README(s), docs/ directories, and the dominant doc-comment style. Note the tone and structure of the best existing page so your drafts match it.
2. **Find recently changed code**: `git log --since="14 days ago" --name-only --no-merges` (widen the window if quiet). Rank touched modules by churn.
3. **Find the gaps**, prioritizing:
   - Recently changed modules whose docs were not touched in the same window (code moved, docs did not).
   - Exported/public APIs with no doc comments, especially ones referenced from many places.
   - Setup and configuration drift: flags, env vars, or commands that exist in code but not in the README, and vice versa (stale docs describing removed behavior are the worst offenders; verify against source).
4. **Verify before writing**: every claim in a draft must come from reading the actual code, not from the old doc or from inference. Run commands you document (build, test, CLI help) when cheap and side-effect free, and copy real output.

## Drafting rules

- Match the repo's existing voice, heading style, and formatting conventions.
- Prefer fixing wrong docs over adding new ones; prefer documenting public surface over internals.
- Keep each draft focused: a doc-comment block, a README section, or one docs page per item. No sweeping rewrites.
- Include runnable, tested examples where the module lends itself to them.

## Safety rails

- Default is draft-and-report: show proposed content inline (or as diffs) without writing to the repo. Apply edits only when the user asked for that in this invocation, and even then keep commits out of it unless requested.
- Never delete existing documentation wholesale; propose replacements alongside what they replace.

## Output format

1. **Gap list**: ranked table of modules/files with missing or stale docs, each with a one-line reason and evidence (commit or code reference).
2. **Drafts**: for the top 3-5 gaps, the proposed documentation content in full, labeled with its target file path and whether it is new or a replacement.
3. **Deferred**: gaps you found but did not draft, so nothing silently drops.
4. **Next step**: offer to apply the drafts (and, separately, to open a PR) on explicit request.
