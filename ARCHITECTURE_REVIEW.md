# Architecture Review — Abstraction & Separation of Concerns

**Repos:** brimstone · jobman · moot · breadwinner
**Date:** 2026-03-04

Evidence base: all 16 milestone specs + source files
`moot/heartbeat.py`, `moot/scheduler.py`, `breadmin_reddit/sync.py`,
`breadwinner/llm/base.py`, `breadwinner/llm/provider.py`,
`breadwinner/handlers/breadmin.py`

---

## Summary

Three hard violations, four soft violations, and four design decisions that need
explicit answers before the integration milestones (Wave 3) can be designed correctly.
The hard violations will cause runtime failures or schema breakage if left in place.
The design decisions are currently recorded as Key Unknowns in individual specs but
need cross-repo resolution — they cannot be answered per-spec in isolation.

---

## Hard Violations

### H1 — moot owns a scheduler, but jobman is the scheduler

**Where:** `src/moot/scheduler.py`, `src/moot/heartbeat.py`

`moot/scheduler.py` implements a `Scheduler` class with `run_morning_cycle()`,
`run_evening_cycle()`, and `run_weekly_cycle()` methods. `moot/heartbeat.py` is
a 400+ line pipeline runner that loads agents, fetches library data, checks budgets,
runs batch API calls, and writes results. Both are invoked via moot's own CLI or
launchd plist — entirely outside of jobman.

This is a direct role duplication. Jobman's entire purpose is to own the scheduling
layer. If moot runs its own scheduler, there is no single source of truth for what
is scheduled, no unified retry/failure handling, no audit trail in the jobman DB, and
no way for jobman's heartbeat to surface moot failures. The two schedulers will also
fight over resource consumption without either knowing about the other.

**What should happen:** moot's cycles (morning, evening, weekly) become breadmin job
handlers. `moot-heartbeat`, `moot-morning-cycle`, `moot-evening-cycle` register in
jobman's handler registry like `breadwinner-heartbeat` already does. `moot/scheduler.py`
and the CLI entry point for `moot/heartbeat.py` are deleted. The logic stays in moot,
but the invocation and scheduling ownership moves to jobman.

**Affected specs:** moot/v0.4.0-council-spine should add "migrate moot scheduler to
jobman job handlers" to its cleanup scope. This is currently missing from the spec.

---

### H2 — breadmin-reddit imports moot directly

**Where:** `breadmin_reddit/sync.py` line 293

```python
def _write_to_moot(self, snapshots: list[SubredditSnapshot]) -> int:
    ...
    from moot.libraries.discourse import DiscourseLibrary
```

The reddit package lazily imports moot and calls `DiscourseLibrary` directly. This
creates a hard package dependency between `breadmin-reddit` and `moot`. If moot's
library API changes, the reddit package breaks silently at runtime (the import is
lazy). If moot is not installed in the same environment, this fails at call time
with no warning at import time.

This also contradicts the jobman architecture: reddit is a jobman package, and all
data flow to moot should route through jobman's coordination layer, not through
direct package imports. The reddit package should not know that moot exists.

**What should happen:** `_write_to_moot()` is deleted from `breadmin_reddit/sync.py`.
The reddit sync job writes aggregated data to a signals store owned by jobman
(a SQLite table in jobman's data dir). Moot reads from this store via the interface
defined in moot/v0.7.0-open-field. The `DiscourseLibrary` write becomes moot's
responsibility when it consumes the signals store — not reddit's.

**Affected specs:** jobman/v0.3.0-signals-pipeline Key Unknown #2 is "should the
reddit job write directly to moot or to an intermediate store?" This is not actually
a question — the direct write is wrong. The spec should resolve this Key Unknown
as "intermediate signals store" and remove it as a question.

---

### H3 — moot has a parallel cost/budget system

**Where:** `src/moot/heartbeat.py` — calls `check_budget()`, `estimate_cost()`

Moot maintains its own daily budget tracking and cost estimation, independent of
the token usage logging in breadmin-llm. Breadmin-llm logs every LLM call with
token counts to a `token_usage` table in the breadmin DB. Moot's budget system
maintains a separate ledger in its own DB.

Two cost tracking systems will diverge immediately — they use different tables,
different estimation methods, and different reset cadences. Any cost dashboard
that reads one will miss the other. Budget circuit breakers built on either system
will be incomplete.

**What should happen:** Moot's `check_budget()` and `estimate_cost()` are removed.
Budget tracking is breadmin-llm's job — it already does it. Any budget enforcement
moot needs (e.g., "stop running heartbeats if daily spend exceeds $X") reads from
breadmin-llm's `token_usage` table or from a breadmin API endpoint that aggregates
costs. This is a single source of truth for the whole platform.

**Affected specs:** moot/v0.4.0-council-spine cleanup scope should include removing
moot's budget system. Currently missing.

---

## Soft Violations

### S1 — breadwinner's `llm/` directory is named misleadingly but correctly implemented

**Where:** `src/breadwinner/llm/base.py`, `src/breadwinner/llm/provider.py`

`base.py` defines breadwinner-specific context dataclasses (`PositionContext`,
`AnalysisContext`, `DeploymentPlanResult`, etc.). `provider.py` wraps breadmin-llm's
`ProviderRegistry` — it does not reimplement provider logic. The comment in
`provider.py` is explicit: "Wraps the breadmin-llm ProviderRegistry."

This is actually correct. Breadwinner needs its own context types because they carry
domain knowledge (positions, exit rules, P&L). The wrapping layer is thin and
appropriate. The only issue is naming: calling it `llm/` implies it might be a
provider implementation, when it's really a domain-specific prompt-and-parse layer.

**Recommended change:** Rename `src/breadwinner/llm/` to `src/breadwinner/prompts/`
(or `src/breadwinner/inference/`). The directory contains prompt builders, response
parsers, and context types — not LLM providers. This is cosmetic but prevents future
contributors from adding provider code here instead of in breadmin-llm.

---

### S2 — The verdict signal schema has no canonical home

**Where:** moot/v0.6.0 Key Unknown #1, jobman/v0.2.0 Key Unknown #1,
breadwinner/v0.4.0 Key Unknown #1

All three specs name the verdict handler interface as their top Key Unknown and each
expects "research" to answer it. But this is not a per-repo research question —
it is one schema that three repos must agree on. The dependency graph correctly
identifies this as a coordination point, but none of the specs say where the output
lives after research.

A schema without a canonical home will drift. Each repo's implementation will
interpret the agreed document slightly differently, and there will be no authoritative
file to resolve disagreements.

**Recommended change:** The shared research output should be committed to one repo
as a file at a stable path that the other two reference explicitly. The most natural
home is `breadmin-jobman/docs/contracts/verdict-signal-v1.json` (jobman is the bus,
so it owns the bus contracts). Both moot and breadwinner specs should name this path
as the authoritative schema location — not a Slack message or a shared research doc
that lives nowhere.

---

### S3 — jobman/knowledge-wire reads moot's DB via raw SQL

**Where:** jobman/v0.1.0 spec — `LibraryReader` does direct SQL against moot.db

The knowledge package's `LibraryReader` reads from moot's `library_*` tables via
direct SQL with no interface contract. If moot renames a library table, adds a column
with a NOT NULL constraint, or changes a field name, the knowledge reader breaks
at runtime with no compile-time signal.

This is a pragmatic choice (no HTTP overhead, no API to define) but it creates
invisible coupling. Every moot DB migration is now also a knowledge package migration.

**Recommended change:** Moot should expose a narrow read API — either a REST endpoint
(`GET /libraries/summaries`) or a stable SQLite view (`CREATE VIEW library_summaries
AS ...`) that jobman reads from. The view approach has zero overhead and can be
maintained as a stable contract even as the underlying tables change. Add it to
moot/v0.4.0-council-spine scope (it's a one-time schema addition).

If the view approach is chosen: add it to moot/v0.4.0, reference it in
jobman/v0.1.0's Key Unknown #1 resolution (push vs pull becomes "read via stable
view"), and remove the Key Unknown from the spec.

---

### S4 — moot/v0.5.0-rgb-council and moot/v0.6.0-trading-floor both have identity
persistence, but only RGB documents it

**Where:** moot/v0.5.0 Scope — "six-layer agent identity model"; moot/v0.6.0 Scope

The RGB council spec defines a six-layer identity model (personality seed, worldview
state, journal, relationship map, track record, context window) and documents Mirror
integration under therapeutic isolation. The financial council spec lists agent personas
but says nothing about identity persistence or Mirror integration for those agents.

If the financial council agents (Meridian, financial_analyst, etc.) follow the same
identity model as RGB agents, that should be stated explicitly. If they don't persist
identity across sessions, that's also a valid choice — but it should be stated.
Silence here means each impl agent will make the decision independently.

**Recommended change:** moot/v0.6.0-trading-floor Scope should explicitly state
whether financial council agents use the six-layer identity model and whether Mirror
can interview them. If yes: reference the moot/v0.5.0 identity model as the shared
mechanism. If no: state "financial council agents are stateless — no identity
persistence, no Mirror integration" as a Non-Goal.

---

## Design Decisions That Need Cross-Repo Resolution

These are recorded as Key Unknowns in individual specs but cannot be answered
per-spec. They need a decision made once, then the answer propagated to all
affected specs before research stages begin.

### D1 — Knowledge context injection: push or pull?

**Specs affected:** jobman/v0.1.0 Key Unknown #1

If **push**: jobman assembles a ContextBrief and appends it to the deliberation
request before calling moot's API. Moot's deliberation input gains an optional
`context_brief` field. Moot does not need to know jobman exists.

If **pull**: moot calls jobman's REST API (`GET /knowledge/context?seeds=...`)
at deliberation start. Moot now has a runtime dependency on jobman being reachable.

**Recommended decision: push.** Moot should not know jobman exists — the dependency
arrow runs moot → jobman at the package level, and reversing it via a pull call
creates a runtime circular dependency (jobman schedules moot; moot calls jobman).
Push keeps moot ignorant of the coordination layer. Record this decision in
`jobman/v0.1.0` and close the Key Unknown before the design stage.

---

### D2 — Who owns the signals store?

**Specs affected:** jobman/v0.3.0, moot/v0.7.0

Both specs reference a "signals store" but neither names it as theirs. Jobman
v0.3.0 writes freshness metadata; moot/v0.7.0 reads Reddit summaries and
portfolio state. The store is written by jobman jobs and read by moot councils.

**Recommended decision:** Jobman owns the signals store (SQLite table in jobman's
data dir). Moot reads it via a file path configured in moot's env (`JOBMAN_SIGNALS_DB`).
This is the same pattern as moot reading its own DB — a known, stable path, no HTTP.
Jobman owns the schema and all writes; moot is a read-only consumer. State this
explicitly in both specs and close the ambiguity.

---

### D3 — Does breadwinner's planning pipeline use breadmin-forge for LLM skill execution?

**Specs affected:** breadwinner/v0.2.0, v0.3.0; jobman/v0.1.0 (forge context)

Breadwinner's planning pipeline makes LLM calls directly through its `llm/provider.py`
shim. Breadmin-forge is a generalized skill execution engine built for exactly this
pattern — declarative skill definitions drive LLM calls with output parsing.

The question is whether breadwinner should migrate its planning pipeline to forge skills
or keep its direct-call approach. The direct approach is more transparent and easier to
debug; forge skills add indirection but enable the knowledge package's `EmitConnection`
action to propagate financial insights into the knowledge graph automatically.

**Recommended decision:** Keep breadwinner's direct-call approach for now, but have
the planning pipeline emit `ConnectionSignal` objects to jobman's knowledge store after
each planning run. This gives the knowledge graph access to financial reasoning without
requiring breadwinner to adopt the full forge skill architecture. Add this as an impl
task in breadwinner/v0.4.0-crossing (it's a natural fit — crossing is the integration
milestone).

---

### D4 — Brimstone's credential proxy scope: product repos or brimstone subprocesses only?

**Specs affected:** brimstone/v0.2.0

Brimstone v0.2.0 adds a credential proxy so API keys never appear in subprocess
environments. The proxy serves brimstone's own agent subprocesses. But the product
repos (moot, breadwinner, jobman) also read API keys from their environments at runtime.
When brimstone dispatches an agent to implement a moot issue, that agent needs to run
`make test`, which needs `ANTHROPIC_API_KEY`, etc.

The spec doesn't address whether brimstone's credential proxy is meant to serve only
brimstone's own orchestrator processes, or also the product repos' test/lint runs
during impl stages.

**Recommended decision:** The credential proxy serves brimstone's agent subprocesses
only. Product repos read credentials from `.env` files (already gitignored, already
in `.env.example`). Brimstone's impl worker skill should explicitly instruct agents to
use the local `.env` for test runs, not expect credentials from the proxy. State this
scope boundary explicitly in the v0.2.0 spec.

---

## Recommended Spec Changes

| Finding | Spec to Update | Change |
|---------|---------------|--------|
| H1 | moot/v0.4.0-council-spine | Add "migrate moot scheduler and heartbeat to jobman job handlers" to Scope and Success Criteria |
| H2 | jobman/v0.3.0-signals-pipeline | Remove Key Unknown #2 (resolved: intermediate signals store); remove `_write_to_moot` from reddit package scope |
| H3 | moot/v0.4.0-council-spine | Add "remove moot budget/cost tracking" to Scope; reference breadmin-llm token_usage as source of truth |
| S1 | breadwinner/v0.2.0 | Note: rename `llm/` → `prompts/` in impl scope |
| S2 | All three v0.6.0/v0.2.0/v0.4.0 | Nominate `breadmin-jobman/docs/contracts/verdict-signal-v1.json` as canonical schema home; add to design stage deliverable |
| S3 | moot/v0.4.0-council-spine | Add "create `library_summaries` SQLite view as stable read contract"; jobman/v0.1.0: resolve Key Unknown #1 as "read via stable view" |
| S4 | moot/v0.6.0-trading-floor | Add explicit statement on financial agent identity persistence and Mirror eligibility |
| D1 | jobman/v0.1.0 | Resolve Key Unknown #1 as "push"; update design stage accordingly |
| D2 | jobman/v0.3.0, moot/v0.7.0 | State jobman owns signals store; moot reads via `JOBMAN_SIGNALS_DB` path |
| D3 | breadwinner/v0.4.0 | Add `EmitConnectionSignal` after planning runs as impl task |
| D4 | brimstone/v0.2.0 | Scope credential proxy to brimstone subprocesses only; document `.env` convention for product repos |
