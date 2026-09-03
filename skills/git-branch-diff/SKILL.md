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

### Step 0 — Refresh the base ref (avoid a stale base)

A local `master`/`main` — and even `origin/master` — can be days old, which silently makes
the whole comparison run against an outdated base. Before resolving anything:

1. Run `git fetch --quiet origin` (read-only; it updates remote-tracking refs only and
   never touches the working tree or the current branch).
2. If the fetch fails (offline, no remote, auth), do **not** stop — continue with the refs
   you have and **record that the base ref could not be refreshed**, so the caller can
   report it.
3. Never run `git pull`, `git merge`, `git rebase`, or anything else that changes the
   working tree or branch state.

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

### Step 4 — Collect base freshness facts

Three-dot diffing means base commits made after the merge-base are excluded — correct for
"what did this branch change", but it also hides that the branch may be built on an old
base. Collect these facts so the caller can judge (and report) the risk:

1. Whether the fetch in Step 0 succeeded.
2. Base tip and age: `git log -1 --format="%h %cI" <base>`.
3. Merge-base: `git merge-base <base> HEAD`.
4. How far the branch is behind: `git rev-list --count <merge-base>..<base>` — the number
   of commits the base gained since this branch forked/last synced.
5. Overlap risk: `git diff --name-only <merge-base>..<base>` — the files the base changed
   in the meantime. Report the intersection with the branch's own changed files (apply the
   caller's path scope), since those are the files where the branch's view of the code may
   already be outdated.
6. Conflict probe against the **current** base: `git merge-tree --write-tree <base> HEAD`.
   This computes the merge result **in memory** — it writes nothing to the working tree,
   the index, or any ref, so it is safe to run on a dirty workspace. A non-zero exit means
   the branch and the up-to-date base conflict; report the conflicting file paths it
   lists. If the installed git does not support `--write-tree` (pre-2.38), skip this and
   record "merge probe: skipped (git too old)".

Report these as plain facts; do not interpret them.

To read how a file looks on the **current** base (e.g. to check whether the base already
changed something the branch relies on), use `git show <base>:<path>` — also read-only.

## Hard rules

- DO NOT run `git status`, a bare `git diff` (no refs), `git diff HEAD` (working
  tree vs HEAD), or any workspace "changed files" / "get changed files" tool. These
  surface uncommitted edits and must not feed the review.
- DO NOT include staged-but-uncommitted changes. Only commit-to-commit differences
  (`<base>...HEAD`) count.
- DO NOT diff two-dot (`<base>..HEAD`) unless the caller specifically needs commits
  unique to HEAD without merge-base semantics; default to three-dot.
- If the working tree is dirty, ignore it — it does not change `<base>...HEAD`.
- DO run `git fetch --quiet origin` first (read-only). DO NOT run `git pull`, `git merge`,
  `git rebase`, or any command that modifies the working tree or branch state.
- DO NOT silently proceed on an unrefreshed base — if the fetch failed, say so in the
  freshness block.
- DO NOT analyze, summarize, interpret, categorize, or comment on the changes. DO NOT
  add tables, explanations, or per-file descriptions. Return the git output verbatim.

## Output

Return exactly, in this order, and nothing else:

1. **Base ref used** (e.g. `origin/master`) and **current branch** name.
2. **Base freshness** — fetch succeeded / failed (with reason), base tip `<short sha>`
   and its commit date, commits the base gained since the merge-base, the list of
   base-changed files that overlap the branch's changed files (or "none"), and the merge
   probe result (clean / conflicting files / skipped).
3. **Changed files** — the raw `--name-status` output.
4. **The diff** — the complete, unmodified `git diff <base>...HEAD` output (or the
   type-scoped diff if the caller requested one). Do not truncate or edit it.

If there are no committed differences between the branch and base, say so explicitly
rather than falling back to working-tree changes.
