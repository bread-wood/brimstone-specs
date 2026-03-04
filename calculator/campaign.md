# Calculator CLI Campaign

## Summary

A command-line calculator that grows from four-operator arithmetic into a
fully-featured graphing calculator with symbolic computation. Each version
is independently useful and demoable. The campaign validates the
brimstone pipeline end-to-end across 25 milestones.

---

## Spec Sequence

v0.1.0–v0.4.0 are **SHIPPED**. v0.5.0 is **IN PROGRESS** (being rebuilt from scratch
with new brimstone). The campaign continues from v0.5.0.

```bash
brimstone run \
  specs/calculator/v0.5.0.md \
  specs/calculator/v0.6.0.md \
  specs/calculator/v0.7.0.md \
  specs/calculator/v0.8.0.md \
  specs/calculator/v0.9.0.md \
  specs/calculator/v1.0.0.md \
  specs/calculator/v1.1.0.md \
  specs/calculator/v1.2.0.md \
  specs/calculator/v1.3.0.md \
  specs/calculator/v1.4.0.md \
  specs/calculator/v1.5.0.md \
  specs/calculator/v1.6.0.md \
  specs/calculator/v1.7.0.md \
  specs/calculator/v1.8.0.md \
  specs/calculator/v1.9.0.md \
  specs/calculator/v1.10.0.md \
  specs/calculator/v1.11.0.md \
  specs/calculator/v1.12.0.md \
  specs/calculator/v2.0.0.md \
  specs/calculator/v2.1.0.md \
  specs/calculator/v2.2.0.md \
  --repo bread-wood/calculator
```

---

## Version Map

| Version | Name | Status | Key addition | Depends on |
|---------|------|--------|-------------|------------|
| v0.1.0 | Cold Start | **SHIPPED** | `calc '2+3'` | — |
| v0.2.0 | Function Library | **SHIPPED** | `sqrt`, `sin`, `pi` | v0.1.0 |
| v0.3.0 | Variables | **SHIPPED** | `x = 5; x * 2` | v0.2.0 |
| v0.4.0 | User-Defined Functions | **SHIPPED** | `def f(x) = ...` | v0.3.0 |
| v0.5.0 | Renderer | IN PROGRESS | `calc plot 'sin(x)'` → PNG | v0.4.0 |
| v0.6.0 | Multi-Curve | Multiple curves + parametric | Multiple expressions on one plot | v0.5.0 |
| v0.7.0 | Interactive | Live window, pan/zoom | `--interactive` | v0.6.0 |
| v0.8.0 | Editor | In-window expression entry | Type to add curves | v0.7.0 |
| v0.9.0 | Implicit | Implicit curves + inequalities | `x^2+y^2=25` | v0.8.0 |
| v1.0.0 | Session | Save/load + REPL | Persistent sessions | v0.9.0 |
| v1.1.0 | Conditionals | `if/then/else`, comparisons | Piecewise functions | v1.0.0 |
| v1.2.0 | Summation | `sum`, `product`, `integral` | Math iteration notation | v1.1.0 |
| v1.3.0 | Complex Numbers | `i`, complex arithmetic | `sqrt(-1)` = `i` | v1.2.0 |
| v1.4.0 | Bitwise | `0xFF`, `&\|^~<<>>` | Developer arithmetic | v1.1.0 |
| v1.5.0 | Lists | `[1..10]`, `map`, `filter` | Collections + statistics | v1.2.0 |
| v1.6.0 | Sliders | Draggable parameters | `a*sin(x)` with slider for `a` | v1.0.0 |
| v1.7.0 | 3D | Surface plots | `calc plot3d 'sin(x)*cos(y)'` | v0.7.0 |
| v1.8.0 | Vector Fields | Arrow plots, streamlines | `calc plotvf '[y,-x]'` | v1.5.0, v0.7.0 |
| v1.9.0 | Data | CSV import, scatter, regression | `--data data.csv` | v0.6.0 |
| v1.10.0 | Pipeline | Pipe mode, script files, watch | `echo expr \| calc` | v1.0.0 |
| v1.11.0 | LaTeX | Expression → LaTeX string | `calc latex 'x^2/4'` | v1.2.0 |
| v1.12.0 | Units | Dimensional arithmetic | `5 km + 3 mi` | v0.1.0 |
| v2.0.0 | Symbolic Diff | `diff` as expression | `diff('x^3', x)` → `3*x^2` | v1.0.0 |
| v2.1.0 | Solver | Equation solving | `solve('x^2=4', x)` | v2.0.0, v1.3.0 |
| v2.2.0 | Simplify | Algebraic simplification | `sin^2+cos^2` → `1` | v2.0.0 |

---

## Impl Sequencing Gates

Research runs in parallel from day one. Implementation gates:

**Core language chain (sequential):**
- v0.1.0 → v0.2.0 → v0.3.0 → v0.4.0

**Graphing chain (after v0.4.0):**
- v0.5.0 → v0.6.0 → v0.7.0 → v0.8.0 → v0.9.0 → v1.0.0

**Language extensions (after v1.0.0, mostly parallel):**
- v1.1.0 after v1.0.0
- v1.2.0 after v1.1.0
- v1.3.0 after v1.2.0
- v1.4.0 after v1.1.0 *(parallel with v1.2.0)*
- v1.5.0 after v1.2.0 *(parallel with v1.3.0)*

**Graphing extensions (parallel paths after their prerequisites):**
- v1.6.0 after v1.0.0 *(parallel with language extensions)*
- v1.7.0 after v0.7.0 *(parallel with v1.0.0+)*
- v1.8.0 after v1.5.0 and v0.7.0
- v1.9.0 after v0.6.0 *(parallel with v0.7.0+)*

**CLI extensions:**
- v1.10.0 after v1.0.0
- v1.11.0 after v1.2.0
- v1.12.0 after v0.1.0 *(can begin anytime; isolated type system change)*

**Symbolic layer (sequential):**
- v2.0.0 after v1.0.0
- v2.1.0 after v2.0.0 and v1.3.0
- v2.2.0 after v2.0.0 *(parallel with v2.1.0)*

---

## Parallel Paths

Several versions are independent after their prerequisites and can run
implementation concurrently:

- **v1.4.0** (Bitwise) and **v1.2.0** (Summation) — both after v1.1.0
- **v1.6.0** (Sliders) — can run alongside any v1.x language extension
- **v1.7.0** (3D) — can run alongside v1.0.0+ work
- **v1.9.0** (Data) — can begin after v0.6.0, well before v1.0.0
- **v1.12.0** (Units) — most isolated; depends only on v0.1.0 evaluator
- **v2.1.0** and **v2.2.0** — both after v2.0.0, independent of each other
