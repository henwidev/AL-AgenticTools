---
name: branch-workspace-diff
description: >-
  Returns the combined code changes a branch delivers: the committed branch-vs-base diff
  (via the git-branch-diff skill) PLUS the current uncommitted workspace changes (staged
  and unstaged). Keeps the two sources separate, then presents them together as raw data.
  Output only — performs no analysis. Keywords: branch plus workspace changes, committed
  and uncommitted, work in progress diff, what this branch delivers, combined diff,
  staged unstaged plus PR diff.
argument-hint: "Optional: base branch (defaults to master, then main); optional path scope (e.g. *.al)"
---

# Branch + Workspace Diff (committed branch diff plus uncommitted workspace changes)

Return one combined view of everything a branch currently delivers: the **committed**
changes it introduced versus its base branch, **plus** the **uncommitted** work-in-progress
in the workspace (staged and unstaged). The two sources are collected separately and then
presented together.

This skill is a **data provider only**. The invoking agent does the analysis.

## ⛔ Two distinct change sources — keep them separate, then combine

- **Branch (committed) changes** — obtain these **only** via the `git-branch-diff` skill
  (`<base>...HEAD`, three-dot/merge-base). For this part you must NOT use `git status`, a
  bare `git diff`, `git diff HEAD`, or a "changed files" / `get_changed_files` view — that
  is exactly what the `git-branch-diff` skill forbids.
- **Workspace (uncommitted) changes** — obtain these **separately and deliberately** with
  `git diff` (unstaged) and `git diff --staged` (staged). This is the intentional second
  input — the in-progress work — not a substitute for the branch diff.

Both inputs are valid and required. Only the **branch** diff carries the "committed-only"
rule (always via the skill); the **workspace** diff is explicitly the uncommitted
work-in-progress.

## Procedure

### Step 1 — Committed branch changes (delegate to `git-branch-diff`)

Load and follow the **`git-branch-diff`** skill, passing through the caller's optional
base branch and the optional path scope (e.g. `-- "*.al"`). From its output, capture:
- the **base ref** it resolved and the **current branch** name,
- its **base freshness** block (fetch outcome, base tip + date, commits the base gained
  since the merge-base, base-changed files overlapping the branch's changed files, merge
  probe result),
- the committed **changed files** (`--name-status`),
- the committed **diff** (`<base>...HEAD`, scoped if requested).

Do not re-implement the committed-diff logic here — the `git-branch-diff` skill owns it.

### Step 2 — Uncommitted workspace changes

Collect the working-tree changes directly, scoped the same way as the caller requested
(e.g. `-- "*.al"`; omit the scope for all files):

1. Unstaged: `git diff <scope>`
2. Staged: `git diff --staged <scope>`

### Step 3 — Present both, combined

Return the committed and uncommitted parts together, clearly labeled so the caller can
tell which change came from which source.

## Hard rules

- DO NOT obtain the **committed** part from `git status`, a bare `git diff` (no refs),
  `git diff HEAD`, or any "changed files" / `get_changed_files` tool — use the
  `git-branch-diff` skill.
- DO use `git diff` and `git diff --staged` for the **uncommitted** part — that is the
  deliberate exception and the whole point of this skill.
- DO apply the same path scope to both sources when the caller requests one.
- DO NOT analyze, summarize, interpret, categorize, or comment on the changes. Return the
  git output verbatim.

## Output

Return exactly, in this order, and nothing else:

1. **Base ref used** (e.g. `origin/master`) and **current branch** name.
2. **Base freshness** — the block returned by `git-branch-diff`, passed through verbatim.
3. **Committed — changed files** — the raw `--name-status` output from `git-branch-diff`.
4. **Committed — diff** — the complete `git diff <base>...HEAD` output (scoped if requested).
5. **Uncommitted (unstaged) — diff** — the complete `git diff <scope>` output.
6. **Uncommitted (staged) — diff** — the complete `git diff --staged <scope>` output.

For any of the three diff sources that is empty, say so explicitly (e.g. "no uncommitted
unstaged changes") rather than omitting it silently. If the base branch cannot be
resolved, report that (as `git-branch-diff` does). If all sources are empty, say there are
no committed and no uncommitted changes.
