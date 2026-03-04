# Platform Campaign

## Repos

- brimstone (bread-wood/brimstone)
- breadmin-jobman (bread-wood/breadmin-jobman)
- moot (bread-wood/moot)
- breadwinner (bread-wood/breadwinner)
- brimstone-beads (bread-wood/brimstone-beads) — created in brimstone/v0.2.2
- brimstone-specs (bread-wood/brimstone-specs) — created in brimstone/v0.2.2; migration of ~/Documents/specs/ happens here
- pantry (bread-wood/pantry) — not yet created

---

## Waves

### Wave 0 — Foundations _(no blocking deps)_

- `brimstone/v0.2.0-hardened-core`
- `moot/v0.4.0-council-spine`
- `breadwinner/v0.2.0-option-forge`

### Wave 1 — Parallel after Wave 0

- `brimstone/v0.2.1-multi-model`
- `moot/v0.5.0-rgb-council`
- `breadwinner/v0.3.0-iron-harvest`
- `breadmin-jobman/v0.1.0-knowledge-wire`

### Wave 2 — Parallel after Wave 1

- `brimstone/v0.2.2-scale-out`
- `moot/v0.6.0-trading-floor` _(verdict contract research — see Key Coordination Points)_
- `breadmin-jobman/v0.2.0-verdict-bridge` _(verdict contract research)_
- `breadwinner/v0.4.0-crossing` _(verdict contract research)_
- `pantry/v0.1.0-meal-planner` _(independent; no cross-repo deps)_

### Wave 3 — impl stages of coordination points

- `brimstone/v0.3.0-rolling-loop`
- `moot/v0.6.0-trading-floor` _(impl, after verdict contract research resolves)_
- `breadmin-jobman/v0.2.0-verdict-bridge` _(impl, after moot/v0.6.0)_
- `breadwinner/v0.4.0-crossing` _(impl, after jobman/v0.2.0)_
- `pantry/v0.2.0-shopping-cart` _(after pantry/v0.1.0)_

### Wave 4 — signals + spec tooling

- `brimstone/v0.3.1-spec-update`
- `breadmin-jobman/v0.3.0-signals-pipeline`
- `moot/v0.7.0-open-field`

### Wave 5 — campaign mode + spec forge

- `brimstone/v0.4.0-long-game`
- `brimstone/v0.5.0-spec-forge` _(after v0.4.0)_

---

## Key Coordination Points

**Verdict contract research (Wave 2):**
moot/v0.6.0, breadmin-jobman/v0.2.0, and breadwinner/v0.4.0 share a single Key Unknown: the verdict contract schema that all three repos will exchange. Research for all three must complete and converge on a shared contract file before impl begins on any of them. The canonical output artifact is a shared contract spec at a stable path (TBD by research agents). None of the three impl stages may begin until the contract is resolved.

---

## Parallel Paths

**pantry** runs entirely independently of the product critical path:
- No dependency on moot, breadmin-jobman, or breadwinner
- No connection to jobman signals store, knowledge graph, or moot councils
- pantry/v0.1.0 can begin as soon as Wave 2 opens; pantry/v0.2.0 follows immediately after v0.1.0
- Blocked only by P1 Key Unknown resolution (grocery delivery API access — Instacart Connect or alternative)

**brimstone** is a parallel build track — it is infrastructure for the platform, not product features:
- brimstone versions do not block product milestones (moot, jobman, breadwinner, pantry)
- brimstone benefits from completing earlier so it can be used to run later waves
- spec-forge (v0.5.0) is the meta-layer: it automates the planning work done manually to produce this file

---

## Critical Path

The minimum sequence that must complete before the platform's core coordination capability (verdict-bridged trading loop) is live:

```
breadwinner/v0.2.0-option-forge
  → breadwinner/v0.3.0-iron-harvest
    → [verdict contract research: moot/v0.6.0 + jobman/v0.2.0 + breadwinner/v0.4.0]
      → moot/v0.6.0-trading-floor (impl)
        → breadmin-jobman/v0.2.0-verdict-bridge (impl)
          → breadwinner/v0.4.0-crossing (impl)
```

moot/v0.4.0 and moot/v0.5.0 also sit on the critical path as prerequisites for moot/v0.6.0.

---

## brimstone run command (full platform)

```bash
brimstone run \
  --campaign specs/CAMPAIGN.md \
  --wave 0
```
