# Milestone Dependency Graph

**Repos:** brimstone · jobman · moot · breadwinner
**Date:** 2026-03-04

---

## Full DAG

Arrows mean "must complete before". Milestones on the same horizontal level can
run in parallel.

```
brimstone                 jobman                  moot                    breadwinner
─────────────────────────────────────────────────────────────────────────────────────

v0.1.x ✓
  │
v0.2.0-hardened-core                          v0.4.0-council-spine    v0.2.0-option-forge
  │                                                 │       │                  │
v0.2.1-multi-model        v0.1.0-knowledge-wire ◄──┘       │           v0.3.0-iron-harvest
  │                                                         │                  │
v0.2.2-scale-out                                            │           ┌──────┘
  │                                                         │           │
v0.3.0-rolling-loop                                         └──────────►│
  │                         ┌──── moot/v0.6.0 ◄─────────────────────────┘
v0.3.1-spec-update          │     (contract)
  │                         │         │
v0.4.0-long-game    v0.2.0-verdict-bridge ◄──── breadwinner/v0.3.0    v0.6.0-trading-floor
                            │                                               │
                    v0.3.0-signals-pipeline          v0.5.0-rgb-council     │  (parallel)
                                │                       (parallel)          │
                                └──────────────────► v0.7.0-open-field      │
                                                                    breadwinner/v0.4.0-crossing
```

---

## Dependency Table

Each dependency includes the reason it's a hard blocker, not just the name.

| Milestone | Depends On | Why |
|-----------|-----------|-----|
| **brimstone** | | |
| v0.1.x-cold-start | ✓ done | — |
| v0.2.0-hardened-core | brimstone/v0.1.x | Adds crash recovery and credential proxy on top of the existing pipeline runner. Without v0.1.x there is no pipeline to harden. |
| v0.2.1-multi-model | brimstone/v0.2.0 | Credential proxy must be running before non-Claude backends are dispatched — API keys for Gemini/GPT can't appear in subprocess environments. Crash recovery must exist before multi-backend parallel sessions run safely. |
| v0.2.2-scale-out | brimstone/v0.2.1 | Multi-model backends must be available before cross-model consensus can run. Repo-scoped checkpoints from v0.2.0 must exist before multi-repo sessions are safe. |
| v0.3.0-rolling-loop | brimstone/v0.2.0, v0.2.2 | Needs crash-safe checkpoints (v0.2.0) for the watchdog to safely re-queue hung agents. Needs multi-repo support (v0.2.2) since rolling dispatch becomes most valuable across many repos simultaneously. |
| v0.3.1-spec-update | brimstone/v0.3.0 | The `--spec-update` stage propagates findings from merged PRs and research docs into future specs. Requires `--qa` and `--release` stages from v0.3.0 to exist before there are completed releases to propagate from. |
| v0.4.0-long-game | brimstone/v0.3.1 | Campaign mode sequences multiple versions. Spec-update (v0.3.1) is how findings flow forward between versions — without it, campaign mode would sequence milestones but leave Key Unknowns stale across them. |
| **moot** | | |
| v0.4.0-council-spine | — | No deps. Cleanup and CouncilPlugin protocol are structural prerequisites that everything else builds on. |
| v0.5.0-rgb-council | moot/v0.4.0 | RGB council must be implemented as a CouncilPlugin from day one. The protocol it implements doesn't exist until v0.4.0. |
| v0.6.0-trading-floor | moot/v0.4.0 | Financial council must also implement CouncilPlugin. Same reason as v0.5.0. |
| v0.6.0-trading-floor | breadwinner/v0.3.0 | The financial council's Key Unknown #2 (deep-pass trigger criteria) requires knowing what a full options position looks like — strike, delta, DTE, P&L. That data isn't available until iron-harvest produces the IV infrastructure and options lifecycle rules. |
| v0.6.0-trading-floor | jobman/v0.2.0 ¹ | Shared Key Unknown: the verdict handler interface must be defined jointly with jobman before moot can implement its side. Research stages coordinate; moot impl follows. |
| v0.7.0-open-field | moot/v0.6.0 | Open Field adds data to the financial council's deliberation context (Reddit signals, options/crypto state). The financial council must exist before its context can be extended. |
| v0.7.0-open-field | jobman/v0.3.0 | The signals pipeline job (Reddit sync, portfolio refresh) writes to the signals store that moot/v0.7.0 reads. The store schema and write logic are jobman's concern. |
| **breadwinner** | | |
| v0.2.0-option-forge | — | No deps. Builds on the existing equity engine. |
| v0.3.0-iron-harvest | breadwinner/v0.2.0 | Backtester replays the plan engine over historical dates. The plan engine must include options phases (2/4/5) from v0.2.0 or the replay produces equity-only results. IV infrastructure uses the options scanner and position models from v0.2.0. |
| v0.4.0-crossing | breadwinner/v0.3.0 | The `bw snapshot --format=jobman` output must include IV rank and a full options position snapshot with live Greeks. IV rank doesn't exist until iron-harvest, and the options state with Greeks comes from v0.2.0 but the IV rank enrichment requires v0.3.0. |
| v0.4.0-crossing | moot/v0.6.0 ¹ | Breadwinner's verdict handler can only be built once the verdict signal schema is defined. That schema is a shared output of the moot/jobman/breadwinner research coordination. |
| v0.4.0-crossing | jobman/v0.2.0 ¹ | The `breadwinner-council-signal` handler receives signals from jobman's verdict bridge. The signal schema and transport must be defined before breadwinner can implement the receiver. |
| **jobman** | | |
| v0.1.0-knowledge-wire | moot/v0.4.0 | The knowledge bootstrap reads moot's `library_*` tables and `library_changelog`. These tables are stable after council-spine's cleanup and procedures merge. Starting before v0.4.0 risks bootstrapping against a schema that is about to change. |
| v0.2.0-verdict-bridge | moot/v0.6.0 ¹ | The verdict bridge polls moot for pending financial council verdicts. The verdict queue (REST endpoint or DB table) doesn't exist until moot v0.6.0 implements it. |
| v0.2.0-verdict-bridge | breadwinner/v0.3.0 | The `breadwinner-state-snapshot` job calls breadwinner's CLI to extract options and crypto positions. The full snapshot (with IV rank and options Greeks) requires iron-harvest. Before that, the snapshot is equity-only and financially incomplete for council deliberation. |
| v0.3.0-signals-pipeline | jobman/v0.2.0 | The portfolio refresh job writes to the portfolio state store. The store schema (versioned, with `schema_version` field) is defined as part of v0.2.0. The signals pipeline depends on that schema being stable before it adds a lighter-weight refresh cadence on top. |

¹ **Coordination point** — see below.

---

## Build Waves

Groups of milestones that can be dispatched in parallel, in order.

### Wave 0 — Foundations
*No blocking deps. Start immediately.*

| Milestone | Repo | Why it's first |
|-----------|------|----------------|
| v0.2.0-hardened-core | brimstone | Credential proxy and crash recovery are prerequisites for safely dispatching any Wave 1+ agents. Without them, agents that die mid-run leave stale labels and orphaned branches. |
| v0.4.0-council-spine | moot | Defines the CouncilPlugin protocol and stabilizes the DB schema. Everything in moot depends on this. Also unblocks jobman/v0.1.0, which reads the stabilized library tables. |
| v0.2.0-option-forge | breadwinner | Options positions and the planning pipeline extensions are required by every downstream breadwinner milestone and by the financial council's deliberation context. |

### Wave 1 — Unblocked after Wave 0
*Run in parallel.*

| Milestone | Repo | What it needs from Wave 0 |
|-----------|------|--------------------------|
| v0.2.1-multi-model | brimstone | Credential proxy from v0.2.0 |
| v0.5.0-rgb-council | moot | CouncilPlugin protocol from v0.4.0 |
| v0.3.0-iron-harvest | breadwinner | Options data models and planning pipeline from v0.2.0 |
| v0.1.0-knowledge-wire | jobman | Stable moot library DB schema from v0.4.0 |

### Wave 2 — Unblocked after Wave 1
*Run in parallel. Includes the critical cross-repo research coordination.*

| Milestone | Repo | What it needs from Wave 1 |
|-----------|------|--------------------------|
| v0.2.2-scale-out | brimstone | Multi-model backends from v0.2.1 (needed for cross-model consensus) |
| **Verdict contract research** | moot + jobman + breadwinner | breadwinner/v0.3.0 must be done so the full snapshot schema (including IV rank and options Greeks) is known before the contract is designed — otherwise the contract undersells the data that breadwinner can actually deliver |

> **How to run the verdict contract research:** Open research issues simultaneously in
> moot/v0.6.0, jobman/v0.2.0, and breadwinner/v0.4.0. The three research agents
> coordinate to produce one shared document defining the verdict signal schema. All
> three design stages consume it. Impl stages then sequence (see Wave 3).

### Wave 3 — Unblocked after Wave 2
*Impl stages of the coordination point. brimstone rolling loop also lands here.*

| Milestone | Repo | What it needs from Wave 2 |
|-----------|------|--------------------------|
| v0.3.0-rolling-loop | brimstone | Crash-safe checkpoints (v0.2.0) and multi-repo support (v0.2.2). Rolling loop becomes valuable only once there's enough concurrent work across repos to saturate it — Wave 3 is that moment. |
| v0.6.0-trading-floor | moot | Verdict contract defined; CouncilPlugin from v0.4.0; full options context from breadwinner/v0.3.0 |
| v0.2.0-verdict-bridge | jobman | moot/v0.6.0 verdict interface implemented; breadwinner/v0.3.0 snapshot available |
| v0.4.0-crossing | breadwinner | Verdict signal schema defined; jobman/v0.2.0 and moot/v0.6.0 both implemented |

> Within Wave 3, the product milestones sequence: **moot/v0.6.0 impl → jobman/v0.2.0
> impl → breadwinner/v0.4.0 impl**. They cannot all start at the same time because each
> builds on the previous repo's output. brimstone/v0.3.0 runs in parallel with all of them.

### Wave 4 — Unblocked after Wave 3

| Milestone | Repo | What it needs from Wave 3 |
|-----------|------|--------------------------|
| v0.3.1-spec-update | brimstone | Completed releases from v0.3.0 to propagate findings from |
| v0.3.0-signals-pipeline | jobman | Portfolio state store schema from v0.2.0; something to schedule (Reddit was already written in the package, just unwired) |
| v0.7.0-open-field | moot | Financial council from v0.6.0 to inject context into; signals pipeline from jobman/v0.3.0 to read from |

### Wave 5 — Unblocked after Wave 4

| Milestone | Repo | What it needs from Wave 4 |
|-----------|------|--------------------------|
| v0.4.0-long-game | brimstone | Spec-update stage from v0.3.1 — long game's campaign mode propagates findings across planned versions, which requires spec-update to already work |

---

## Critical Path

The longest chain of sequential dependencies. Everything else can run alongside it.

```
breadwinner/v0.2.0-option-forge          builds options positions + planning pipeline
  → breadwinner/v0.3.0-iron-harvest      adds IV rank + full options snapshot capability
  → [verdict contract research]          defines the shared signal schema (all three repos)
    → moot/v0.6.0-trading-floor          implements financial council + verdict storage
      → jobman/v0.2.0-verdict-bridge     wires verdict polling + portfolio state delivery
        → breadwinner/v0.4.0-crossing    implements signal receiver + pending guard
          → jobman/v0.3.0-signals-pipeline  adds scheduled Reddit + portfolio refresh
            → moot/v0.7.0-open-field     injects signals into financial deliberations
```

**brimstone is not on the product critical path.** It runs alongside. The only
product-blocking brimstone gate is v0.2.0 (safe dispatch) before Wave 1, and v0.2.2
(multi-repo) before Wave 3 if you want a single brimstone session to orchestrate the
cross-repo coordination rather than running each repo's pipeline separately.

---

## Key Coordination Points

### 1. Verdict Contract (Wave 2 → Wave 3)

**What it is:** moot/v0.6.0, jobman/v0.2.0, and breadwinner/v0.4.0 all share one Key
Unknown — what does a financial council verdict contain, and how does it travel from
moot to breadwinner via jobman?

**Why it's a coordination point and not just a dependency:** Each repo's spec names it
as *their* Key Unknown #1, but it's actually one question with one answer. If researched
in isolation, the three repos will produce incompatible schemas. The research stages must
produce a single shared document.

**How to resolve it:** Run three research agents simultaneously (one per repo), with an
explicit instruction to produce a shared `docs/research/verdict-contract.md` that all
three design stages reference. The schema must be stable before any of the three begins
its design stage.

**What unblocks after:** Once the contract is defined, the impl stages sequence:
moot builds the verdict storage and exposure → jobman builds the poller and delivery
→ breadwinner builds the signal receiver. Each repo's impl can begin as soon as the
previous one merges.

### 2. moot DB stability for jobman/knowledge-wire

**What it is:** `jobman/v0.1.0-knowledge-wire` reads moot's `library_*` tables and
`library_changelog` directly via SQL. If those tables change shape after the knowledge
bootstrap runs, the watcher breaks.

**Why moot/v0.4.0 is the right gate:** Council-spine removes stale stubs, merges
`procedures/` into `councils/`, and stabilizes the package structure. After that, the
library tables are not expected to change shape again. Starting knowledge-wire before
v0.4.0 lands means the bootstrap may run against a schema that disappears in the cleanup.

**What it does not need:** knowledge-wire does not need v0.5.0 (rgb-council adds no new
library tables) or v0.6.0 (financial council adds `FinancialLibrary`, but knowledge-wire's
bootstrap is idempotent and will discover it when it arrives).

### 3. brimstone multi-repo gate

**What it is:** Wave 3 requires coordinating moot, jobman, and breadwinner
simultaneously. brimstone/v0.2.2-scale-out is what makes that possible from a single
orchestrator session.

**Why it matters:** Without v0.2.2, each repo's pipeline must be started as a separate
brimstone session. That works, but the Wave 3 sequencing (moot impl → jobman impl →
breadwinner impl) requires the orchestrator to watch for PR merges across repos and
trigger the next repo's impl stage automatically. That cross-repo watching is v0.2.2
capability.

**What happens if v0.2.2 isn't ready:** Wave 3 still executes, but manually — the
human operator triggers each repo's impl stage after the previous one merges. The work
is the same; the automation is missing.
