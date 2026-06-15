---
name: git-branch-diff
description: >-
  Returns the raw committed diff of the current git branch versus a base branch
  (PR-style, three-dot/merge-base), ignoring uncommitted and staged working-tree
  edits. Output only — performs no analysis. Keywords: branch vs master, branch vs
  main, PR diff, git diff base...HEAD, merge-base, committed changes only.
argument-hint: "Optional: base branch (defaults to master, then main)"
---

# Git Branch Diff (committed branch vs base)

Return the raw code changes a branch introduces versus its base branch, the same
way a pull request shows them. Compare **commit to commit**, so uncommitted and
unstaged working-tree edits are never included.

This skill is a **data provider only**. The invoking agent does the analysis.

## Why this matters

A workspace "changed files" view (and `git status` / a bare `git diff`) reports
**uncommitted working-tree edits** — local noise like editor settings, version
bumps, or in-progress work. That is NOT the branch'\''s contribution. Reviews must use
the **committed** difference between the branch tip and the base branch only.

## Procedure

### Step 1 — Resolve the base branch

1. If the caller supplied a base branch, use it.
2. Otherwise auto-detect, preferring `master`, then `main`:
   - `git rev-parse --verify --quiet origin/master` → if it exists, base is `origin/master`.
   - else `git rev-parse --verify --quiet origin/main` → base is `origin/main`.
   - else fall back to the local `master`, then local `main`.
3. Prefer the `origin/<base>` remote-tracking ref when it exists, so the comparison
   reflects the real base and not a stale local copy. If working offline, the local
   branch is acceptable — note which ref you used.
4. If neither base can be resolved, stop and report that the base branch was not found.

### Step 2 — Resolve the current branch

- `git rev-parse --abbrev-ref HEAD` → the current branch name (for reporting).
- The committed tip to review is always `HEAD`.

### Step 3 — Collect the committed diff (three-dot / merge-base)

Always use the **three-dot** form so you compare `HEAD` against the merge-base of
`<base>` and `HEAD` — i.e. exactly the changes this branch introduced, ignoring
commits that landed on the base afterwards:

1. File overview (status + path):
   `git diff <base>...HEAD --name-status`
2. Full diff:
   `git diff <base>...HEAD`
3. Scoped diff only when the caller asked for specific file types (e.g. AL):
   `git diff <base>...HEAD -- "*.al"`

## Hard rules

- DO NOT run `git status`, a bare `git diff` (no refs), `git diff HEAD` (working
  tree vs HEAD), or any workspace "changed files" / "get changed files" tool. These
  surface uncommitted edits and must not feed the review.
- DO NOT include staged-but-uncommitted changes. Only commit-to-commit differences
  (`<base>...HEAD`) count.
- DO NOT diff two-dot (`<base>..HEAD`) unless the caller specifically needs commits
  unique to HEAD without merge-base semantics; default to three-dot.
- If the working tree is dirty, ignore it — it does not change `<base>...HEAD`.
- DO NOT analyze, summarize, interpret, categorize, or comment on the changes. DO NOT
  add tables, explanations, or per-file descriptions. Return the git output verbatim.

## Output

Return exactly, in this order, and nothing else:

1. **Base ref used** (e.g. `origin/master`) and **current branch** name.
2. **Changed files** — the raw `--name-status` output.
3. **The diff** — the complete, unmodified `git diff <base>...HEAD` output (or the
   type-scoped diff if the caller requested one). Do not truncate or edit it.

If there are no committed differences between the branch and base, say so explicitly
rather than falling back to working-tree changes.
