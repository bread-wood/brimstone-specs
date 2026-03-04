# Minesweeper Campaign

## Summary

A terminal Minesweeper implementation that grows from a playable game into a
fully-featured daily challenge platform with a constraint solver, no-guess
board generation, game replay, alternative topologies, and difficulty rating.
Each version is independently demoable. Serves as a second pipeline
proof-of-concept alongside the Calculator campaign.

---

## Spec Sequence

```
composer plan \
  specs/minesweeper/v0.1.x.md \
  specs/minesweeper/v0.2.x.md \
  specs/minesweeper/v0.3.x.md \
  specs/minesweeper/v0.4.x.md \
  specs/minesweeper/v0.5.x.md \
  specs/minesweeper/v0.6.x.md
```

---

## Version Map

| Version | Name | Theme | Key addition | Depends on |
|---------|------|-------|-------------|------------|
| v0.1.x | Foundation | Playable terminal game | `minesweeper` — full game loop | — |
| v0.2.x | Solver | Constraint-based hints + auto-solve | `h` for hint; `--auto-solve` | v0.1.x |
| v0.3.x | No-Guess | Boards requiring zero guesses | `--no-guess` flag | v0.2.x |
| v0.4.x | Records | Persistent stats + replay | `minesweeper stats` / `replay` | v0.1.x |
| v0.5.x | Variants | Hex and toroidal topologies | `--topology hex\|torus` | v0.1.x |
| v0.6.x | Challenge | Daily challenge + difficulty rating | `minesweeper daily` / `rate` | v0.2.x, v0.4.x |

---

## Impl Sequencing Gates

Research runs in parallel from day one. Implementation gates:

- v0.2.x after: v0.1.x released *(solver reads board module)*
- v0.3.x after: v0.2.x released *(no-guess generation depends on solver)*
- v0.4.x after: v0.1.x released *(parallel with v0.2.x — records only need game loop)*
- v0.5.x after: v0.1.x released *(parallel with v0.2.x and v0.4.x — topology is a board module concern)*
- v0.6.x after: v0.2.x and v0.4.x released *(difficulty rating needs solver; streak needs stats store)*

---

## Parallel Paths

After v0.1.x, three independent tracks can run concurrently:

- **Solver track**: v0.2.x → v0.3.x (sequential; no-guess needs solver)
- **Records track**: v0.4.x (needs only v0.1.x)
- **Variants track**: v0.5.x (needs only v0.1.x)

v0.6.x is the convergence point: it needs the solver (v0.2.x) for difficulty
rating and the stats store (v0.4.x) for streak tracking.

---

## Key Architectural Decisions Tracked Across Versions

Each version's Key Unknown resolves (or doesn't) a constraint established earlier:

| Unknown | Established in | Resolved in |
|---------|----------------|-------------|
| Board state representation | v0.1.x | v0.5.x (topology validates extensibility) |
| Terminal rendering approach | v0.1.x | v0.5.x (hex rendering validates flexibility) |
| Solver completeness | v0.2.x | v0.3.x (no-guess proves solver can determine all provable moves) |
| Topology abstraction | v0.1.x constraint | v0.5.x Key Unknown |
| Difficulty signal reliability | v0.6.x | validated empirically in v0.6.x testing |
