---
name: architecture-context
description: >-
  Load when a requirement touches system boundaries, integrations, non-functional
  requirements (NFRs), quality attributes, or architectural decisions. Provides
  analysis lenses from C4 Model, arc42, Martin Fowler, and ThoughtWorks to
  sharpen requirements assessment. Keywords: boundary, integration, NFR, quality
  attribute, performance, scalability, ADR, bounded context, container, system,
  architecture decision, cross-cutting.
argument-hint: "Requirement text or work item with architectural scope"
---

# Architecture Context — Analysis Guidance

Load this skill when a requirement references **system boundaries, integrations,
non-functional requirements, quality attributes, or architectural decisions**. Use
the lenses below to add depth to the standard requirements assessment.

---

## Lens 1 — C4 Model: Scope & Boundary Clarity

**Source:** c4model.com

### Core vocabulary

| Term | Meaning |
|------|---------|
| `Person` | A human role that interacts with the system |
| `Software System` | A deployable, independently operable unit (e.g. BC, Azure Function) |
| `Container` | A runtime unit inside a system (web app, database, service bus) |
| `Component` | A logical grouping inside a container (codeunit, module) |
| `Boundary` | The line between in-scope and out-of-scope |
| `Relationship` | A dependency, call, or data flow between two elements |

### How to apply in requirements analysis

1. **Scope check** — Can you draw a C4 Context diagram from the story alone?
   Flag if the story does not name the `Person` (actor), the `Software System`
   being changed, or the external systems it touches.
2. **Boundary crossing** — If the story's WHEN clause crosses a container or
   system boundary (e.g. BC → Azure Function → email), flag each integration
   point as a dependency that requires its own acceptance criterion.
3. **Missing actor** — If no `Person` is identified in the story, the scope
   is ambiguous. Flag for PO clarification.

### Flag template

```
ARCHITECTURE FLAG [C4 — Boundary]: [describe the unclarified boundary or missing actor]
  Impact: [what is unknown as a result]
  Question for PO: [specific question]
```

---

## Lens 2 — arc42: NFR Completeness & Quality Scenarios

**Source:** arc42.org/overview

### Quality goals (arc42 Section 1 + 10)

arc42 maps to **ISO 25010** quality characteristics. When a requirement implies
a non-functional need, check whether it is expressed as a measurable quality scenario:

| ISO 25010 Characteristic | Examples of implied NFR in a story |
|---|---|
| Performance Efficiency | "fast", "real-time", "instantly", "no delay" |
| Reliability | "always available", "no data loss", "failover" |
| Security | "only authorised users", "audit trail", "encrypted" |
| Maintainability | "easy to configure", "no code change required" |
| Usability | "intuitive", "user-friendly", "one click" |
| Portability | "works on any device", "cloud-native" |

### Quality scenario format

A well-formed quality scenario has these parts:

```
Stimulus:    [who/what triggers the behaviour]
Environment: [system state when it happens]
Response:    [what the system does]
Metric:      [measurable threshold — the acceptance criterion]
```

**Example:**
> _Vague:_ "The page must load quickly."
> _Quality scenario:_ Stimulus: user opens the Workshop Feedback list.
> Environment: 500 records in the table, standard BC SaaS.
> Response: page renders fully.
> Metric: within 2 seconds, measured end-to-end.

### How to apply in requirements analysis

1. Scan story text for vague quality adjectives (fast, secure, reliable, scalable).
2. For each one found, check if a measurable metric exists anywhere in the story.
3. If no metric exists, flag using the template below.
4. Check arc42 Section 2 (Constraints) vocabulary: is the NFR actually a constraint
   (a fixed boundary that cannot be traded off) rather than a goal?

### Flag template

```
ARCHITECTURE FLAG [arc42 — NFR]: "[quoted vague term]" lacks a quality scenario.
  ISO 25010 characteristic: [e.g. Performance Efficiency]
  Missing metric: [what threshold would make this testable]
  Question for PO: [specific question]
```

---

## Lens 3 — Martin Fowler: Architectural Decisions & Reversibility

**Source:** martinfowler.com/architecture/

### Key principle

> Architecture = the decisions that are hard to change.

When a story **encodes an architectural decision** (e.g. "store data in a separate
table", "call via REST", "use event-driven processing"), that decision should be
captured as an **Architecture Decision Record (ADR)**, not buried in a story.

### ADR format (lightweight)

```
Context:      [Why does this decision need to be made?]
Decision:     [What was decided?]
Consequences: [What becomes easier? What becomes harder?]
```

### Reversibility check

For every design choice implied in the story, ask:

- **Reversible in a sprint?** → implementation detail, no flag needed
- **Reversible in a quarter?** → architectural trade-off, flag as a decision point
- **Reversible only with significant rework?** → flag as a hard architectural constraint

### Fitness functions

If the story implies a quality property that should be continuously guarded
(e.g. "response time must never exceed 3s"), flag that this property is a candidate
for a **fitness function** — an automated check in CI that enforces the architectural
rule over time.

### Flag template

```
ARCHITECTURE FLAG [Fowler — ADR]: Story implies an architectural decision.
  Decision embedded: [describe the implicit decision]
  Reversibility: [reversible / trade-off / hard constraint]
  Recommended action: [capture as ADR / clarify with architect]
```

---

## Lens 4 — ThoughtWorks: Domain Language & Team Constraints

**Source:** thoughtworks.com/insights/topics/software-architecture

### DDD vocabulary check (Bounded Context / Ubiquitous Language)

When a story uses business terms, verify:

1. **Ubiquitous Language** — does the term have a single, agreed definition within
   this BC module/domain? If the same word means different things in different
   parts of the codebase or business, flag it.
2. **Bounded Context crossing** — if the story moves data between two domains
   (e.g. Sales and Inventory), flag the translation/mapping requirement. There must
   be an explicit acceptance criterion for how conflicts are resolved.

| DDD Term | Relevance to requirements |
|---|---|
| `Bounded Context` | Each context owns its model; cross-context data needs explicit mapping |
| `Ubiquitous Language` | Terms in GIVEN/WHEN/THEN must match the domain glossary |
| `Domain Event` | If the story triggers a business event, name it explicitly |
| `Aggregate` | If the story modifies related entities atomically, identify the aggregate root |

### "Just enough architecture" check

ThoughtWorks principle: capture what's needed for the **next decision**, not
the full future. Flag if a story is trying to solve architecture problems
three sprints away — it may be premature and should be split.

### Conway's Law / team ownership

If a story requires changes in a system or service owned by a **different team**,
flag the dependency. Stories that cross team boundaries require explicit coordination
acceptance criteria (who delivers what, by when).

### Flag template

```
ARCHITECTURE FLAG [TW — Domain]: "[quoted term]" is ambiguous across bounded contexts.
  Contexts affected: [list]
  Question for PO: [clarify canonical definition or owner]
```

---

## When to raise an ARCHITECTURE FLAG vs. a LANGUAGE FLAG

| Situation | Flag type |
|---|---|
| Vague adjective with no metric (fast, secure) | ARCHITECTURE FLAG [arc42 — NFR] |
| Term undefined in the domain glossary | ARCHITECTURE FLAG [TW — Domain] |
| Design choice embedded in story text | ARCHITECTURE FLAG [Fowler — ADR] |
| Story scope crosses system boundary | ARCHITECTURE FLAG [C4 — Boundary] |
| Ambiguous pronoun or unclear subject | LANGUAGE FLAG (ra-story-language) |

---

## Output block

Append this block to the assessment when architectural flags are found.
If no flags are raised, omit the block entirely.

```
ARCHITECTURE ANALYSIS:
  C4 BOUNDARY FLAGS:
    - [flag or "none"]
  NFR / QUALITY SCENARIO FLAGS:
    - [flag or "none"]
  ARCHITECTURAL DECISION FLAGS:
    - [flag or "none"]
  DOMAIN LANGUAGE FLAGS:
    - [flag or "none"]
  ARCHITECTURE VERDICT: [clean / needs attention / blocking]
    [1 sentence: what is the highest-priority architectural risk in this requirement]
```
