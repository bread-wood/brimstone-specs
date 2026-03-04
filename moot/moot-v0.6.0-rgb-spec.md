# Moot v0.6.0-rgb — Red/Blue/Green Deliberation Council

## Overview

Moot is a three-agent deliberation council that debates decisions involving change — whether to pursue it, how much, how fast, and what to preserve. Three agents (Blue for ambition, Green for cultivation, Red for limits) engage in structured multi-round dialogue, facilitated by a neutral process tool (Grey) that detects convergence or divergence and produces the final output. Moot is one of three councils in a shared plugin architecture alongside Prism and Hoard, and exposes the same interface contract. Built for a user who needs externalized, structured thinking about life direction, habit evaluation, project decisions, and growth planning.

---

## Success Criteria

- [ ] Given a pre-framed topic, Blue's Round 1 output is oriented toward possibility, vision, or forward motion — not assessment of current state or risk
- [ ] Given a pre-framed topic, Green's Round 1 output assesses the current landscape — what is working, what is dying, what has room to grow — not vision or risk
- [ ] Given a pre-framed topic, Red's Round 1 output identifies risks, constraints, and challenges to proposed change — not vision or cultivation
- [ ] Each agent's Round 2 response references specific claims from both other agents' Round 1 positions (not generic responses)
- [ ] Each agent's Round 2 response either defends, adapts, or concedes specific points — not a restatement of Round 1
- [ ] Grey terminates deliberation after at most 3 rounds
- [ ] Grey produces a convergence synthesis when agents' final positions are compatible, naming which agent's concerns shaped the synthesis and any residual caveats
- [ ] Grey produces a divergence report when agents' positions are irreconcilable, stating each agent's final position, the specific unresolved tensions, and the alignment pattern (which two are closer, which is the outlier)
- [ ] Grey's divergence report includes a diagnostic interpretation of the alignment pattern per the three defined cases (Blue+Green vs Red, Red+Green vs Blue, Blue+Red vs Green)
- [ ] Grey never contributes a position, opinion, or perspective to the deliberation content
- [ ] Agent identity persists between sessions: worldview state, journal, relationship map, and track record are loaded at session start and written at session end
- [ ] Each agent's journal entry for a session reflects what it argued, what surprised it, and what it remains uncertain about
- [ ] The Mirror can interview each Moot agent using the standard interview protocol without accessing Grey or other agents' sealed data

---

## Scope

**Included:**

- Three deliberation agents: Blue (ambition), Green (cultivation), Red (limits)
- Grey facilitator: process management, convergence/divergence detection, output synthesis
- Three-round maximum deliberation structure (Opening, Response, Final Position)
- Two output formats: convergence synthesis and divergence report
- Alignment pattern diagnostics in divergence output
- Six-layer agent identity model (personality seed, worldview state, journal, relationship map, track record, context window)
- Plugin interface compatible with Prism and Hoard
- Mirror integration under therapeutic isolation model
- Pre-framed topic input from routing layer

---

## Failure Modes

**Agent output quality:**
- Agent produces output that does not reflect its color orientation (e.g. Blue argues for caution) -> flag in Grey's process log; if persistent across sessions, surface in Mirror interview drift assessment
- Agent's Round 2 response does not reference other agents' positions -> Grey logs failure; response is still passed to other agents but flagged for Mirror review
- Agent produces empty or malformed output -> Grey terminates session, logs error, no partial output delivered to user

**Facilitator failures:**
- Grey cannot determine convergence vs divergence after Round 3 -> default to divergence output; present all three final positions without diagnostic interpretation; flag uncertainty
- Grey's synthesis misrepresents an agent's position -> no runtime detection; this is a Mirror-level concern measured over time through agent interview consistency

**Identity persistence failures:**
- Agent state file missing at session start -> initialize from personality seed only; log that this is a cold start; worldview, journal, relationship map, and track record begin empty
- Agent state file corrupt or partially written -> treat as missing; fall back to personality seed; log corruption event
- Write failure at session end -> retry once; if still failing, log error and preserve session output in memory for manual recovery; do not silently drop session data

**API/model failures:**
- Model call for an agent fails mid-round -> retry once; if still failing, Grey terminates session with error; no partial deliberation output delivered
- Model call returns but output is truncated -> Grey logs truncation; if enough content to proceed, continue; if not, treat as failure and terminate

**Input failures:**
- Empty or missing topic -> return error immediately; no agents invoked
- Topic too long for context window after identity loading -> return error with explanation; no agents invoked

---

## Interface Contracts

Moot exposes the shared council plugin interface. The specific contract is defined at the system level; these are Moot's requirements within that interface.

**Input:**

| Field | Required | Content |
|-------|----------|---------|
| topic | Yes | Pre-framed deliberation topic with sufficient context for agents to engage |
| config | Yes | Council identifier (`moot`) for agent identity loading and Mirror namespacing |

**Output:**

| Field | Content |
|-------|---------|
| outcome | `convergence` or `divergence` |
| rounds_completed | Integer, 1-3 |
| synthesis | Present when outcome is `convergence`. Grey's unified position statement. |
| positions | Present when outcome is `divergence`. Object with `blue`, `green`, `red` final positions. |
| alignment_pattern | Present when outcome is `divergence`. One of: `blue_green_vs_red`, `red_green_vs_blue`, `blue_red_vs_green`, `no_clear_pattern`. |
| diagnostic | Present when outcome is `divergence` and alignment_pattern is not `no_clear_pattern`. Interpretation of what the pattern signals. |
| process_log | Grey's per-round process notes (continue/synthesize/terminate decisions and reasoning). |
| errors | Array of any warnings or flags raised during deliberation. |

**Side effects per session:**

- Each agent's worldview state updated
- Each agent's journal appended
- Each agent's relationship map updated
- Each agent's track record appended

---

## Constraints

- Maximum 3 deliberation rounds per session; Grey may end earlier
- Grey has no personality, no persistent state, no identity layers; it is pure process
- Agents are named by color only: Blue, Green, Red — no seat titles, no character names
- Personality seeds encode color orientation as intrinsic temperament, not role assignment
- Therapeutic isolation: agents cannot access Mirror records, other agents' journals, or other agents' sealed interview data
- Deliberations are manually triggered; no scheduling, no cron, no cadences
- No dual-identity or infiltrator mechanics; each agent is a single identity
- Must expose the same plugin interface as Prism and Hoard

---

## Key Unknowns

1. **[P1]** What personality seeds produce authentic Blue/Green/Red reasoning rather than role-playing a label? The answer determines whether agents generate genuine deliberation or just restate their assigned orientation. Seed research must answer: what does each color sound like in conversation, how do blind spots manifest naturally, and how should seeds be structured so the orientation is temperament rather than instruction.

2. **[P2]** What heuristics should Grey use to detect convergence vs divergence vs productive-but-unresolved? The answer determines Grey's termination logic. Naive keyword matching will fail; Grey needs to assess whether positions are actually moving toward each other or just softening language without changing substance.

---

## Non-Goals

- Multiple agents per color (team-based internal deliberation)
- Scheduling or automated triggering of deliberations
- Routing architecture (system-level concern)
- Shared plugin interface definition (system-level concern)
- Grey participating in deliberation content in any capacity
- Dual-identity, infiltrator, or outsider agent mechanics
- Arbiter or Black/White council configurations (retired)
- Prism or Hoard council design
- Mirror protocol design (shared system, separate spec)
- Personality seed definitions (deferred to research task from Key Unknown 1)
