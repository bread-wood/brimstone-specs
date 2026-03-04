# System Capabilities — Full Platform Use Cases

**Repos:** brimstone · jobman · moot · breadwinner
**Date:** 2026-03-04

Evidence base: all milestone specs, architecture review, dependency graph

---

## What This Platform Is

Four repos that together form a personal AI operating system — an autonomous loop
that watches markets, reads culture, deliberates with structured councils, acts on
decisions, and continuously improves its own software.

The architecture has two layers:

**Infrastructure layer** (brimstone, jobman): Automation backbone. Brimstone dispatches
AI agents to build software. Jobman collects signals from the world, maintains a
knowledge graph, and coordinates data flow between product repos.

**Product layer** (moot, breadwinner): Decision-making surface. Moot hosts deliberation
councils that reason over life and financial questions. Breadwinner manages a live
portfolio and executes trades.

---

## Core Use Cases

### 1. Morning Financial Brief

**What happens**: Every morning, jobman's signals pipeline refreshes the portfolio
snapshot (positions, Greeks, IV rank, P&L vs cost basis), runs the Reddit sentiment
job to capture overnight discourse in financial subreddits, and writes both to the
signals store. Breadwinner's planning pipeline runs, evaluating exit rules and scanning
for new options candidates. Moot's financial council receives the assembled context —
current positions, IV environment, sentiment signals — and deliberates on whether any
position warrants review.

**User experience**: A Telegram message arrives with a structured brief: open positions
and their current P&L delta since last review, any positions the financial council
flagged as needing attention (with the council's reasoning summarized), and the top
options candidates identified by the scanner. No manual log-in, no dashboard to poll.
The system surfaces what changed and why it might matter.

**What it requires**: breadwinner/v0.2.0 (options positions), v0.3.0 (IV rank),
v0.4.0 (snapshot format), jobman/v0.2.0 (portfolio delivery), jobman/v0.3.0 (signals
pipeline), moot/v0.6.0 (financial council), moot/v0.7.0 (signals context injection)

---

### 2. Financial Council Deliberation on a Position

**What happens**: A position hits a threshold — IV rank spikes, P&L crosses a loss
limit, or the expiry window narrows. Jobman's verdict bridge detects the trigger
(pulled from breadwinner's state snapshot) and initiates a financial council
deliberation in moot. The council receives the full position context: entry price,
current Greeks, IV environment, the original thesis, and the Reddit sentiment
signals for the underlying. Agents deliberate — one anchors to base rates and
historical outcomes for similar setups, one argues from the current data, one plays
skeptic. If the decision is high-stakes (defined by position size and time-to-expiry),
the deliberation upgrades to Opus. The council produces a verdict: hold, exit, adjust.

**User experience**: A Telegram notification presents the council's verdict with
a summary of the key arguments and the dissent (if any). The user can approve the
verdict or override it. If approved, jobman routes the verdict back to breadwinner,
which executes the trade.

**What makes this non-trivial**: The council isn't pattern-matching on rules. It's
reasoning about the specific position in its current context. The skeptic agent
can surface information the user hasn't considered. The base-rate anchor prevents
overreacting to noise. The outcome is logged to the knowledge graph as a structured
connection between the setup, the reasoning, and the result — feeding future council
deliberations with track record data.

**What it requires**: Full Wave 3 (moot/v0.6.0, jobman/v0.2.0, breadwinner/v0.4.0)

---

### 3. Structured Life Deliberation (RGB Council)

**What happens**: The user submits a question — about a career decision, a
relationship tension, a values conflict, a major purchase. The RGB council
activates: Blue (ambition and growth) argues for expansion and opportunity cost.
Green (cultivation and depth) argues for sustainability and what already exists.
Red (limits and risk) argues for what could go wrong. Grey facilitates the rounds,
detects when the three perspectives are converging on a shared insight vs. when
they're in productive tension that shouldn't be prematurely resolved.

Each agent maintains a six-layer identity — a personality seed, accumulated worldview
state from prior sessions, a journal of past deliberations, a relationship map
tracking how their view of the user's values has shifted over time, and a track
record of what their prior recommendations led to. The agents know each other and
reference prior debates.

**User experience**: A deliberation transcript with labeled voices, a Grey summary
of where the council landed, and a clear statement of what remained unresolved
(with Grey's assessment of whether resolution is needed now or whether sitting with
the tension is the right answer). Mirror can interview any agent afterward to probe
their reasoning.

**What makes this non-trivial**: Accumulated identity means the council gets better
at deliberating for this specific user over time. Red's prior objections that turned
out to be right carry forward as track record. Green's pattern of undervaluing
opportunity cost (if that's what the data shows) becomes known to Grey. The system
develops institutional memory about how good its own reasoning has been.

**What it requires**: moot/v0.4.0 (CouncilPlugin), moot/v0.5.0 (RGB council impl)

---

### 4. Software Development on Autopilot

**What happens**: A GitHub issue in any of the four product repos is labeled with
the right stage label (`stage/research`, `stage/design`, `stage/impl`) and module
label. Brimstone picks it up, claims it, creates a branch, dispatches an isolated
worker agent, monitors the PR, resolves review feedback, and squash-merges after
CI passes. For multi-issue milestones (e.g., a 6-issue impl stage), brimstone
dispatches multiple agents in parallel, respects module isolation so no two agents
touch the same code, and sequences them when one agent's output is required by the
next.

**User experience**: The user opens an issue, labels it, and checks back later to
find a merged PR. For a full milestone with research/design/impl stages, brimstone
runs each stage in order, carries findings forward via spec-update, and surfaces only
the decisions that require human input (design approvals, Key Unknown resolutions).

**Autonomous mode**: With brimstone/v0.4.0 (long-game campaign mode), the user
specifies a sequence of milestones and brimstone drives the entire roadmap — picking
up where each milestone leaves off, propagating findings into the next spec, managing
the wave structure from the dependency graph automatically.

**What it requires**: brimstone/v0.2.0 (safe dispatch), v0.2.1 (multi-model),
v0.2.2 (multi-repo), v0.3.0 (rolling loop), v0.3.1 (spec-update), v0.4.0 (campaign)

---

### 5. Knowledge Graph That Learns From Everything

**What happens**: Every significant event in the system produces structured knowledge.
Each financial council verdict is stored as a connection in the knowledge graph:
the setup, the reasoning, the vote, the outcome. Each life deliberation produces
connections between the question posed, the arguments made, and how the user acted.
Each breadwinner planning run emits `ConnectionSignal` objects — which setups
triggered, which were passed, what the IV environment looked like. Over time,
the knowledge graph builds a queryable record of the system's reasoning and its
track record.

Jobman's knowledge wire also reads moot's libraries — structured knowledge about
deliberation frameworks, financial concepts, historical context that moot agents
use as reference. The library reader keeps an index updated as moot's libraries
evolve, and injects relevant library context into new deliberations via
`ContextBrief` objects assembled by the `ContextAssembler`.

**User experience**: The user can query the knowledge store — "what setups did the
financial council flag but the user didn't act on, and what happened to those
positions?" The answer is a structured report drawn from the graph, not a chat
summary. Council agents can query the graph during deliberation to surface their
own track record on similar questions.

**What it requires**: jobman/v0.1.0 (knowledge wire), breadwinner/v0.4.0
(ConnectionSignal emission), moot/v0.7.0 (context injection from knowledge store)

---

### 6. Reddit as a Financial Intelligence Layer

**What happens**: The Reddit sync job runs on a scheduled cadence, reading
financial subreddits (wallstreetbets, options, investing, and tickers relevant to
open positions). It produces `SubredditSnapshot` objects that capture post volume,
sentiment trends, notable threads, and ticker mentions. These land in jobman's
signals store. Moot's financial council reads the signals store during deliberation —
when discussing a position, agents can see whether the underlying has unusual Reddit
activity that might signal retail crowding or a narrative shift.

**What makes this non-trivial**: Reddit is not treated as a source of alpha in
isolation — it's one input among many in a structured deliberation. The council
agent that reads it applies the same skepticism filter as any other data source.
The knowledge graph tracks whether past Reddit-signal-influenced decisions performed
better or worse than those where Reddit signals were absent or contradictory.

**What it requires**: jobman/v0.3.0 (Reddit sync job), moot/v0.7.0 (signals context)

---

### 7. The Platform Builds Itself

**What happens**: When an architecture review produces a recommended spec change
(e.g., "migrate moot scheduler to jobman job handlers"), brimstone's spec-update
stage propagates the recommendation into the affected spec's next stage. The spec
gains a new implementation task. Brimstone picks it up in the next campaign cycle,
dispatches a worker, and the system self-heals its architectural violations.

The four repos are themselves managed by brimstone as a platform. When a new
capability is identified — say, a new data source, a new council type, a new
breadwinner strategy phase — the user writes a minimal spec or GitHub issue and
brimstone drives the implementation. The system is its own primary user.

**What it requires**: brimstone/v0.3.1 (spec-update), v0.4.0 (long-game)

---

## Platform Properties That Emerge From Integration

These aren't features of individual repos — they emerge from the four repos working
together.

**Single source of truth per concern**: LLM cost tracking lives in breadmin-llm's
`token_usage` table. Data collection lives in jobman job handlers. Council deliberation
lives in moot. Trade execution lives in breadwinner. No duplication of responsibility
means no drift between systems.

**Deliberation-gated execution**: Breadwinner doesn't trade autonomously on rule
triggers alone. High-stakes decisions route through moot's financial council before
execution. The council can say no. This is a human-in-the-loop architecture where
the "human in the loop" role is partly played by a structured multi-agent council —
with the actual human receiving a verdict summary and retaining override authority.

**Traceable reasoning**: Every council verdict, every planning run, every agent
decision is logged with its inputs and reasoning. The knowledge graph makes this
queryable. The system can explain why it recommended any action by showing the
full deliberation record.

**Compounding intelligence**: Each deliberation makes the next one better. Agent
identity accumulates track record. The knowledge graph grows. Library content is
indexed and retrievable. The system becomes progressively more calibrated to this
user's specific values, risk tolerance, and decision patterns — not by fine-tuning,
but by building structured memory through normal use.

---

## Full Platform Stack (Wave 5 Complete)

```
User (Telegram / CLI)
        ↕
 brimstone                       ← orchestrates software development
        ↕
 breadmin-jobman                 ← signals, knowledge, coordination bus
   ├── knowledge wire            ← moot library index + context assembly
   ├── verdict bridge            ← financial council ↔ breadwinner relay
   └── signals pipeline          ← Reddit sync + portfolio refresh
        ↕
 moot                            ← deliberation councils
   ├── RGB council               ← life decisions (Blue/Green/Red/Grey)
   └── financial council         ← trading decisions (Meridian + roster)
        ↕
 breadwinner                     ← portfolio execution
   ├── options engine            ← scanner, Greeks, exit rules
   ├── IV infrastructure         ← rank, backtester, signal pipeline
   └── crossing bridge           ← verdict receiver, jobman snapshot
```

The user sits at the top, receiving notifications and approving verdicts.
Brimstone sits alongside, building and maintaining the whole system.
Jobman is the coordination layer — no product repo calls another directly.
Moot and breadwinner are isolated product domains that communicate only through
jobman-owned contracts.
