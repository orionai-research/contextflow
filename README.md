# ContextFlow

**Runtime-Adaptive Orchestration for LLM-Native Multi-Agent Systems in Operational Environments**

> A research project under the [OrionAI](https://github.com/orionai-research) research lab  
> Mahboub Parhizkar, PhD — [Google Scholar](https://scholar.google.ca/citations?user=mZco0BcAAAAJ) · [LinkedIn](https://linkedin.com/in/mahboub-parhizkar) · [ORCID](https://orcid.org/0000-0003-4856-211X)  
> Research metrics: 200+ citations · h-index: 5 · i10-index: 4

---

## Research Question

> *How can LLM-native multi-agent systems dynamically select, adapt, and re-plan their orchestration strategies at runtime based on operational context signals — and what are the measurable effects on coordination quality, efficiency, and reliability?*

In most multi-agent frameworks, orchestration strategy — which agents run, in what order, with what fallbacks, under what coordination pattern — is still decided primarily at **design time**. When runtime conditions change (task complexity shifts, an agent fails, intermediate results alter the problem), systems built on static graphs often cannot adapt their coordination approach effectively.

Recent work (2025–2026) has begun addressing this: systems such as AdaptOrch, MASFly, CARD, and AOrchestra now adapt topology, roles, or recovery at runtime. **The problem is no longer unstudied — but it is not settled.** Existing approaches are fragmented, evaluated mainly on general AI benchmarks, and rarely measure *coordination quality* separately from task success.

ContextFlow investigates **operational orchestration science**: how to formalise runtime context signals, implement bounded strategy reasoning integrated with durable workflows, and empirically evaluate when adaptive orchestration improves coordination efficiency and reliability in enterprise-representative environments — and when it adds overhead without benefit.

---

## The Problem in Detail

Current LLM-native orchestration frameworks (LangGraph, AutoGen, CrewAI) have solved the *mechanics* of agent coordination well. They provide:

- Graph-based agent state machines
- Role-based task decomposition
- Tool-augmented reasoning loops
- Conversational agent interaction

What remains unsolved is **systematic adaptive strategy selection under operational constraints**. Framework defaults still require developers to specify coordination patterns upfront. Ad hoc dynamic routing (supervisor agents, conditional edges) exists but lacks formal models, comparable evaluation, and evidence of when adaptation helps versus hurts.

This matters most in **operational environments** — enterprise systems where tasks are variable, conditions change, agents fail, and the cost of wrong coordination is real. Insurance claim processing, banking transaction orchestration, healthcare workflow automation — these environments need more than benchmark-optimised adaptation; they need measurable coordination quality, bounded recovery, and integration with durable workflow infrastructure.

---

## Research Contributions

| ID | Contribution | Novelty note |
|----|-------------|--------------|
| C1 | A **taxonomy of runtime context signals** for operational orchestration decisions (task complexity, agent state, intermediate result quality, latency budget, failure history) | Complements task-DAG routing (AdaptOrch); grounded in enterprise constraints |
| C2 | A **strategy-aware orchestration layer** integrated with durable workflows — bounded meta-reasoning about *how* to coordinate agents | Builds on supervisor/adaptive patterns; focuses on operational integration, not first adaptive system |
| C3 | An **evaluation methodology** for coordination quality beyond task success — adaptation latency, agent utilisation, strategy-switch correctness, recovery boundedness | Extends CLEAR-style enterprise metrics with coordination-path metrics |
| C4 | **Empirical findings** comparing static, supervisor-routing, and adaptive orchestration across operational scenarios — including when adaptation fails | Strong baselines required; honest negative results expected (H4) |

---

## Architecture

ContextFlow is built around a **closed adaptation control loop** — an LLM-native reinterpretation of the MAPE-K pattern (Monitor → Analyse → Plan → Execute, over shared Knowledge). The loop is deliberately *bounded*: every adaptation is constrained by latency, cost, and frequency budgets so that meta-reasoning cannot run away (directly addressing H4).

```
                          ┌───────────────────────────────────────────────┐
                          │            API GATEWAY  (FastAPI)              │
                          │   auth · request → TaskSpec · result egress    │
                          └───────────────────────────┬───────────────────┘
                                                       │ TaskSpec
                                                       ▼
   ┌─────────────────────────────────  ADAPTATION CONTROL LOOP  ─────────────────────────────────┐
   │                                                                                              │
   │   (1) MONITOR                 (2) ANALYSE                  (3) PLAN / DECIDE                  │
   │  ┌──────────────────┐       ┌──────────────────┐        ┌──────────────────────────────┐    │
   │  │ Context Observer │  sig  │ Signal Interpreter│ trigger│  Strategy Reasoning Engine    │    │
   │  │ complexity·health│──────▶│ + Mismatch Detector──────▶│  ◄── CORE RESEARCH ARTIFACT    │    │
   │  │ latency·failures │       │ (trigger conditions)│      │  LLM meta-planner             │    │
   │  │ cost·result-qual │       └──────────────────┘        │  Strategy Library (templates) │    │
   │  └────────▲─────────┘                                   │  Adaptation Governor (bounds) │    │
   │           │ telemetry                                   │  Decision Ledger (audit trace)│    │
   │           │                                             └───────────────┬──────────────┘    │
   │           │                                              OrchestrationDirective              │
   │           │                                                             ▼                    │
   │  ┌────────┴──────────────────────────────  (4) EXECUTE  ───────────────────────────────┐    │
   │  │           Orchestration Runtime  —  LangGraph graph compiled from directive          │    │
   │  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────────┐                        │    │
   │  │  │ Planning     │    │ Execution    │    │ Retrieval        │   agents are pluggable  │    │
   │  │  │ Agent        │    │ Agent        │    │ Agent (Qdrant RAG)│                        │    │
   │  │  └──────────────┘    └──────────────┘    └──────────────────┘                        │    │
   │  └─────────────────────────────────────────────────────────────────────────────────────┘    │
   └──────────────────────────────────────────────────────────────────────────────────────────────┘
                                                       │  durable steps · signals · replans
                                                       ▼
   ┌──────────────────────────────────────────────────────────────────────────────────────────────┐
   │  DURABLE EXECUTION & MESSAGING      Temporal (durable workflows/retries) · Kafka (signal bus)   │
   └──────────────────────────────────────────────────────────────────────────────────────────────┘
                                                       │
   ┌──────────────────────────────────────────────────────────────────────────────────────────────┐
   │  OBSERVABILITY & EVALUATION HARNESS    Prometheus · Grafana · decision traces · metrics export   │
   │  Baseline controllers:  [static graph] · [supervisor routing] · [adaptive]  ← experiment switch  │
   └──────────────────────────────────────────────────────────────────────────────────────────────┘
```

### Core component — Strategy Reasoning Engine

The engine is the research artifact; everything else is proven, production-grade infrastructure. It comprises five sub-components:

| Sub-component | Responsibility |
|---------------|----------------|
| **Signal Interpreter + Mismatch Detector** | Convert raw telemetry into typed context signals (C1 taxonomy); decide *whether* adaptation is warranted (avoids reasoning on every step) |
| **Strategy Library** | A versioned catalogue of coordination strategies — topology templates (sequential, parallel, hierarchical, hybrid) plus recovery policies — that the planner selects and parameterises rather than free-forming |
| **LLM Meta-Planner** | Maps interpreted context → a coordination strategy decision, grounded by the Strategy Library and prior decisions (RQ3) |
| **Adaptation Governor** | Enforces hard bounds: latency budget, cost budget, max adaptation frequency, and guardrails that veto unsafe/oscillating switches |
| **Decision Ledger** | Records every decision (signals in → strategy out → outcome) as an auditable trace — the substrate for coordination-quality metrics (C3) and reproducibility |

### Design principles

1. **Bounded adaptation** — adaptation is opt-in per trigger and capped by budgets; the default path is cheap.
2. **Strategy as data** — strategies are library templates, not free-form LLM output, making decisions auditable and comparable.
3. **Durable by construction** — strategy steps execute as Temporal activities; a replan is a workflow signal, so adaptation survives crashes.
4. **Evaluation is first-class** — the same harness runs static, supervisor-routing, and adaptive controllers behind one switch, so comparisons are apples-to-apples (C4).
5. **Differentiation** — prior adaptive systems (AdaptOrch, MASFly, CARD, self-healing orchestrators) inform the design; ContextFlow's deltas are operational grounding, durable-workflow integration, the Adaptation Governor's explicit bounds, and coordination-quality measurement.

> Full detail: [`architecture/system-design.md`](architecture/system-design.md) · [`architecture/strategy-reasoning.md`](architecture/strategy-reasoning.md)

---

## Repository Structure

```
contextflow/
├── research/
│   ├── gap-analysis.md           # Formal research gap and literature review
│   ├── project-plan.md           # Full plan + Definition of Done (source of truth)
│   ├── taxonomy.md               # C1 runtime context-signal taxonomy
│   ├── hypotheses.md             # Formal hypotheses and expected findings
│   ├── evaluation-design.md      # Coordination-quality metrics + experiment design
│   ├── findings.md               # Empirical results and hypothesis verdicts
│   └── literature-notes/         # Annotated summaries of key papers
├── architecture/
│   ├── system-design.md          # Architecture decisions and rationale
│   ├── strategy-reasoning.md     # Core component (Strategy Reasoning Engine) design
│   ├── adr/                      # Architecture decision records (DQ1–DQ4 …)
│   └── diagrams/                 # System and component diagrams
├── strategy_library/             # Versioned coordination strategy templates (YAML)
├── contextflow/
│   ├── gateway/                  # FastAPI gateway
│   ├── context_observer/         # MONITOR: runtime signal collection
│   ├── signal_interpreter/       # ANALYSE: aggregation + mismatch/trigger detection
│   ├── strategy_engine/          # CORE: meta-planner · governor · decision ledger
│   ├── controllers/              # static · supervisor · adaptive (one interface)
│   ├── agents/
│   │   ├── planning_agent/
│   │   ├── execution_agent/
│   │   └── retrieval_agent/
│   ├── workflow/                 # Temporal workflow + activity definitions
│   └── infrastructure/           # Kafka, Docker configurations
├── experiments/
│   ├── scenarios/                # Operational scenario definitions (S1–S3)
│   ├── harness/                  # Experiment matrix runner + fault injection
│   ├── results/                  # Raw and processed results (checksummed)
│   └── analysis/                 # Notebooks + figure generation scripts
├── paper/
│   └── drafts/                   # Working paper sections
├── docker-compose.yml
└── README.md
```

---

## Technology Stack

| Concern | Technology | Rationale |
|---------|-----------|-----------|
| Agent coordination | LangGraph | Graph-based agent state control, Python-native |
| LLM backend | OpenAI API + Ollama | Interchangeable — reproducible in any environment |
| Workflow engine | Temporal | Python-native, production-grade, LLM-era aligned |
| Event streaming | Apache Kafka | Real-time context signal propagation |
| Vector memory | Qdrant | Fast similarity search for retrieval agent |
| API gateway | FastAPI | Lightweight, async-first |
| Deployment | Docker Compose | Fully reproducible research environment |
| Observability | Prometheus + Grafana | Latency and throughput benchmarking |

---

## Evaluation Scenarios

To ensure generalisability, adaptive vs. static orchestration will be evaluated across three operational scenario types:

| Scenario | Description | Variable factor |
|----------|-------------|-----------------|
| S1 — Linear workflow | Sequential task execution, low complexity | Agent failure rate |
| S2 — Branching workflow | Conditional task routing, medium complexity | Task complexity variance |
| S3 — Dynamic re-planning | Mid-execution context shift requiring strategy change | Context signal volatility |

Metrics collected: coordination accuracy, adaptation latency, agent utilisation, failure recovery time, end-to-end throughput, strategy-switch correctness, and adaptation overhead (testing H4).

Baselines include static orchestration, supervisor routing, and representative adaptive patterns from recent literature — not only naive static graphs.

---

## Research Roadmap

The roadmap below is the high-level view. The authoritative plan — with per-work-package
**Definition of Done**, gates, risks, and reproducibility checklist — lives in
[`research/project-plan.md`](research/project-plan.md). The schedule is **~24 weeks** (gated), with
an MVP baseline at week 8; the original 8-week sketch is retained only as the MVP milestone (M3).

| Milestone | Target | Work package | Exit gate |
|-----------|--------|--------------|-----------|
| M0 — Kickoff | Wk 0 | — | Gap analysis + architecture committed ✅ |
| M1 — Foundations ready | Wk 2 | WP0 | Repo scaffolded, `docker compose up` healthy |
| M2 — Formal model frozen | Wk 5 | WP1 (C1) | Context-signal taxonomy + strategy library |
| M3 — Baseline MVP | Wk 8 | WP2 (RQ2) | Static + supervisor controllers run S1, durable |
| M4 — Adaptive engine working | Wk 13 | WP3 (C2) | Bounded adaptive controller runs S1–S3 |
| M5 — Evaluation harness ready | Wk 16 | WP4 (C3) | Coordination-quality metrics automated |
| M6 — Results complete | Wk 21 | WP5 (C4) | Full matrix run; H1–H4 verdicts (incl. H4 overhead) |
| M7 — Draft + artifact | Wk 24 | WP6 | First paper draft + reproducible public artifact |

Current focus: **Phase 1 / WP0–WP1** — literature notes (AdaptOrch, MASFly, CARD, CLEAR,
self-healing orchestrators), context-signal taxonomy, and repository scaffolding.

---

## Research Positioning

ContextFlow is positioned as **operational AI systems research** — the study of intelligent systems that act, coordinate, and adapt in real enterprise environments.

- **Not** a tutorial implementation of LangGraph or AutoGen
- **Not** a chatbot or conversational AI application
- **Not** a benchmark of existing LLM capabilities
- **Is** an investigation of how to formalise, implement, and empirically evaluate runtime-adaptive orchestration under operational constraints — building on but distinct from recent adaptive systems (AdaptOrch, MASFly, CARD, AOrchestra)

This framing aligns with postdoctoral research tracks in autonomous agents, intelligent systems, AI infrastructure, and distributed AI at both university research labs and industry research groups.

---

## About OrionAI

OrionAI is an independent AI research initiative focused on operational intelligent systems, distributed AI infrastructure, and LLM-native orchestration architectures for real-world enterprise environments.

---

## Status

🔬 **Active research** — Phase 1 in progress

*Research inquiries and collaboration proposals welcome via [LinkedIn](https://linkedin.com/in/mahboub-parhizkar)*
