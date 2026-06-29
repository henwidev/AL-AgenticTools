---
name: document-number
description: >-
  Resolves the ticket/document number for the current git branch from its name (e.g.
  SC-XXXXXX, HD-XXXXXX, DLA-XXXXXX, CS-XXXXXX), discarding any trailing branch suffix.
  Used to name changelog and documentation files. Output only — performs no analysis, and
  never guesses. Keywords: document number, ticket number, SC-XXXXXX, HD-XXXXXX, DLA, CS,
  branch name parsing, feature branch, file name resolution.
argument-hint: "Optional: the document number, if it cannot be derived from the branch name"
---

# Document Number (resolve ticket/document number from the branch)

Return the **document number** of the current branch so a caller can name a ticket-scoped
file (e.g. `changelogs/{DOCUMENT-NUMBER}.md` or `docs/{DOCUMENT-NUMBER}-{lang}.md`).

This skill is a **data provider only** — it returns the document number and nothing else.

## Procedure

1. If the caller supplied a document number, validate it against the accepted prefixes
   below and use it.
2. Otherwise get the current branch: `git rev-parse --abbrev-ref HEAD`.
3. Branch names start with `feature/<document number><unrelated>`. Extract **only** the
   document number — a recognised prefix followed by its digits:
   - Accepted prefixes: `SC-`, `HD-` (most common), also `DLA-` and `CS-`, each followed
     by the ticket digits (e.g. `SC-025072`, `HD-019345`).
4. **Discard everything after the document number.** The branch may append unrelated
   information — user shortcuts/initials, or words like `feedback` or `correction` — and
   none of that may end up in the result (e.g. `feature/SC-025072-jw-feedback` →
   `SC-025072`).
5. If no document number can be derived from the branch name (and none was supplied),
   **ask the user for it. Never guess it.**

## Hard rules

- DO NOT guess or invent a document number. If it cannot be derived and was not supplied,
  ask the user.
- DO NOT include any trailing branch suffix (initials, `feedback`, `correction`, etc.).
- DO NOT add a file extension, folder, or language suffix — return only the bare document
  number; the caller composes the file name.

## Output

Return the resolved **document number** (e.g. `SC-025072`) and, briefly, the **branch
name** it was derived from. If it cannot be resolved, say so explicitly and that the user
must supply it.
