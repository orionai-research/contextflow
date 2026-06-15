# Research Gap Analysis

**Project:** ContextFlow — Runtime-Adaptive Orchestration for LLM-Native Multi-Agent Systems  
**Author:** Mahboub Parhizkar, PhD  
**Lab:** OrionAI  
**Document:** `/research/gap-analysis.md`  
**Last updated:** June 2026

---

## 1. Research Domain

This project addresses a problem at the intersection of three converging research areas:

1. **LLM-native multi-agent systems** — the design and coordination of intelligent agents powered by large language models
2. **Adaptive workflow orchestration** — the dynamic execution, monitoring, and re-planning of computational workflows based on runtime state
3. **Operational AI infrastructure** — the engineering of AI systems that function reliably in real enterprise environments with variable, unpredictable conditions

Each area is individually active and well-funded. Between 2025 and 2026, research on runtime-adaptive multi-agent orchestration accelerated significantly — several systems now adapt topology, agent roles, or recovery policy at execution time. **The intersection is no longer empty, but it remains scientifically unsettled:** existing approaches are fragmented, incomparable, and largely evaluated on general NLP benchmarks rather than operational enterprise workflows with explicit SLOs, failure budgets, and coordination-quality metrics.

ContextFlow therefore does not claim to be the first system that adapts orchestration at runtime. It investigates **how to formalise, implement, and empirically evaluate** runtime-adaptive orchestration under operational constraints — and **when adaptation helps versus when it adds cost without benefit**.

---

## 2. Survey of Existing Work

### 2.1 LLM-Native Multi-Agent Frameworks

The period 2023–2025 saw rapid development of frameworks for coordinating LLM-powered agents:

**LangGraph** (LangChain, 2024) introduced graph-based agent state machines, allowing developers to define explicit coordination topologies. Agents transition between states based on LLM outputs and tool results. The default model defines the coordination graph at design time. LangGraph subsequently added **supervisor and router patterns** in which an LLM selects the next agent dynamically — a practical form of runtime routing, but without a formal model of orchestration strategy, adaptation triggers, or coordination-quality evaluation.

**AutoGen** (Wu et al., 2023) proposed a conversational multi-agent model in which agents coordinate through structured message passing. Task decomposition and agent role assignment are handled through LLM-generated conversation. GroupChat supports dynamic speaker selection, but the overall interaction pattern and agent roster remain developer-configured.

**CrewAI** (2024) introduced role-based agent crews with explicit process types (sequential, hierarchical). It improved developer ergonomics but treats coordination strategy as a design-time decision.

**MetaGPT** (Hong et al., 2023) assigned standardised software engineering roles to LLM agents, demonstrating that structured coordination patterns can improve output quality for specific task types. Its coordination model is specialised for software development and does not generalise to operational enterprise workflows.

**OpenAI Agents SDK, Microsoft Agent Framework, and similar production toolkits** (2024–2026) expose handoff-based routing and tool-calling loops. These provide engineering primitives for dynamic delegation but do not supply a research-grade model of strategy adaptation, bounded replanning, or comparative evaluation across orchestration approaches.

**Limitation:** Frameworks solve coordination *mechanics* well and increasingly support ad hoc dynamic routing. They do not provide a **unified formal model** of orchestration strategy, a **taxonomy of runtime context signals** that should drive strategy change, or **rigorous evaluation** of coordination quality as distinct from task outcome. Adaptation, where it exists, is pattern-specific rather than systematically studied.

### 2.2 Adaptive Workflow Orchestration

Workflow orchestration research has addressed adaptation in several forms:

**Exception handling and compensation** — the Saga pattern (Garcia-Molina & Salem, 1987) and BPMN compensation events provide structured mechanisms for recovering from agent or service failures. These address *failure recovery*, not *strategy adaptation*. The compensation logic is also defined at design time.

**Resource-aware scheduling** — distributed workflow engines such as Temporal and Apache Airflow support dynamic task scheduling based on resource availability and priority queues. This addresses *execution efficiency*, not *coordination strategy*.

**Self-adaptive systems** — the autonomic computing paradigm (Kephart & Chess, 2003) proposed systems that monitor, analyse, plan, and execute adaptations (MAPE-K loop). This framework predates LLMs and has been applied primarily to infrastructure management, not to the coordination logic of intelligent agent systems.

**AI-augmented workflow automation** (Dumas et al., 2023; van der Aalst, 2023) — recent work has introduced LLMs as participants in workflow execution, primarily for natural language task handling. The orchestration layer itself typically remains static.

**Limitation:** Classical workflow adaptation operates on infrastructure (resource allocation, failure recovery) rather than on multi-agent *coordination strategy* (topology selection, agent roster, interaction pattern, recovery policy). The bridge between durable workflow engines and LLM-native strategy reasoning remains underdeveloped.

### 2.3 LLM-Based Planning and Meta-Reasoning

Research on LLM-based planning is relevant but distinct:

**ReAct** (Yao et al., 2022) demonstrated that LLMs can interleave reasoning and action in single-agent settings. It does not address multi-agent coordination.

**Reflexion** (Shinn et al., 2023) introduced self-reflection as a mechanism for agents to learn from failures across episodes. This is episodic learning, not within-episode coordination adaptation.

**Tree of Thoughts** (Yao et al., 2023) and **Graph of Thoughts** (Besta et al., 2023) extended LLM reasoning to tree and graph structures for complex problem solving. These operate within a single LLM reasoning context, not across distributed agent coordination.

**LLM-based task decomposition** (Khot et al., 2022; Zhou et al., 2023) investigated how LLMs break complex tasks into subtasks. This informs agent planning but does not address how the coordination *strategy* between agents should adapt to runtime observations.

**Limitation:** Meta-reasoning research has expanded, but most work still operates at the single-agent or single-episode level. Multi-agent coordination strategy adaptation — how a system of agents should change *how they work together* based on collective runtime observations — is increasingly practised but not yet formalised as a distinct, comparable research object.

### 2.4 Evaluation of Multi-Agent Systems

Evaluation methodology has begun to evolve, but significant gaps remain:

Most academic benchmarks (GAIA, AgentBench, WebArena) still measure **task success rate** — whether the agent system produced the correct final output. This metric:

- Does not distinguish between efficient and inefficient coordination paths to the same outcome
- Does not capture coordination quality — redundant agent invocations, unnecessary handoffs, communication overhead
- Does not measure adaptability consistently across varying task conditions
- Does not address operational metrics relevant to enterprise deployment: cost per task, SLA compliance, reliability under load

**CLEAR** (Beyond Accuracy, 2025) proposed a five-dimensional enterprise evaluation framework — Cost, Latency, Efficacy, Assurance, Reliability — with metrics such as cost-normalized accuracy, pass@k reliability, and policy adherence score. This is an important step toward multidimensional agent evaluation, but it measures **agent system outcomes**, not **coordination strategy quality** as a first-class object (e.g., adaptation latency, strategy-switch correctness, agent utilisation efficiency, recovery boundedness).

**Self-healing agentic orchestrators** (2026) introduced formal reliability objectives, failure classes, recovery budgets, and fault-injection benchmarks for tool-augmented LLM systems. This work is closely related to ContextFlow's reliability concerns but focuses on single-orchestrator recovery loops rather than comparative evaluation of coordination topologies under operational workflow constraints.

**Limitation:** No widely adopted framework evaluates **coordination quality** — the efficiency, stability, and appropriateness of the orchestration path — as distinct from both task outcome quality and generic enterprise agent metrics. This makes it difficult to compare adaptive orchestration approaches empirically or to determine when adaptation is worth its overhead.

### 2.5 Runtime-Adaptive Multi-Agent Orchestration (2025–2026)

The most direct challenge to an "empty gap" claim comes from recent work that explicitly addresses runtime orchestration adaptation. This literature must be engaged honestly:

**AdaptOrch** (2026) formalises task-adaptive multi-agent orchestration, dynamically selecting among canonical topologies (parallel, sequential, hierarchical, hybrid) based on task dependency structure. It demonstrates that topology-aware orchestration can outperform static single-topology baselines on coding, reasoning, and RAG benchmarks. Its contribution is primarily **algorithmic topology routing** from task DAGs; it does not address operational context signals (latency budgets, failure history, SLA pressure) or integration with durable workflow infrastructure.

**MASFly / MAS-on-the-Fly** (2026) enables dynamic adaptation of entire multi-agent systems at test time via retrieval-augmented collaboration-pattern instantiation and a global Watcher agent for process supervision. It achieves strong benchmark results (e.g., TravelPlanner) but optimises for **benchmark task success**, not operational coordination metrics, and does not provide a general taxonomy of adaptation triggers for enterprise workflows.

**CARD / AMACP** (2025) proposes conditional graph generation for adaptive multi-agent communication topologies, incorporating dynamic environmental signals and supporting runtime topology updates without retraining. This is architecturally close to ContextFlow's goals but targets academic reasoning benchmarks (HumanEval, MATH, MMLU) under simulated capability shifts rather than enterprise operational scenarios with explicit failure injection and workflow durability requirements.

**AOrchestra** (2026) treats sub-agents as dynamically creatable four-tuples (instruction, context, tools, model) orchestrated by a dedicated meta-agent. It demonstrates that on-demand sub-agent creation improves long-horizon task performance. Its focus is **agent instantiation** rather than formal strategy taxonomy, coordination-quality evaluation, or comparison against production workflow baselines.

**Self-healing agentic orchestrators** (2026) implement monitor–detect–diagnose–recover–verify loops with explicit recovery budgets and fault-injection evaluation. This addresses **reliability under failure** — a key operational concern — but does not comparative-evaluate coordination topology adaptation or agent utilisation efficiency across scenario types.

**Synthesis:** Runtime-adaptive orchestration is **no longer a novel problem statement in the abstract**. Multiple credible systems now adapt topology, roles, or recovery at runtime. The remaining scientific gaps are:

1. **Fragmentation** — each system uses different definitions of "strategy," different triggers, and different benchmarks; results are not comparable.
2. **Missing operational grounding** — most evaluations use general AI benchmarks, not enterprise workflow scenarios with SLOs, durable execution, and controlled failure injection.
3. **Missing coordination-quality metrics** — even recent adaptive systems primarily report task success, not whether the adaptation decision itself was correct, timely, and cost-effective.
4. **Unknown adaptation boundaries** — little work systematically studies **when adaptive orchestration degrades performance** relative to well-tuned static or supervisor-routing baselines.

---

## 3. The Research Gap

Synthesising the above, the gap ContextFlow addresses can be stated precisely and honestly:

> **Runtime-adaptive orchestration for LLM-native multi-agent systems is now an active research area, but lacks a unified formal model of orchestration strategy and runtime context signals, lacks evaluation methodology that measures coordination quality separately from task outcome, and lacks controlled empirical evidence of when adaptation improves operational performance versus when it adds overhead without benefit.**

Three specific sub-gaps:

**Gap 1 — Conceptual (partially open):** Multiple systems adapt orchestration at runtime, but there is no **shared vocabulary or taxonomy** of orchestration strategies and the runtime context signals that should trigger strategy change in operational environments. Existing work uses incompatible definitions (topology, agent roster, communication graph, recovery policy). A formal, operational signal taxonomy — grounded in enterprise constraints rather than benchmark task structure alone — remains a contribution.

**Gap 2 — Architectural (partially open):** Architectures for runtime adaptation exist (supervisor routing, watcher agents, conditional graph generation, self-healing loops), but no architecture integrates **(a)** formal context-signal observation, **(b)** bounded strategy reasoning with explicit adaptation triggers, and **(c)** durable workflow execution** under operational SLOs. The integration of LLM-native strategy reasoning with production-grade workflow infrastructure (Temporal, observability, failure budgets) is underexplored.

**Gap 3 — Empirical (open):** Comparative evidence on adaptive versus static orchestration is fragmented and benchmark-centric. **Controlled experiments** comparing adaptive orchestration against strong baselines — including expert-tuned static graphs, supervisor routing, and recent adaptive systems — across operational scenario types with coordination-quality metrics are lacking. Critically, the conditions under which adaptation **fails** (overhead, instability, wrong replans) are underreported.

---

## 4. Research Questions

**RQ1 (Conceptual):** What constitutes a *runtime context signal* in operational multi-agent orchestration, and how can such signals be formally taxonomised to support reliable strategy reasoning — in a way that complements but differs from task-DAG-based topology routing (AdaptOrch) and benchmark-centric adaptation (MASFly, CARD)?

**RQ2 (Architectural):** What architectural primitives are required to implement a strategy reasoning layer that observes operational context signals, adapts coordination strategy with explicit bounds (latency budget, recovery budget, adaptation frequency), and integrates with durable workflow execution in a distributed LLM-native multi-agent system?

**RQ3 (Algorithmic):** How should an LLM-based meta-planner map observed operational context to coordination strategy decisions, and what grounding mechanisms produce **consistent, auditable** strategy selection compared to ad hoc supervisor routing?

**RQ4 (Empirical):** What are the measurable effects of runtime-adaptive orchestration on coordination accuracy, adaptation latency, agent utilisation, failure recovery, and end-to-end cost — compared to static orchestration, supervisor routing, and recent adaptive baselines — across operational scenarios of varying complexity and failure rate? Under what conditions does adaptation degrade performance?

---

## 5. Hypotheses

**H1:** A multi-agent system with runtime-adaptive orchestration will recover more effectively from agent failures than a system with static orchestration, measured by task completion rate and recovery latency across repeated failure injection experiments — **when failure rate exceeds a threshold above which static fallback paths are exhausted**.

**H2:** Adaptive orchestration will reduce unnecessary agent invocations in low-complexity tasks — where static orchestration over-provisions agents — while maintaining task quality, resulting in measurably lower end-to-end latency and cost for simple operational scenarios.

**H3:** As task complexity variance increases, the performance gap between adaptive and well-tuned static orchestration will widen in coordination-quality metrics — but only when context signals are informative; adaptive systems will not uniformly dominate.

**H4 (null-risk hypothesis):** On stable, low-complexity tasks with predictable structure, runtime-adaptive orchestration will **increase** end-to-end latency and cost relative to static orchestration due to meta-reasoning overhead, without improving task outcome quality. Adaptation overhead will be measurable and non-trivial.

---

## 6. Scope and Boundaries

**In scope:**
- Synchronous and near-real-time operational workflows (request-response and event-driven)
- LLM-native agent coordination using LangGraph as the agent framework
- Three operational scenario types: linear, branching, and dynamic re-planning
- Comparative baselines: static orchestration, supervisor routing, and representative adaptive patterns from recent literature
- Open-source, fully reproducible implementation using Docker Compose
- Coordination-quality metrics extending beyond task success (adaptation latency, agent utilisation, strategy-switch correctness, recovery boundedness)

**Out of scope:**
- Claiming first discovery of runtime-adaptive orchestration (prior work exists — see §2.5)
- Streaming or continuous real-time orchestration (sub-100ms latency requirements)
- Reinforcement learning approaches to strategy adaptation (future work)
- Multi-modal agent inputs beyond text and structured data
- User interface design for enterprise deployment
- Replicating general NLP benchmarks (GAIA, WebArena) as the primary evaluation setting

**Relationship to prior adaptive systems:** ContextFlow does not aim to outperform AdaptOrch, MASFly, or CARD on their respective benchmarks. It aims to **formalise operational adaptation**, **integrate with durable workflow infrastructure**, and **evaluate coordination quality** in enterprise-representative scenarios — a complementary contribution.

---

## 7. Positioning Relative to Prior Work

| Dimension | Framework defaults (LangGraph, CrewAI) | Recent adaptive systems (2025–2026) | ContextFlow |
|-----------|----------------------------------------|-------------------------------------|-------------|
| Orchestration strategy | Design-time, developer-defined | Runtime-adaptive, system-generated | Runtime-adaptive, **operationally grounded** |
| Adaptation trigger | Manual / conditional edges | Task structure, benchmark signals, watcher heuristics | **Formal context-signal taxonomy** (C1) |
| Meta-reasoning scope | Single-agent or ad hoc routing | Topology / role / graph adaptation | **Bounded strategy reasoning layer** integrated with durable workflows |
| Evaluation metric | Task success rate | Task success + limited cost/latency | **Coordination quality framework** (C3) beyond CLEAR-style outcome metrics |
| Target environment | General development | Academic / general AI benchmarks | **Operational enterprise workflows** |
| Adaptation failure analysis | Not studied | Rarely reported | **Explicit hypothesis (H4)** |

**Differentiation summary:**

- vs. **AdaptOrch:** ContextFlow focuses on operational context signals and workflow durability, not DAG-derived topology routing alone.
- vs. **MASFly / CARD:** ContextFlow targets enterprise scenario evaluation with coordination-quality metrics, not benchmark task success optimisation.
- vs. **AOrchestra:** ContextFlow formalises strategy adaptation and evaluates coordination efficiency, not only dynamic sub-agent creation.
- vs. **Self-healing orchestrators:** ContextFlow comparative-evaluates coordination strategy adaptation across scenario types, not only fault recovery loops.
- vs. **CLEAR:** ContextFlow adds **coordination-path metrics** (agent utilisation, adaptation correctness, strategy-switch latency) as distinct from enterprise agent outcome metrics.

---

## 8. References

### Foundational multi-agent and planning
- Besta, M. et al. (2023). *Graph of Thoughts.* arXiv:2308.09687.
- Hong, S. et al. (2023). *MetaGPT.* arXiv:2308.00352.
- Khot, T. et al. (2022). *Decomposed Prompting.* arXiv:2210.02406.
- Shinn, N. et al. (2023). *Reflexion.* arXiv:2303.11366.
- Wu, Q. et al. (2023). *AutoGen.* arXiv:2308.08155.
- Yao, S. et al. (2022). *ReAct.* arXiv:2210.03629.
- Yao, S. et al. (2023). *Tree of Thoughts.* arXiv:2305.10601.
- Zhou, D. et al. (2023). *Least-to-Most Prompting.* arXiv:2205.10625.

### Workflow and operational systems
- Dumas, M. et al. (2023). *Fundamentals of Business Process Management.* Springer.
- Garcia-Molina, H. & Salem, K. (1987). *Sagas.* ACM SIGMOD.
- Kephart, J. & Chess, D. (2003). *The Vision of Autonomic Computing.* IEEE Computer.
- van der Aalst, W. (2023). *Process Mining and AI.* IEEE Intelligent Systems.

### Evaluation
- Beyond Accuracy (2025). *A Multi-Dimensional Framework for Evaluating Enterprise Agentic AI Systems (CLEAR).* arXiv:2511.14136.

### Runtime-adaptive orchestration (2025–2026) — direct prior work
- AdaptOrch (2026). *Task-Adaptive Multi-Agent Orchestration in the Era of LLM Performance Convergence.* arXiv:2602.16873.
- AOrchestra (2026). *Automating Sub-Agent Creation for Agentic Orchestration.* arXiv:2602.03786.
- CARD / AMACP (2025). *Conditional Agentic Graph Designer for Adaptive Multi-Agent Communication.* OpenReview: JgvJdICc6P.
- MASFly / MAS-on-the-Fly (2026). *Dynamic Adaptation of LLM-based Multi-Agent Systems at Test Time.* arXiv:2602.13671.
- Self-Healing Agentic Orchestrators (2026). *Reliable Tool-Augmented LLM Systems.* arXiv:2606.01416.

### Framework documentation
- LangChain (2024). *LangGraph.* Supervisor and multi-agent patterns. https://langchain-ai.github.io/langgraph/
- OpenAI (2025). *Agents SDK.* Handoff-based agent orchestration.

---

*This is a living document. Section 2.5 and the refined gap statement (§3) were updated in June 2026 to reflect the rapidly evolving adaptive orchestration literature. New relevant work should be added to `/research/literature-notes/` as annotated summaries.*
