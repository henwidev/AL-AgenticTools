# GitHub Copilot Agents & Skills

This repository contains custom GitHub Copilot agents, skills, and instructions for Business Central AL development.

---

## Agents

### Featured agents

#### AL PR Alignment Reviewer
Verifies that the code changes on a branch are aligned with the specification and documentation before a pull request is opened. Produces three difference lists: **spec vs. documentation**, **spec vs. code**, and **code vs. documentation**. Use this as a final quality gate before raising a PR or for checking a PR that is in place already.

#### AL Documentation Writer
Writes or updates customer-facing functional documentation for an AL feature. It first runs a spec-vs-code difference check and waits for confirmation before writing structured bilingual (German/English) documentation into the `docs/` folder. Can also invoke the **AL Changelog Writer** to append the changelog entry in the same run. Hands off to the AL PR Alignment Reviewer when done.

#### AL Changelog Writer
Appends a single customer-facing one-liner to the `changelogs/` folder for the current feature branch. Derives the change summary from committed and uncommitted AL object changes on the branch. Keeps changelog entries positive, non-technical, and scoped to what the customer sees.

#### AL API Modifier
Extends or modifies existing Business Central API pages and queries in their owning AL app while preserving historical contracts. Treats all externally observable contract changes, including field additions, as breaking: copies affected API objects with verified free IDs and names, exposes the changes only in a new minor `APIVersion`, and carries unchanged current endpoints into that release. Internal refactoring with no external behavior change stays in place without a version bump. Use **AL API Developer** for new APIs.

---

### Supporting agents

| Agent | Purpose |
|---|---|
| **Requirements Analyst** | Reviews Azure DevOps work items for quality: acceptance criteria completeness, vague language, and missing information. |
| **Tech Designer** | Produces a technical design for a Business Central requirement or feature, based on requirements-analyst output or a work item. |
| **AL API Developer** | Implements custom BC API endpoints (QueryType/PageType = API) in the sibling API app. |
| **AL Test Implementer (tegossuite)** | Fills in stubbed GIVEN-WHEN-THEN test procedures in AL test codeunits targeting the tegossuite (WMR) and the customer PTE. |

#### Sub-agents (not invoked directly)

| Agent | Purpose |
|---|---|
| **pr-spec-doc-diff** | Sub-agent of AL PR Alignment Reviewer. Compares specification against documentation and returns the diff list. |
| **ra-story-language** · **ra-scenario-structure** · **ra-docs-gap** · **ra-al-impact** | Sub-agents of Requirements Analyst covering language quality, GWT scenario structure, documentation gaps, and AL impact assessment. |
| **requirement-analyst-single** | Single-pass variant of the requirements analyst for lightweight checks. |

---

## Skills

| Skill | Purpose |
|---|---|
| **architecture-context** | Provides codebase architecture context to agents that need a structural overview. |
| **branch-workspace-diff** | Returns the combined diff of committed branch changes plus current uncommitted workspace changes. |
| **document-number** | Derives the ticket/document number from the current branch name. |
| **git-branch-diff** | Returns the committed diff of the current branch versus the base branch. |
| **gwt-scenario-review** | Reviews GIVEN-WHEN-THEN scenarios for completeness and structure. |

---

## Instructions

| Instruction | Purpose |
|---|---|
| [BC Dev Connection Defaults](instructions/bc-dev-connection.instructions.md) | Uses `http://BC28`, server instance `BC`, and environment type `OnPrem` for supported bc-dev MCP connection parameters unless another target is specified. Does not configure `launch.json`. |
| [Windows](instructions/windows.instructions.md) | Requires native Windows and PowerShell-compatible terminal commands; uses Linux commands only when explicitly targeting a Linux environment. |
| [German tegossuite terminology](instructions/tegossuite-terminology.instructions.md) | Defines English-German domain translations for tegossuite/enwis documentation, changelogs, and user-facing text, including term-specific usage notes. |
