# ContextFlow — Project Plan & Definition of Done

**Project:** ContextFlow — Runtime-Adaptive Orchestration for LLM-Native Multi-Agent Systems
**Author:** Mahboub Parhizkar, PhD
**Lab:** OrionAI
**Document:** `/research/project-plan.md`
**Status:** Draft v0.1 — June 2026
**Companion docs:** [`gap-analysis.md`](gap-analysis.md) · [`../architecture/system-design.md`](../architecture/system-design.md) · [`../architecture/strategy-reasoning.md`](../architecture/strategy-reasoning.md)

---

## 1. How to read this plan

This is the single source of truth for *what counts as done*. It defines:

- the **overall Definition of Done (DoD)** for the project (§3),
- a **work breakdown structure** of work packages, each with its own DoD (§5),
- a **realistic, gated timeline** (§6),
- **traceability** from work packages to contributions, research questions, and hypotheses (§4),
- a **risk register** (§7), **reproducibility checklist** (§8), and **publication mapping** (§9).

A task is *done* only when its acceptance criteria are met **and** its artifacts are committed and
reproducible. "Works on my machine" is not done.

---

## 2. Objective

Deliver a formalised, implemented, and empirically evaluated framework for **bounded
runtime-adaptive orchestration** of LLM-native multi-agent systems in operational settings, plus
an **evaluation methodology for coordination quality** that distinguishes adaptive orchestration
from static and supervisor-routing baselines — including an honest account of when adaptation does
*not* help.

---

## 3. Overall Definition of Done (project-level)

The project is considered complete when **all** of the following hold:

- [ ] **D1 — Formal model:** the C1 context-signal taxonomy and the strategy representation are
      specified, documented, and instantiated in the Strategy Library.
- [ ] **D2 — Working system:** the full stack runs from `docker compose up`, executes scenarios
      S1–S3, and supports the three controllers (static, supervisor, adaptive) behind one switch.
- [ ] **D3 — Bounded adaptation:** the Adaptation Governor demonstrably enforces latency, cost,
      and frequency budgets; no run exceeds its budgets; a deterministic fallback always exists.
- [ ] **D4 — Evaluation methodology:** the coordination-quality metric suite (C3) is implemented,
      documented, and computed automatically from the Decision Ledger.
- [ ] **D5 — Empirical results:** a complete experiment matrix (controllers × scenarios × failure
      rates × seeds) is executed, analysed, and reported — including the H4 overhead finding.
- [ ] **D6 — Reproducibility:** any reported figure can be regenerated end-to-end from committed
      code, config, seeds, and raw results (§8 checklist passes).
- [ ] **D7 — Dissemination:** a complete first paper draft exists, mapped to a target venue, with
      a public artifact (repo + README + reproduction guide).

---

## 4. Traceability matrix

| Work package | Contributes to | Answers | Tests |
|--------------|----------------|---------|-------|
| WP0 Foundations | — | — | — |
| WP1 Formal model & taxonomy | C1 | RQ1 | — |
| WP2 Core infra & baselines | C2 (partial) | RQ2 | — |
| WP3 Strategy Reasoning Engine | C2 | RQ2, RQ3 | H1–H4 (enables) |
| WP4 Evaluation methodology & harness | C3 | RQ4 | H1–H4 |
| WP5 Experiments & analysis | C4 | RQ4 | H1–H4 |
| WP6 Writing & dissemination | C1–C4 | all | all |

---

## 5. Work breakdown structure (with per-WP Definition of Done)

### WP0 — Foundations & repository scaffolding
**Objective:** make the repo a research-grade, reproducible environment.
**Tasks:**
- Scaffold the repository tree (per README structure).
- `docker-compose.yml` skeleton with pinned service versions.
- Python project (deps, lint, format, test, pre-commit), `Makefile`/CLI entry points.
- ADR template + contribution/reproducibility conventions.

**Definition of Done:**
- [ ] `docker compose up` starts all infra services healthy (Temporal, Kafka, Qdrant, Prometheus, Grafana, Ollama optional).
- [ ] `make test` and `make lint` run green in CI (or local script).
- [ ] CONTRIBUTING + ADR template committed; versions pinned.

---

### WP1 — Formal model & context-signal taxonomy (C1, RQ1)
**Objective:** define *what* the system observes and *what* a strategy is.
**Tasks:**
- Specify the C1 taxonomy of runtime context signals (kinds, semantics, units, sources).
- Define the strategy representation and the initial Strategy Library templates.
- Define trigger conditions (mismatch detection) per signal kind.
- Resolve DQ1 (library granularity) initial position; record as ADR.

**Definition of Done:**
- [ ] `research/taxonomy.md` enumerates each signal: definition, source, type, example.
- [ ] ≥ 4 strategy templates committed under `strategy_library/` and schema-validated.
- [ ] Trigger conditions documented and unit-tested against synthetic snapshots.
- [ ] ADR-001 (library granularity) committed.

---

### WP2 — Core infrastructure & baselines (C2 partial, RQ2)
**Objective:** a runnable orchestration substrate with two baselines, *before* adaptivity.
**Tasks:**
- FastAPI gateway: request → `TaskSpec` → start Temporal workflow.
- Context Observer emitting `ContextSignal`s to Kafka.
- LangGraph orchestration runtime compiling an `OrchestrationDirective`.
- Three agents (planning, execution, retrieval + Qdrant RAG) behind a uniform interface.
- `StaticController` and `SupervisorController` implementing `OrchestrationController`.
- Temporal durability: replan-as-signal plumbing (even if adaptive logic is stubbed).

**Definition of Done:**
- [ ] An end-to-end task runs through gateway → workflow → agents → result.
- [ ] Both baseline controllers run scenario S1 to completion.
- [ ] A killed worker mid-task resumes deterministically (durability test passes).
- [ ] Context signals are visible on Kafka and in Grafana.

---

### WP3 — Strategy Reasoning Engine (C2, RQ2/RQ3)
**Objective:** implement the core artifact — bounded, auditable adaptive control.
**Tasks:**
- Signal Interpreter + Mismatch Detector (cheap default path).
- LLM Meta-Planner with structured output + library grounding.
- Adaptation Governor enforcing budgets and anti-oscillation; deterministic fallback.
- Decision Ledger (append-only, queryable).
- `AdaptiveController` wiring the above into the controller interface.
- Resolve DQ2 (trigger thresholds) and DQ3 (prompt context budget) initial positions (ADRs).

**Definition of Done:**
- [ ] `AdaptiveController` runs S1–S3 to completion.
- [ ] Governor provably caps replans/cost/latency (tests force each bound and assert veto).
- [ ] Malformed planner output triggers deterministic fallback (test).
- [ ] Every decision produces a complete `DecisionRecord` (100% trace coverage test).
- [ ] Selection-consistency probe implemented (repeated identical inputs → stability score).
- [ ] ADR-002 (triggers), ADR-003 (prompt budget) committed.

---

### WP4 — Evaluation methodology & harness (C3, RQ4)
**Objective:** measure coordination quality fairly and automatically.
**Tasks:**
- Implement the C3 metric suite from the Decision Ledger + telemetry.
- Experiment harness: controller × scenario × failure-rate × seed matrix runner.
- Deterministic fault injection (agent failures, latency spikes) with fixed schedules.
- Scenario definitions S1–S3 with tunable variable factors.
- Results export to `experiments/results/` + analysis notebooks.
- Resolve DQ4 (minimal metric set); ADR.

**Definition of Done:**
- [ ] `harness run --controller X --scenario S --faults F --seed N` produces a results file.
- [ ] All C3 metrics computed automatically; documented in `research/evaluation-design.md`.
- [ ] Fault injection is deterministic (same seed → same fault schedule).
- [ ] A smoke matrix (all controllers × S1, 3 seeds) completes and produces a comparison table.
- [ ] ADR-004 (metric set) committed.

---

### WP5 — Experiments, analysis & findings (C4, RQ4, H1–H4)
**Objective:** produce the empirical evidence, honestly.
**Tasks:**
- Run the full experiment matrix (controllers × S1–S3 × failure rates × ≥5 seeds).
- Statistical analysis (effect sizes, confidence intervals, significance where appropriate).
- Test each hypothesis H1–H4 explicitly, including negative/overhead results.
- Generate publication figures + tables; archive raw results.

**Definition of Done:**
- [ ] Full matrix executed; raw results archived and checksummed.
- [ ] Each of H1–H4 has an explicit accept/reject/nuanced verdict with evidence.
- [ ] H4 (adaptation overhead) is quantified, not hidden.
- [ ] `research/findings.md` summarises results; figures regenerate from scripts.

---

### WP6 — Writing & dissemination (C1–C4)
**Objective:** communicate the work.
**Tasks:**
- Paper draft (intro, related work from gap-analysis, method, evaluation, results, limitations).
- Limitations section that states adaptation boundaries plainly.
- Public artifact: cleaned repo + reproduction guide.
- LinkedIn technical series (optional, derivative of paper sections).

**Definition of Done:**
- [ ] Complete first draft mapped to a specific target venue (§9).
- [ ] Reproduction guide lets a third party regenerate ≥1 headline figure.
- [ ] Repository public-ready (license, README, no secrets).

---

## 6. Timeline (gated, realistic)

The original README sketched 8 weeks. That is credible only for an **MVP / workshop result**, not
a top-venue submission. This plan uses a **~24-week** schedule with gates; an aggressive 8-week
track is preserved as the MVP milestone (M3).

| Milestone | Target | Gate criteria (exit) |
|-----------|--------|----------------------|
| **M0 — Kickoff** | Wk 0 | gap-analysis + architecture committed (done) |
| **M1 — Foundations ready** | Wk 2 | WP0 DoD met |
| **M2 — Formal model frozen** | Wk 5 | WP1 DoD met |
| **M3 — Baseline MVP** | Wk 8 | WP2 DoD met (← aggressive-track deliverable) |
| **M4 — Adaptive engine working** | Wk 13 | WP3 DoD met |
| **M5 — Evaluation harness ready** | Wk 16 | WP4 DoD met |
| **M6 — Results complete** | Wk 21 | WP5 DoD met |
| **M7 — Draft + artifact** | Wk 24 | WP6 DoD met → project DoD (§3) |

Gates are hard: a milestone is not "reached" until its WP DoD checklist is fully ticked.

---

## 7. Risk register

| ID | Risk | Likelihood | Impact | Mitigation |
|----|------|-----------|--------|------------|
| R1 | Adaptive ≈ well-tuned static (weak signal) | Med | High | Strong baselines + H4 framing turns this into a finding, not a failure |
| R2 | LLM non-determinism undermines reproducibility | High | Med | Fixed seeds, low temp, Ollama local model, selection-consistency probe |
| R3 | Scope creep (Kafka+Temporal+agents+engine) overruns | High | High | Gated MVP at M3; adaptivity stubbed until baseline solid |
| R4 | Meta-planner cost dominates (overhead) | Med | Med | Cheap mismatch guard + Governor budgets; measured as H4 |
| R5 | Reviewer says "not novel vs AdaptOrch/MASFly" | Med | High | Differentiation table (gap-analysis §7) + operational/coordination-quality deltas |
| R6 | Scenarios not enterprise-representative | Med | Med | Ground S1–S3 in a concrete domain (e.g., claims processing) with SLOs |
| R7 | Single-researcher bandwidth | High | Med | Prioritise C3+C4 (the strongest deltas) if time-constrained |

---

## 8. Reproducibility checklist (gate for D6)

- [ ] All service/image/library versions pinned.
- [ ] Random seeds fixed and recorded per run.
- [ ] Fault schedules deterministic and stored with results.
- [ ] Raw results + config archived and checksummed under `experiments/results/`.
- [ ] One command regenerates each headline figure from raw results.
- [ ] LLM prompt templates versioned alongside the Strategy Library.
- [ ] `README` reproduction guide verified on a clean checkout.

---

## 9. Publication mapping

| Output | Primary venue candidates | Backbone WPs |
|--------|--------------------------|--------------|
| Full system + empirical study | IEEE Trans. Services Computing; ICSOC 2026 | WP2–WP5 |
| Coordination-quality evaluation methodology | IEEE Intelligent Systems; AAMAS (eval track) | WP4 |
| Formal taxonomy + position | JAIR; AAMAS | WP1 |
| Negative/boundary result (when adaptation hurts) | workshop → journal extension | WP5 (H4) |

Recommended lead submission: **systems + empirical study** (largest, most defensible), with the
evaluation methodology factored out as a companion paper if results are strong.

---

## 10. Immediate next actions (this week)

1. WP0: scaffold repo tree + `docker-compose.yml` skeleton + Python project. (M1)
2. WP1: draft `research/taxonomy.md` and the first 4 Strategy Library templates.
3. Convert the open design questions (DQ1–DQ4) into tracked ADR stubs under `architecture/`.

---

*Living document. Update WP DoD checkboxes as work completes; record decisions as ADRs; keep the
timeline gates honest.*
