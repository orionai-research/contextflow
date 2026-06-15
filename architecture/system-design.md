# System Design

**Project:** ContextFlow — Runtime-Adaptive Orchestration for LLM-Native Multi-Agent Systems
**Author:** Mahboub Parhizkar, PhD
**Lab:** OrionAI
**Document:** `/architecture/system-design.md`
**Status:** Draft v0.1 — June 2026
**Companion docs:** [`gap-analysis.md`](../research/gap-analysis.md) · [`strategy-reasoning.md`](strategy-reasoning.md) · [`../research/project-plan.md`](../research/project-plan.md)

---

## 1. Purpose and scope

This document specifies the ContextFlow system architecture: its layers, components, data
contracts, control flow, failure model, and the technology choices behind each. It is the
authoritative reference for implementation work packages WP2–WP4 in the
[project plan](../research/project-plan.md).

The architecture is shaped by three commitments from the gap analysis:

1. **Operational grounding** — the system must run enterprise-representative workflows with
   explicit SLOs, cost budgets, and controlled failure injection, not only benchmark tasks.
2. **Bounded adaptation** — meta-reasoning must be cheap by default and capped by budgets, so
   that adaptation overhead is measurable and controllable (hypothesis H4).
3. **Evaluation as a first-class concern** — static, supervisor-routing, and adaptive
   controllers must be interchangeable behind one interface to enable fair comparison (C4).

---

## 2. Design goals and non-goals

### Goals
- **G1 — Strategy-as-data:** coordination strategies are versioned library artifacts, not
  free-form LLM text, so decisions are auditable, repeatable, and comparable.
- **G2 — Bounded, auditable adaptation:** every strategy change is governed by latency, cost,
  and frequency budgets and recorded as a decision trace.
- **G3 — Durability:** an in-flight workflow (including a mid-execution replan) survives process
  crashes and resumes deterministically.
- **G4 — Pluggable controllers:** the orchestration runtime accepts a controller interface;
  baselines and the adaptive engine implement the same interface.
- **G5 — Reproducibility:** the entire stack runs from `docker compose up` with pinned versions
  and deterministic seeds where the LLM permits.

### Non-goals
- Sub-100 ms streaming orchestration (out of scope per gap analysis §6).
- Reinforcement-learning strategy adaptation (future work).
- Production-grade auth, multi-tenancy, or UI.
- Beating AdaptOrch/MASFly/CARD on their own benchmarks.

---

## 3. Layered architecture

ContextFlow is organised as a **closed adaptation control loop** (an LLM-native MAPE-K) wrapped
by ingress, durable-execution, and observability layers.

```
 Ingress         ── API Gateway (FastAPI)
   │
 Control loop    ── (1) Monitor  : Context Observer
   │                (2) Analyse  : Signal Interpreter + Mismatch Detector
   │                (3) Plan     : Strategy Reasoning Engine (CORE)
   │                (4) Execute  : Orchestration Runtime (LangGraph) + Agents
   │
 Durability      ── Temporal (durable workflows) + Kafka (signal bus)
   │
 Observability   ── Prometheus + Grafana + Decision Ledger + Evaluation Harness
```

### 3.1 API Gateway Layer
- **Tech:** FastAPI (async).
- **Responsibility:** accept a request, authenticate (static token in research mode), validate
  and normalise it into a `TaskSpec`, start a Temporal workflow, and stream the result back.
- **Why:** lightweight, async-first, Python-native — keeps ingress out of the research path.

### 3.2 Context Observation Layer (MONITOR)
- **Tech:** Python collectors + Kafka producer.
- **Responsibility:** emit typed `ContextSignal` events from runtime telemetry:
  task-complexity estimate, agent health/state, latency-budget consumption, failure history,
  cost-budget consumption, and intermediate-result quality.
- **Why Kafka:** decouples signal production from consumption; lets the control loop and the
  evaluation harness both subscribe without coupling; models real event-driven enterprise flow.

### 3.3 Signal Interpreter + Mismatch Detector (ANALYSE)
- **Responsibility:** aggregate raw signals into the **C1 context-signal taxonomy**, and decide
  *whether* an adaptation is warranted via explicit **trigger conditions** (e.g., latency-budget
  burn rate, repeated failure of an agent, complexity estimate crossing a band boundary).
- **Why a separate detector:** prevents invoking the LLM meta-planner on every step. The default
  path is a cheap rule check; the expensive path (LLM reasoning) only fires on a trigger. This is
  the architectural lever for controlling adaptation overhead (H4).

### 3.4 Strategy Reasoning Engine (PLAN / DECIDE) — CORE
See [`strategy-reasoning.md`](strategy-reasoning.md) for the full component design. Summary:

| Sub-component | Role |
|---------------|------|
| Strategy Library | Versioned catalogue of topology templates (sequential, parallel, hierarchical, hybrid) + recovery policies |
| LLM Meta-Planner | Maps interpreted context → a strategy selection/parameterisation, grounded by the library and the decision history |
| Adaptation Governor | Enforces latency/cost/frequency budgets; vetoes unsafe or oscillating switches; provides a deterministic fallback |
| Decision Ledger | Append-only record of `(signals, candidate strategies, chosen strategy, rationale, outcome)` for audit, metrics (C3), and reproducibility |

Output: an `OrchestrationDirective` — a declarative description of the coordination graph to run.

### 3.5 Orchestration Runtime + Agents (EXECUTE)
- **Tech:** LangGraph compiles the `OrchestrationDirective` into an executable graph.
- **Agents:** Planning, Execution, Retrieval (Qdrant + RAG). Agents are **pluggable** behind a
  uniform interface; the directive decides which run, in what topology, with what fallbacks.
- **Why LangGraph:** mature graph-based agent state control, Python-native, supports the
  supervisor/router patterns we use as a baseline controller.

### 3.6 Durable Execution & Messaging
- **Temporal:** each strategy step runs as a Temporal activity; the workflow holds coordination
  state; a replan arrives as a **signal**, so adaptation is durable and crash-safe.
- **Kafka:** the signal bus for `ContextSignal` events between observation and the control loop.
- **Why both:** Temporal gives *durability and deterministic replay*; Kafka gives *event-driven
  signal propagation*. Together they model an operational enterprise substrate.

### 3.7 Observability & Evaluation Harness
- **Prometheus + Grafana:** latency, throughput, cost, utilisation dashboards.
- **Decision Ledger export:** the source of coordination-quality metrics.
- **Evaluation Harness:** runs scenarios S1–S3 against any controller (static / supervisor /
  adaptive) selected via an **experiment switch**, with deterministic fault injection.

---

## 4. Controller abstraction (the key to fair evaluation)

All orchestration decisions flow through one interface:

```python
class OrchestrationController(Protocol):
    def decide(self, task: TaskSpec, context: ContextSnapshot,
               history: DecisionHistory) -> OrchestrationDirective: ...
```

| Controller | `decide()` behaviour | Role |
|------------|----------------------|------|
| `StaticController` | Returns a fixed, expert-authored directive; ignores context | Baseline (lower bound on adaptivity) |
| `SupervisorController` | LLM picks the next agent per step; no strategy library, no bounds | Baseline (current best practice) |
| `AdaptiveController` | Full Strategy Reasoning Engine with governor + ledger | ContextFlow contribution |

Because every controller emits the same `OrchestrationDirective` into the same runtime, the
experiment harness compares them under identical scenarios, faults, and metrics. This is what
makes C4's comparison credible.

---

## 5. Core data contracts

```python
@dataclass
class TaskSpec:
    task_id: str
    payload: dict                 # domain task (e.g., claim record)
    sla: SLA                      # latency_budget_ms, cost_budget, priority
    scenario: ScenarioId          # S1 | S2 | S3 (experiment tagging)

@dataclass
class ContextSignal:              # emitted continuously to Kafka
    task_id: str
    kind: SignalKind              # COMPLEXITY | AGENT_HEALTH | LATENCY | FAILURE | COST | RESULT_QUALITY
    value: float
    ts: float

@dataclass
class ContextSnapshot:            # interpreter aggregate handed to the planner
    task_id: str
    complexity_band: int
    latency_budget_remaining_ms: int
    cost_budget_remaining: float
    failures: list[FailureEvent]
    result_quality: float

@dataclass
class OrchestrationDirective:     # the strategy, as data
    strategy_id: str              # references Strategy Library template
    topology: Topology            # SEQUENTIAL | PARALLEL | HIERARCHICAL | HYBRID
    agents: list[AgentRef]
    fallbacks: list[RecoveryPolicy]
    bounds: AdaptationBounds      # max_replans, max_cost, max_latency

@dataclass
class DecisionRecord:             # append-only, the audit + metrics substrate
    task_id: str
    snapshot: ContextSnapshot
    candidates: list[str]
    chosen: str
    rationale: str
    triggered_by: SignalKind
    outcome: DecisionOutcome      # accepted | vetoed | rolled_back; latency, cost
```

---

## 6. Control flow (one adaptive task)

1. Gateway validates request → `TaskSpec` → starts Temporal workflow.
2. `AdaptiveController.decide()` produces an initial `OrchestrationDirective` (cold start: from
   complexity band only).
3. Runtime compiles the directive into a LangGraph graph; agents begin executing as Temporal
   activities.
4. Context Observer streams `ContextSignal`s to Kafka throughout execution.
5. Signal Interpreter aggregates into a `ContextSnapshot`; Mismatch Detector checks triggers.
6. **No trigger →** continue on current strategy (cheap default path).
7. **Trigger →** Meta-Planner proposes a new strategy from the Library → Adaptation Governor
   checks budgets/guardrails → accept (signal a replan to the workflow) or veto (keep current).
8. Every decision (accepted or vetoed) is written to the Decision Ledger.
9. On completion, the harness exports outcome + coordination-quality metrics.

---

## 7. Failure model

| Failure | Detection | Handling |
|---------|-----------|----------|
| Agent transient error | Activity exception | Temporal retry policy (bounded) |
| Agent persistent failure | Repeated failures in signal history | Mismatch trigger → recovery policy / topology change |
| Meta-planner timeout / invalid output | Governor validation | Deterministic fallback to last-known-good strategy |
| Adaptation oscillation | Governor frequency cap | Veto further switches within window |
| Process crash | Temporal | Deterministic workflow replay from history |
| Kafka lag | Observer monitoring | Degrade to last snapshot; flag in ledger |

The Adaptation Governor guarantees **liveness** (a bounded fallback always exists) and **safety**
(no unbounded replanning, no cost-budget breach).

---

## 8. Technology stack and rationale

| Concern | Technology | Rationale |
|---------|-----------|-----------|
| Agent coordination | LangGraph | Graph-based agent state control; supports supervisor baseline |
| LLM backend | OpenAI API + Ollama | Interchangeable; Ollama enables reproducible, cost-free local runs |
| Workflow engine | Temporal | Durable, deterministic replay; replan-as-signal |
| Event streaming | Apache Kafka | Decoupled, event-driven context-signal propagation |
| Vector memory | Qdrant | Retrieval agent backing store |
| API gateway | FastAPI | Async, lightweight ingress |
| Deployment | Docker Compose | One-command reproducible environment |
| Observability | Prometheus + Grafana | Latency/throughput/cost benchmarking |
| Metrics/analysis | Python (pandas, matplotlib) | Decision-ledger analysis, figures |

*Note:* Celery was removed from the original sketch — Temporal subsumes durable task execution,
and a second task queue would add complexity without research value.

---

## 9. Deployment topology (research environment)

```
docker compose:
  gateway (FastAPI)            qdrant
  control-loop (observer +     temporal + temporal-ui
    interpreter + engine)      kafka + zookeeper
  agents (planning/exec/       prometheus + grafana
    retrieval workers)         ollama (optional local LLM)
```

All services are pinned and seeded; experiment runs are launched via the harness CLI and write
results to `experiments/results/` for analysis.

---

## 10. Open design questions (tracked in the project plan)

- **DQ1:** Granularity of the Strategy Library — how many templates before the planner's choice
  becomes ill-posed? (informs C1/C2)
- **DQ2:** Trigger sensitivity — static thresholds vs. learned/online thresholds for the Mismatch
  Detector. (informs H3)
- **DQ3:** How much context to put in the meta-planner prompt before grounding degrades? (RQ3)
- **DQ4:** Minimal viable metric set that still captures coordination quality. (C3)

These are resolved during WP1/WP3 and recorded as architecture decision records (ADRs) under
`architecture/` as they are settled.
