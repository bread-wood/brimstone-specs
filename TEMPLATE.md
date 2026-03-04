# [Repo] [Version] — [Theme]

> **Who reads this file and why**
>
> This spec is the authoritative statement of *what* this milestone delivers and *why*.
> It is consumed by:
> - **research agents** — investigate Key Unknowns; findings constrain design choices
> - **design agents** — use Scope, Constraints, and Failure Modes to bound design docs
> - **impl agents** — use Success Criteria as acceptance criteria for each issue
> - **qa** — runs every Success Criterion and every Failure Mode as a test case
> - **human reviewers** — use Non-Goals to prevent scope creep
>
> Write this file before seeding issues. Keep it product-focused (what and why),
> not technical (not how). The how belongs in design docs produced during the design stage.
> Do not prescribe issue decomposition — that is the planning agent's job.

---

## Overview

> 3–5 sentences. What does this milestone deliver, why does it exist now, and what does it
> unblock? Name any hard prerequisites (milestones that must complete first). Avoid
> implementation detail — focus on the problem being solved and the capability being added.

[Replace with overview. Include prerequisite sentence if applicable:
"**Prerequisite:** X must complete before this milestone — [one sentence why]."]

---

## Success Criteria

> Binary, testable, observable. Each criterion must be independently verifiable.
> Use exact commands, exact outputs, exact conditions.
>
> Good: `` `bw snapshot --format=jobman` exits 0 and produces JSON with `schema_version: "1"` ``
> Bad: "Snapshot output should be well-formed"
>
> Avoid "should" and "may" — use exact observable outcomes.
> Criteria become qa test cases. If you can't write a test for it, it doesn't belong here.

- [ ] [Criterion]
- [ ] [Criterion]
- [ ] `make test && make lint` pass on mainline

---

## Scope

> What IS included. Be explicit — if it's not listed, agents will not build it.

**Included:**

- [Component or capability]
- [Component or capability]

**Explicitly excluded:**

- [Thing that might seem in scope but isn't — with a pointer to where it lives if known]

---

## Failure Modes

> For each significant boundary this milestone touches, specify what happens when it
> fails. These become qa test cases. Group by boundary type.
>
> Format: condition → expected behavior (log level, alert, retry, degrade, or fail fast).

**[Boundary name — e.g. "External API", "Job execution", "DB write"]:**
- [Failure condition] → [expected behavior]
- [Failure condition] → [expected behavior]

**[Another boundary]:**
- [Failure condition] → [expected behavior]

---

## Interface Contracts

> The data shapes and API surfaces that cross boundaries. These are the contracts
> other repos or stages will code against — they must be stable before impl begins.
> Include only contracts that cross a boundary (repo, process, or DB schema).
> Omit internal implementation details.

**[Contract name — e.g. "REST API", "Job handler input", "SQLite schema", "CLI"]:**

```
[schema, interface signature, or example payload]
```

---

## Constraints

> Hard limits the implementation must respect. Non-negotiable. Include:
> - Dependency restrictions (no direct imports between X and Y)
> - Schema versioning requirements
> - Environment / runtime requirements
> - Backward-compatibility requirements

- [Constraint]
- [Constraint]

---

## Key Unknowns

> Only questions where the **answer changes the design**. If knowing the answer
> wouldn't change what you build or how you build it, omit it. Aim for 1–3 maximum.
>
> Priorities:
> - **P1** — must be resolved before design begins (blocks the design stage)
> - **P2** — should be resolved before impl begins (important but not blocking)
>
> Write as a question whose answer determines an implementation approach or boundary.
> Do NOT enumerate candidate solutions in the question — that is the research agent's job.
> Ask what needs to be decided, not how to decide it.
>
> If a question has an obvious answer that most practitioners would agree on,
> it is not a Key Unknown — make the call and state it in Constraints.
>
> For cross-repo coordination points (where multiple repos share one Key Unknown):
> state which other specs share this unknown and name the canonical output artifact
> that all repos will reference (e.g. a shared contract file at a stable path).

1. **[P1]** [Question: answer determines a major design decision]

---

## Non-Goals

> Explicitly excluded. Prevents scope creep and tells agents what NOT to build.
> Be specific — vague exclusions are ignored.
> Add a version pointer if the item is deferred rather than permanently excluded.

- [Excluded feature or concern — "(v0.X.0)" if deferred]
- [Excluded feature or concern]
