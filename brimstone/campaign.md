# brimstone Campaign

## Summary

The brimstone self-build campaign. brimstone extends itself version by version,
each release adding capabilities used to execute the next. By v0.4.0 the tool
can plan and sequence its own future versions as a campaign. Each version is
independently demoable and releasable.

---

## Spec Sequence

```bash
brimstone run \
  specs/brimstone/v0.2.0-hardened-core.md \
  specs/brimstone/v0.2.1-multi-model.md \
  specs/brimstone/v0.2.2-scale-out.md \
  specs/brimstone/v0.3.0-rolling-loop.md \
  specs/brimstone/v0.3.1-spec-update.md \
  specs/brimstone/v0.4.0-long-game.md \
  specs/brimstone/v0.5.0-spec-forge.md \
  --repo bread-wood/brimstone
```

Note: v0.1.0 is SHIPPED. The campaign begins at v0.2.0.

---

## Version Map

| Version | Name | Status | Key addition | Depends on |
|---------|------|--------|-------------|------------|
| v0.1.0 | Beads Foundation | **SHIPPED** | BeadStore, WorkBead, PRBead, MergeQueue, Watchdog, agent PR ownership, checkpoint v3 | — |
| v0.2.0 | Hardened Core | PLANNED | Credential proxy; CI/GHA deployment; improved crash recovery | — |
| v0.2.1 | Multi-Model | PLANNED | Backend abstraction; complexity estimator; Telegram | v0.2.0 |
| v0.2.2 | Scale Out | PLANNED | `brimstone-beads` + `brimstone-specs` repo creation; spec migration from local; dedicated bead write queue; platform repo registry; `brimstone run --campaign`; spec write routing; bead inter-repo milestone deps; multi-repo orchestration; cross-model consensus | v0.2.1 |
| v0.3.0 | Rolling Loop | PLANNED | Rolling dispatch; `brimstone status`; qa/release cmds | v0.2.2 |
| v0.3.1 | Spec Update | PLANNED | `brimstone run --spec-update` | v0.3.0 |
| v0.4.0 | Long Game | PLANNED | Campaign mode; impl sequencing gate; multi-milestone spec-update | v0.3.1 |
| v0.5.0 | Spec Forge | PLANNED | Interactive spec design mode; structured interview; TEMPLATE.md validation; architecture guard; campaign update | v0.4.0 |

### What shipped in v0.1.0 (already out of scope for v0.2.0+)

These were originally scoped to v0.2.0 or v0.3.0 but landed in v0.1.0:
- **BeadStore + bead types** — WorkBead, PRBead, MergeQueue (was v0.2.0 "crash recovery")
- **Watchdog** (`_watchdog_scan`) — zombie agent detection and recovery (was v0.3.0 "agent watchdog")
- **Agent PR ownership** — agents handle CI (max 3) and review (max 2) themselves
- **MergeQueue** — sequential squash merges via `_process_merge_queue()`
- **Checkpoint schema v3** — slim run metadata + backoff only

v0.2.0 scope should focus on: **credential proxy** (API key never in subprocess env),
**GHA deployment** (trigger on issues event, no personal session required), and
**improved crash recovery** (beyond what BeadStore already provides).

---

## Impl Sequencing Gates

- **v0.2.1 after v0.2.0**: credential proxy must be running before non-Claude backends
  are dispatched
- **v0.2.2 after v0.2.1**: multi-model backends must exist before cross-model consensus
  can run
- **v0.3.0 after v0.2.2**: rolling dispatch preserves all v0.2.x security layers
- **v0.3.1 after v0.3.0**: spec-update runs after a release; requires `brimstone run
  --release` from v0.3.0
- **v0.4.0 after v0.3.1**: multi-milestone spec-update extends v0.3.1; impl sequencing
  gate depends on v0.3.0's rolling dispatch loop
- **v0.5.0 after v0.4.0**: spec-forge updates campaigns; campaign mode must exist before
  spec-forge can write campaign updates
- **v0.5.0 also requires v0.2.2**: spec write routing and repo registry must be in place
  before spec-forge can resolve where to write specs for each repo

---

## Parallel Paths

This campaign is fully sequential — each version is a direct prerequisite for the next.
No parallel implementation tracks.

Research for all milestones runs concurrently from day one; only `stage/impl` is gated.

---

## Key Architectural Decisions Tracked Across Versions

| Unknown | Established in |
|---------|----------------|
| Credential proxy intercept mechanism | v0.2.0 |
| Checkpoint schema for multi-repo extensibility | v0.2.0 |
| Backend interface abstraction for new backends | v0.2.1 |
| Complexity estimator signal generalisability | v0.2.1 |
| Cross-model consensus diff strategy | v0.2.2 |
| Platform repo registry format and location | v0.2.2 |
| Bead inter-repo milestone dep blocking mechanism | v0.2.2 |
| Bead write queue primitive (separate from MergeQueue) | v0.2.2 |
| Minimum context for spec elaboration | v0.3.1 |
| Impl sequencing gate signal (release tag vs open issues) | v0.4.0 |
| Multi-milestone spec-update commit strategy | v0.4.0 |
| Spec-forge intake format (free-form vs structured vs multi-turn) | v0.5.0 |
| Wave position insertion strategy for ambiguous dependency ordering | v0.5.0 |
