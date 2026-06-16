# ContextFlow — Paper Outline
## Bounded Runtime-Adaptive Orchestration for LLM-Native Multi-Agent Systems

**Target venue (primary):** IEEE Transactions on Services Computing (systems + empirical study)
**Target venue (secondary):** ICSOC 2026 · AAMAS 2026 (eval/methodology track)
**Companion paper (optional split):** Coordination-quality evaluation methodology → IEEE Intelligent Systems
**Target length:** 10–12 pages (journal) / 8 pages (conference)
**Draft milestone:** M7 (Week 24) — see [`research/project-plan.md`](research/project-plan.md)
**Honesty rule for this paper:** we do **not** claim to be the first runtime-adaptive orchestrator. Recent systems (AdaptOrch, MASFly, CARD, AOrchestra, self-healing orchestrators) already adapt at runtime. Our contribution is *formalisation + operational grounding + coordination-quality evaluation + honest adaptation boundaries*.

---

## Title Options

1. *ContextFlow: Bounded Runtime-Adaptive Orchestration for LLM-Native Multi-Agent Systems in Operational Environments*
2. *Strategy-as-Data: Auditable, Bounded Adaptation of Multi-Agent Coordination at Runtime*
3. *When Should Multi-Agent Systems Re-Plan? Measuring Coordination Quality Under Operational Constraints*

**Recommended:** Option 1 — names the system, states the two key qualifiers (*bounded*, *operational*) that differentiate it from existing adaptive systems.

---

## Positioning sentence (use verbatim in intro + abstract + related work)

> Runtime-adaptive orchestration is now an active area, but existing systems are fragmented, benchmark-centric, and report task success rather than coordination quality. ContextFlow contributes a unified formal model, a durable operational architecture with *bounded* adaptation, and an evaluation methodology that measures coordination quality — including when adaptation does not pay off.

---

## Abstract (write last — skeleton)

Structure: Context (1) → Honest state of the art (1) → Remaining gap (1) → Contributions (2) → Key findings incl. boundary result (2).

Draft skeleton:
> LLM-native multi-agent systems increasingly underpin operational enterprise workflows. Recent work has shown that adapting coordination at runtime — selecting topology, roles, or recovery dynamically — can outperform static designs. However, these systems are evaluated mainly on general benchmarks, define "strategy" incompatibly, and report task success rather than the quality of the coordination itself; the conditions under which adaptation helps or hurts remain unclear. We present ContextFlow, a framework for *bounded* runtime-adaptive orchestration in which a meta-planning layer observes operational context signals and selects coordination strategies from a versioned library, subject to explicit latency, cost, and frequency budgets, with every decision recorded as an auditable trace. We contribute (C1) a taxonomy of runtime context signals, (C2) a strategy-aware architecture integrated with durable workflow execution, (C3) a coordination-quality evaluation methodology, and (C4) a controlled comparison against static, supervisor-routing, and adaptive baselines. Across three operational scenarios with deterministic fault injection, we find [KEY FINDING], and critically that [BOUNDARY FINDING — H4: adaptation adds measurable overhead without benefit on stable, low-complexity tasks].

Keep under 250 words.

---

## Section 1 — Introduction (~1 page)

**Objective:** Motivate the problem, state the *honest* gap, state contributions.

**Paragraph structure:**
1. LLM-native multi-agent systems in operational enterprise environments (claims, banking, healthcare workflows).
2. The shift from design-time to runtime coordination — and that recent systems (2025–2026) already adapt at runtime. **Acknowledge prior art up front.**
3. What is still missing: no unified formal model; benchmark-centric evaluation; no coordination-quality metrics; unknown adaptation boundaries.
4. The operational angle: SLOs, cost budgets, durable execution, controlled failure — under-served by current evaluations.
5. Contributions (numbered C1–C4) with the novelty qualifier for each.
6. Key result preview, including the honest H4 boundary result.
7. Paper structure.

**Key references:** LangGraph, AutoGen, CrewAI; AdaptOrch, MASFly, CARD, AOrchestra, self-healing orchestrators; CLEAR; MAPE-K; ReAct.

---

## Section 2 — Related Work (~1.5–2 pages)

**Objective:** Literature fluency + precise positioning. Mirror [`research/gap-analysis.md`](research/gap-analysis.md).

**2.1 LLM-Native Multi-Agent Frameworks** — AutoGen, LangGraph (incl. supervisor/router), CrewAI, MetaGPT, production SDKs. *Point:* mechanics solved; ad hoc dynamic routing exists; no formal strategy model, no coordination-quality evaluation.

**2.2 Adaptive Workflow Orchestration** — Saga, Temporal/Airflow scheduling, MAPE-K, AI-augmented BPM. *Point:* adapts infrastructure (resources, failure recovery), not multi-agent coordination strategy.

**2.3 LLM Planning and Meta-Reasoning** — ReAct, Reflexion, ToT, GoT, decomposition. *Point:* single-agent / single-episode; not multi-agent coordination-strategy adaptation.

**2.4 Multi-Agent System Evaluation** — AgentBench, GAIA, WebArena (task success); **CLEAR** (cost/latency/efficacy/assurance/reliability). *Point:* even CLEAR measures *outcomes*, not coordination-path quality.

**2.5 Runtime-Adaptive Orchestration (2025–2026) — direct prior work.** AdaptOrch (topology routing from DAGs), MASFly (test-time MAS adaptation + watcher), CARD/AMACP (conditional graph generation), AOrchestra (dynamic sub-agent creation), self-healing orchestrators (recovery loops). *Point + differentiation table:* each is fragmented, benchmark-centric, and outcome-reported; ContextFlow adds operational grounding, durable integration, bounded adaptation, and coordination-quality measurement.

**Closing paragraph:** the precise, honest gap statement (one paragraph).

---

## Section 3 — Problem Formulation (~0.75 pages)

**Objective:** Formal rigour.

**Content:**
- Multi-agent orchestration system formal definition (agents, tasks, coordination topology).
- **Coordination strategy** as a runtime variable selected from a strategy space $\mathcal{S}$ (the library), not a design-time constant.
- **Runtime context signal** formal definition (taxonomy C1) and the context snapshot $c_t$.
- **Bounded adaptive orchestration problem:** at trigger times, choose $S^* \in \mathcal{S}$ maximising coordination quality $Q$ subject to budgets $B$ (latency, cost, switch-frequency). Make the *bounds* explicit in the formalism — this is the key qualifier vs. prior work.
- **Coordination quality $Q$** definition (C3) — distinct from task outcome quality.

---

## Section 4 — ContextFlow Architecture (~2 pages)

**Objective:** Reproducible description; focus on the novel component. Mirror [`architecture/system-design.md`](architecture/system-design.md).

**4.1 System Overview** — the closed adaptation control loop (LLM-native MAPE-K): Monitor → Analyse → Plan/Decide → Execute, over durable execution. Architecture diagram.

**4.2 Context Observation (MONITOR)** — signals collected, granularity, Kafka propagation; reference C1.

**4.3 Signal Interpreter + Mismatch Detector (ANALYSE)** — cheap default path; explicit trigger conditions; why this controls overhead (H4).

**4.4 Strategy Reasoning Engine (PLAN/DECIDE) — core contribution.** Mirror [`architecture/strategy-reasoning.md`](architecture/strategy-reasoning.md):
- Strategy Library (strategy-as-data; topology templates + recovery policies).
- LLM Meta-Planner (input grounding, structured output, determinism aids).
- **Adaptation Governor** (latency/cost/frequency bounds, anti-oscillation, deterministic fallback).
- **Decision Ledger** (auditable trace → metrics + reproducibility).

**4.5 Orchestration Runtime + Agents (EXECUTE)** — LangGraph compiles directive; planning/execution/retrieval agents; pluggable behind controller interface.

**4.6 Durable Execution & Messaging** — Temporal (replan-as-signal, deterministic replay), Kafka (signal bus).

**4.7 Controller Abstraction** — `StaticController`, `SupervisorController`, `AdaptiveController` behind one interface — the basis for fair comparison.

---

## Section 5 — Evaluation Methodology (~1.5 pages)

**Objective:** The coordination-quality framework (C3) + experimental design. Mirror [`research/evaluation-design.md`].

**5.1 Coordination-Quality Metrics (beyond task success / beyond CLEAR):**
- Coordination accuracy — appropriate strategy for observed context (vs. counterfactual hold).
- Adaptation latency — trigger → directive applied.
- Strategy-switch correctness — did the switch improve the targeted signal?
- Agent utilisation efficiency — useful invocations / total.
- Recovery boundedness — replans/task vs. cap; oscillation rate.
- **Adaptation overhead** — extra cost+latency attributable to reasoning (the H4 metric).
- Decision auditability — % decisions with complete, valid trace.

**5.2 Experimental Scenarios:** S1 linear (variable failure rate), S2 branching (variable complexity), S3 dynamic re-planning (variable signal volatility). Grounded in a concrete operational domain (e.g., claims processing) with SLOs.

**5.3 Baselines (strong, not strawman):**
- `Static` — expert-tuned fixed graph (not naive).
- `Supervisor` — LLM router/handoff per step (current best practice).
- `Adaptive` — ContextFlow (bounded strategy reasoning).

**5.4 Implementation Details:** model versions (OpenAI + Ollama local), hardware, Docker Compose, fixed seeds, deterministic fault schedules, #runs (≥5 seeds).

---

## Section 6 — Results (~1.5–2 pages)

**Objective:** Honest, statistically appropriate reporting. Map to RQ4 + H1–H4.

**6.1 Coordination-Quality Comparison (RQ4)** — full metric × controller × scenario table; where adaptive wins and where it does not.
**6.2 Failure Recovery (H1)** — adaptive vs. static vs. supervisor under fault injection; threshold behaviour.
**6.3 Cost/Latency & the Overhead Boundary (H2, H4)** — when reasoning pays off; quantify adaptation overhead on stable low-complexity tasks. **Do not hide H4.**
**6.4 Complexity Variance (H3)** — gap vs. complexity variance; dependence on signal informativeness.
**6.5 Qualitative Strategy Analysis** — which strategies the planner chose; selection consistency (RQ3); auditability via Decision Ledger examples.

---

## Section 7 — Discussion (~0.75 pages)

- Hypothesis verdicts H1–H4 (confirmed / partial / refuted) and what each means.
- **When adaptation is worth its overhead and when it is not** (the practical takeaway).
- Comparison-of-fairness note: strong baselines, identical scenarios/faults.
- Limitations: single domain, LLM non-determinism, scenario coverage, simulated faults.
- Threats to validity (internal, external, construct, reproducibility).

---

## Section 8 — Conclusion and Future Work (~0.5 pages)

- One-paragraph contribution summary (C1–C4).
- Key finding + boundary finding in one sentence each.
- Future work: (1) RL/online-learned triggers and strategy selection; (2) multi-modal context signals; (3) large-scale real enterprise workflow datasets; (4) formal guarantees on adaptation stability.

---

## References

Target: 30–40 references. Prioritise:
- **2025–2026 adaptive orchestration:** AdaptOrch (arXiv:2602.16873), MASFly (arXiv:2602.13671), CARD/AMACP (OpenReview JgvJdICc6P), AOrchestra (arXiv:2602.03786), self-healing orchestrators (arXiv:2606.01416).
- **Evaluation:** CLEAR / Beyond Accuracy (arXiv:2511.14136).
- **Core frameworks:** AutoGen, LangGraph, Temporal.
- **Foundations:** ReAct, Reflexion, ToT, GoT, MAPE-K, Saga, van der Aalst.
- **Author's prior work** (establishes continuity in distributed/operational AI).
- Venue-relevant papers (prior IEEE TSC / ICSOC / AAMAS).

---

## Submission Checklist

- [ ] Intro acknowledges 2025–2026 adaptive systems explicitly (no "first/empty gap" claims)
- [ ] All four contributions (C1–C4) appear explicitly in introduction with novelty qualifiers
- [ ] Differentiation table vs. AdaptOrch/MASFly/CARD/AOrchestra/self-healing present in §2.5
- [ ] All four hypotheses (H1–H4) addressed in results, including H4 overhead/boundary result
- [ ] Strong baselines (expert-tuned static + supervisor routing), not strawman static
- [ ] Coordination-quality metrics defined and computed from Decision Ledger
- [ ] Reproducibility: Docker Compose + pinned versions + seeds + deterministic fault schedules
- [ ] GitHub link + arXiv preprint
- [ ] Abstract under 250 words; within page limit
