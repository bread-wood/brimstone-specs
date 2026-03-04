# wowbot Campaign

## Summary

A fully automated World of Warcraft bot for WoW Midnight (retail), macOS only.
Progresses from a detection threat model through AH automation, open-world
gathering and farming, leveling, dungeon content, behavioral evasion, and
a capital compounding meta-layer.

Each version is independently demoable. Versions within a minor (0.x.0, 0.x.1)
share infrastructure; versions across minors require the previous minor to be
released before implementation begins. Parallel paths are noted explicitly.

---

## Spec Sequence

```
composer plan \
  specs/wowbot/v0.0.0-threat-model.md \
  specs/wowbot/v0.1.0-foundation.md \
  specs/wowbot/v0.1.1-ah-scanner.md \
  specs/wowbot/v0.2.1-ah-flipper.md \
  specs/wowbot/v0.3.0-navigator.md \
  specs/wowbot/v0.3.1-gatherer.md \
  specs/wowbot/v0.3.3-disenchanter.md \
  specs/wowbot/v0.3.4-work-orders.md \
  specs/wowbot/v0.4.0-combat-engine.md \
  specs/wowbot/v0.4.1-farmer.md \
  specs/wowbot/v0.4.2-multi-spec.md \
  specs/wowbot/v0.4.3-transmog-hunter.md \
  specs/wowbot/v0.5.0-leveler.md \
  specs/wowbot/v0.5.1-dungeon.md \
  specs/wowbot/v0.5.2-carry-coordinator.md \
  specs/wowbot/v0.6.0-sentinel.md \
  specs/wowbot/v0.6.1-scheduler.md \
  specs/wowbot/v0.7.2-patch-oracle.md \
  specs/wowbot/v0.7.3-compound.md
```

---

## Version Map

| Version | Name | Theme | Key addition | Depends on |
|---------|------|-------|-------------|------------|
| v0.0.0 | Threat Model | Detection surface audit | Research doc; no code | — |
| v0.1.0 | Foundation | Screen + input layer | `wowbot status` prints game state JSON | v0.0.0 findings |
| v0.1.1 | AH Scanner | AH listing extraction | `wowbot ah scan` → JSON | v0.1.0 |
| v0.2.1 | AH Flipper | Buy + relist cycle | `wowbot ah flip` transacts | v0.1.1 |
| v0.3.0 | Navigator | Waypoint movement | `wowbot move` follows routes | v0.1.0 |
| v0.3.1 | Gatherer | Node harvest + bank | `wowbot gather` loops | v0.3.0 |
| v0.3.3 | Disenchanter | DE arbitrage | `wowbot de-arb` buys gear, sells mats | v0.1.1, v0.2.1 |
| v0.3.4 | Contractor | Work order fulfillment | `wowbot contractor` fulfills commissions | v0.3.2 |
| v0.4.0 | Combat Engine | Targeting + rotation | `wowbot fight` | v0.3.0 |
| v0.4.1 | Farmer | Sustained farming loop | `wowbot farm` runs for hours | v0.4.0, v0.3.0 |
| v0.4.2 | Roster | Multi-spec support | Any spec via config file | v0.4.1 |
| v0.4.3 | Antiquarian | Transmog hunting | `wowbot antiquarian` clears old content | v0.4.0, v0.1.1 |
| v0.5.0 | Climber | XP leveling | `wowbot level` advances zones | v0.4.2, v0.3.0 |
| v0.5.1 | Instance | Dungeon clearing | `wowbot dungeon` with a group | v0.4.2, v0.3.0 |
| v0.5.2 | Fixer | Carry coordinator | `wowbot fixer` sells carry runs | v0.5.1, v0.1.0 |
| v0.6.0 | Sentinel | Behavioral self-monitoring | Session bounds + kill rate CV | v0.5.1 |
| v0.6.1 | Clockwork | Population-aware scheduler | `wowbot clockwork` switches modes | v0.6.0, v0.1.1 |
| v0.7.2 | Patch Oracle | Patch note trade signals | `wowbot oracle` positions inventory | v0.7.0 |
| v0.7.3 | Compound | Capital allocation | `wowbot compound` scales budgets | v0.7.0, v0.7.1, v0.6.1 |

---

## Impl Sequencing Gates

Research runs in parallel across all milestones from day one.
Implementation gates are enforced by `composer run`:

**Core chain:**
- v0.1.0 after: v0.0.0 released
- v0.1.1 after: v0.1.0 released

**AH path** (parallel with v0.3.x after v0.1.0):
- v0.2.1 after: v0.1.1 released
- v0.3.3 after: v0.2.1 released *(DE arb needs AH posting)*
- v0.3.4 after: v0.3.2 released *(contractor needs crafter)*

**Movement path** (parallel with v0.2.x after v0.1.0):
- v0.3.0 after: v0.1.0 released
- v0.3.1 after: v0.3.0 released

**Combat path:**
- v0.4.0 after: v0.3.0 released
- v0.4.1 after: v0.4.0 released
- v0.4.2 after: v0.4.1 released
- v0.4.3 after: v0.4.0 and v0.1.1 released *(transmog needs AH posting)*

**Content path** (v0.5.0 and v0.5.1 parallel after v0.4.2):
- v0.5.0 after: v0.4.2 released
- v0.5.1 after: v0.4.2 released
- v0.5.2 after: v0.5.1 released *(carry needs dungeon engine)*

**Evasion and intelligence layer:**
- v0.6.0 after: v0.5.1 released
- v0.6.1 after: v0.6.0 and v0.1.1 released

**Meta layer** (v0.7.2 and v0.7.3 parallel after v0.7.0 and v0.7.1):
- v0.7.2 after: v0.7.0 released *(oracle signals consumed by ledger)*
- v0.7.3 after: v0.7.0, v0.7.1, and v0.6.1 released

---

## Unspecced Dependencies

The following versions are referenced as dependencies but not yet specced.
They must be specced before the versions that depend on them enter design:

| Version | Referenced by | Notes |
|---------|---------------|-------|
| v0.2.2 | campaign notes | AH sniping; non-goal of v0.2.1 |
| v0.2.3 | campaign notes | Listing portfolio management |
| v0.3.2 | v0.3.4 | Crafter; take mats → craft → post |
| v0.7.0 | v0.7.2, v0.7.3 | Ledger; price history + P&L |
| v0.7.1 | v0.7.3 | Syndicate; multi-character orchestration |

---

## Notes

- **v0.2.0 gap**: merged into v0.2.1 (standalone market analysis too small for pipeline run).
- **v0.0.0**: Research-only. Its "release" is the accepted threat model document, not a binary.
- **v0.5.2 (Fixer)**: High-risk; requires a threat model addendum covering chat interaction
  before its design stage begins. This is a hard gate documented in the spec.
- **v0.7.x gap**: v0.7.0 and v0.7.1 are unspecced; they must be written and planned
  before v0.7.2 and v0.7.3 enter research.
