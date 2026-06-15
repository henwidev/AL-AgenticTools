---
name: gwt-scenario-review
description: >-
  Use when acceptance criteria are written in GIVEN-WHEN-THEN format (lines
  starting with GIVEN / WHEN / THEN, scenario blocks, test automation
  scenarios). Checks alignment with task description. Keywords: GIVEN
  WHEN THEN, GWT, acceptance test automation, scenario structure. Do NOT load
  for plain-prose or bullet-list criteria, or when no acceptance criteria are
  present.
argument-hint: "Paste or reference the requirement with GIVEN-WHEN-THEN scenarios to review"
---

# GIVEN-WHEN-THEN Scenario Review

## Procedure

### Step 1 — Locate all GIVEN-WHEN-THEN scenarios

1. Find every block that uses GIVEN / WHEN / THEN in the requirement.
2. Assign each block a short label (use the scenario name if present, otherwise
   its first few words) to reference in the report.

### Step 2 — Check GIVEN clause

For each scenario, verify:
- Does GIVEN describe **concrete data and preconditions**?
  (e.g. specific records, field values, user roles)
- Flag if GIVEN is vague — e.g. "the system is configured", "the user has access"
  without specifying what configuration or access level.

### Step 3 — Check WHEN clause

- Must contain **exactly one clause** per scenario.
- Must describe **a single user action**, starting with a verb
  (e.g. "Create a new…", "Delete the…", "Submit the…").
- Flag if: multiple actions are chained, the phrasing describes a state instead
  of an action, or the verb is missing.

### Step 4 — Check THEN clause

- Must verify the **outcome of the WHEN action** using data from GIVEN.
- Must be **specific and checkable** — no vague language like "works correctly",
  "is handled", "displays the right value".
- Flag if the outcome cannot be independently verified from the scenario's own
  GIVEN data.

### Step 5 — Check alignment

- Does each scenario test behaviour that is **described in the task description**?
- Flag any scenario that tests behaviour outside the task scope, or that
  contradicts the task description.

### Step 6 — Report

Output **only** the structured block below — no introduction, no summary prose:

```
ACCEPTANCE CRITERIA: [present / missing / insufficient]
  GIVEN-WHEN-THEN: [yes / no / partial]
  SCENARIO ISSUES: [none / list per scenario]
    - Scenario "[label]": [GIVEN / WHEN / THEN] — [what is wrong]
  ALIGNMENT: [aligned / mismatched / not applicable]
    [if mismatched: quote the scenario and explain the conflict]
```
