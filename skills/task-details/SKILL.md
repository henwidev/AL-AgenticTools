---
name: task-details
description: >-
  Resolves local AL task details by document number or task folder. Finds the task
  description, internal comment, and diagnosis documentation for writing or reviewing
  AL feature docs and authoring test specifications, with legacy documentation fallback.
argument-hint: "Optional: task folder, task file path, or document number"
---

# Resolve local task details

This is a read-only input resolver. Return the document number, resolved task directory,
the sources used (paths or supplied text), and any missing or ambiguous inputs. Do not
fetch a work item, write files, or change the caller's scope or diff rules.

## Find the task folder

1. Use explicitly supplied or handed-over text and file paths for each input role
   before looking it up. An attached/named task folder, or a task file in such a
   folder, establishes the lookup directory and its document number. Prefer a
   supplied or handed-over document number to the branch-derived number. If explicit
   inputs point to different tickets, ask which to use.
2. Otherwise, when lookup or an output path is needed, resolve the number with
   `document-number`. Locate `docs\{DOCUMENT-NUMBER}` under the relevant AL app root
   (the directory with `app.json`, with `changelogs` alongside `docs` where present).
   Do not assume the git root is the app root. Prefer the app identified by the
   supplied context or a unique matching ticket folder; if multiple apps remain
   plausible, ask for the folder instead of picking the first match. If the task
   folder does not exist, return its intended path under the resolved app's `docs`
   directory. If all required inputs are already supplied and no output path is
   needed, do not stop to resolve a missing number or app solely for optional inputs.
3. Read missing inputs by role from that directory:
   - `task-description.md` is the specification, including any acceptance criteria.
     Use relevant local images referenced by the task when needed to understand it;
     do not automatically follow external links.
   - `internal-comment.md` is optional internal context, not a requirement or
     customer-facing documentation. Preserve the caller's rules for how it can be used.
   - Use the documentation selection below when the caller needs documentation.

   Report a missing required source to the caller only after checking its applicable
   fallbacks; do not ask for absent optional inputs. If an explicitly supplied path
   cannot be read, report the path and ask for a correction rather than silently
   substituting another file.

## Select documentation

Use explicit documentation text or a supplied path first. Otherwise, use the first
applicable source in this order:

1. `diagnosis-comment-agent-{lang}.md` in the task folder, with `-de` or `-en` matching
   the specification. If only one language file exists, it can be read; report its
   language if different. If both exist and the language is unclear, use the
   latest-modified file as described below.
2. `diagnosis-comment.md` in the task folder. It is existing functional documentation
   even if it is bilingual or uses a different layout. Do not confuse it with the
   specification or the internal comment.
3. Legacy `docs\{DOCUMENT-NUMBER}-{lang}.md` at the same app root, using the same
   language-selection rules. Check this even when the ticket folder exists but has no
   diagnosis documentation.

When both language variants in the same selection tier exist and the specification's
language is unclear, inspect both files' filesystem modification times, for example with
`Get-Item -LiteralPath $candidatePaths | Select-Object FullName, LastWriteTimeUtc`
where `$candidatePaths` contains the two actual paths. Choose the file with the later
`LastWriteTimeUtc`; report the selected path, both timestamps, and that recency was the
tie-breaker. If either timestamp is unavailable or they are equal, ask which file to use.
Modification time is a heuristic for the current working tree, not proof of the last
authored edit: a checkout, copy, or sync may change it. If it is known to be misleading,
ask instead. A supplied source, the selection-tier order, and a known matching
specification language always take precedence over modification time.

For example, `docs\HD-044079\task-description.md` supplies the specification and
`docs\HD-044079\diagnosis-comment.md` can supply the documentation. The writer's German
output is `docs\HD-044079\diagnosis-comment-agent-de.md`, not a replacement for the
existing diagnosis comment. The reviewer can use that generated file on its next run.
