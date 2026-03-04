# Platform Milestone Priority List

Ordered by wave (start order), then by critical path position within each wave.
`[CP]` = on the critical path to the verdict-bridged trading loop.
`[INF]` = brimstone infrastructure (parallel track, not product critical path).
`[IND]` = independent (no cross-repo deps).

---

## Wave 0 — Start immediately

| # | Milestone | Track | Blocks |
|---|-----------|-------|--------|
| 1 | `breadwinner/v0.2.0-option-forge` | [CP] | breadwinner/v0.3.0 |
| 2 | `moot/v0.4.0-council-spine` | [CP] | moot/v0.5.0 |
| 3 | `brimstone/v0.2.0-hardened-core` | [INF] | brimstone/v0.2.1 |

---

## Wave 1 — After Wave 0 completes

| # | Milestone | Track | Blocks |
|---|-----------|-------|--------|
| 4 | `moot/v0.5.0-rgb-council` | [CP] | moot/v0.6.0 |
| 5 | `breadwinner/v0.3.0-iron-harvest` | [CP] | breadwinner/v0.4.0 |
| 6 | `breadmin-jobman/v0.1.0-knowledge-wire` | [CP] | jobman/v0.2.0 |
| 7 | `brimstone/v0.2.1-multi-model` | [INF] | brimstone/v0.2.2 |

---

## Wave 2 — After Wave 1 completes

| # | Milestone | Track | Blocks |
|---|-----------|-------|--------|
| 8 | `moot/v0.6.0-trading-floor` _(research)_ | [CP] | moot/v0.6.0 impl |
| 9 | `breadmin-jobman/v0.2.0-verdict-bridge` _(research)_ | [CP] | jobman/v0.2.0 impl |
| 10 | `breadwinner/v0.4.0-crossing` _(research)_ | [CP] | breadwinner/v0.4.0 impl |
| 11 | `brimstone/v0.2.2-scale-out` | [INF] | brimstone/v0.3.0 |
| 12 | `pantry/v0.1.0-meal-planner` | [IND] | pantry/v0.2.0 |

> **Note:** Items 8–10 are a coordinated research block. None of the three impl stages
> may begin until all three research stages converge on a shared verdict contract schema.

---

## Wave 3 — After verdict contract resolves + Wave 2 completes

| # | Milestone | Track | Blocks |
|---|-----------|-------|--------|
| 13 | `moot/v0.6.0-trading-floor` _(impl)_ | [CP] | jobman/v0.2.0 impl |
| 14 | `breadmin-jobman/v0.2.0-verdict-bridge` _(impl)_ | [CP] | breadwinner/v0.4.0 impl |
| 15 | `breadwinner/v0.4.0-crossing` _(impl)_ | [CP] | — (critical path complete) |
| 16 | `brimstone/v0.3.0-rolling-loop` | [INF] | brimstone/v0.3.1 |
| 17 | `pantry/v0.2.0-shopping-cart` | [IND] | — |

---

## Wave 4 — After Wave 3 completes

| # | Milestone | Track | Blocks |
|---|-----------|-------|--------|
| 18 | `breadmin-jobman/v0.3.0-signals-pipeline` | product | — |
| 19 | `moot/v0.7.0-open-field` | product | — |
| 20 | `brimstone/v0.3.1-spec-update` | [INF] | brimstone/v0.4.0 |

---

## Wave 5 — After Wave 4 completes

| # | Milestone | Track | Blocks |
|---|-----------|-------|--------|
| 21 | `brimstone/v0.4.0-long-game` | [INF] | brimstone/v0.5.0 |
| 22 | `brimstone/v0.5.0-spec-forge` | [INF] | — |

---

## Critical Path (minimum sequence to trading loop)

```
moot/v0.4.0
  → moot/v0.5.0
    ↘
breadwinner/v0.2.0                    [verdict contract research]
  → breadwinner/v0.3.0  →  moot/v0.6.0-research  ─┐
                         →  jobman/v0.2.0-research  ├→ contract resolved
breadmin-jobman/v0.1.0               →  bw/v0.4.0-research  ─┘
  ↗                                        ↓
                                  moot/v0.6.0-impl
                                    → jobman/v0.2.0-impl
                                      → breadwinner/v0.4.0-impl  ✓
```

**Total waves on critical path: 0 → 1 → 2 (research) → 3 (impl)**

---

## Parallel Tracks Summary

| Track | Milestones | Dependency on critical path |
|-------|------------|----------------------------|
| pantry | v0.1.0 → v0.2.0 | None — starts Wave 2, runs independently |
| brimstone infra | v0.2.0 → v0.2.1 → v0.2.2 → v0.3.0 → v0.3.1 → v0.4.0 → v0.5.0 | None — parallel self-build throughout |
