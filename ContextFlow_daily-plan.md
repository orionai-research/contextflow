# ContextFlow — Daily Execution Plan

**Project:** ContextFlow | **Lab:** OrionAI
**Start:** Tuesday, 16 June 2026
**Authoritative plan:** [`research/project-plan.md`](research/project-plan.md) (work packages WP0–WP6, milestones M0–M7, Definition of Done)
**Daily structure:** 8 hours — Learning (2h) · Development (4h) · Documentation (1h) · Visibility (1h)

> **Why this isn't 8 weeks.** The credible schedule is ~24 weeks, gated by Definition of Done (see project plan). This file gives **day-by-day detail for the foundation phase (Weeks 1–3, WP0→WP1)** because that's near enough to plan precisely, then **weekly blocks for Weeks 4–24**. Expand each later block into daily detail as its milestone approaches — planning day-by-day six months out is fiction.

> **Honesty rule carried from the research:** we are not the first to adapt orchestration at runtime. Reading and writing should engage AdaptOrch, MASFly, CARD, AOrchestra, self-healing orchestrators, and CLEAR — and our delta (formal model, operational grounding, bounded adaptation, coordination-quality metrics).

---

## Milestone map (targets)

| Milestone | Target week | Work package | Exit gate (DoD) |
|-----------|-------------|--------------|-----------------|
| M1 — Foundations ready | Wk 2 (by Jun 26) | WP0 | `docker compose up` healthy; lint/test green |
| M2 — Formal model frozen | Wk 5 (by Jul 17) | WP1 | taxonomy + ≥4 strategy templates + triggers |
| M3 — Baseline MVP | Wk 8 (by Aug 7) | WP2 | static + supervisor controllers run S1; durable |
| M4 — Adaptive engine working | Wk 13 (by Sep 11) | WP3 | bounded adaptive controller runs S1–S3 |
| M5 — Evaluation harness ready | Wk 16 (by Oct 2) | WP4 | coordination-quality metrics automated |
| M6 — Results complete | Wk 21 (by Nov 6) | WP5 | full matrix; H1–H4 verdicts incl. H4 |
| M7 — Draft + artifact | Wk 24 (by Nov 27) | WP6 | paper draft + reproducible artifact |

---

# PHASE 1 — FOUNDATION (Weeks 1–3) · daily detail

## Week 1 — WP0 scaffolding + WP1 kickoff (Tue 16 – Fri 19 Jun)

### Tuesday 16 Jun — Repo + environment skeleton
- Learning (2h): Re-read your own `gap-analysis.md` §2.5 + skim AdaptOrch and MASFly abstracts/intros (know your neighbours).
- Dev (4h): Scaffold repo tree (per README structure); init Python project (deps, ruff/black, pytest, pre-commit); `Makefile`/CLI stubs.
- Docs (1h): `research/literature-notes/2026-adaptive-orchestration.md` — start annotated notes (AdaptOrch, MASFly).
- Visibility (1h): Update LinkedIn profile (headline/about/featured); finalise **Post 1** draft (do not publish yet).

### Wednesday 17 Jun — Docker Compose base
- Learning (2h): Temporal core concepts (workflows, activities, signals); Temporal vs Airflow/Camunda.
- Dev (4h): `docker-compose.yml` skeleton — FastAPI gateway health endpoint + Temporal + Temporal UI up and healthy.
- Docs (1h): ADR-template; ADR-000 "stack choices" (Temporal over Camunda; drop Celery).
- Visibility (1h): **Publish Post 1 — Project announcement.** Reply to comments.

### Thursday 18 Jun — Messaging + vector infra
- Learning (2h): Kafka for event-driven AI signal propagation; Qdrant basics.
- Dev (4h): Add Kafka + Zookeeper + Qdrant + Prometheus + Grafana to compose; verify all services healthy; one test Kafka produce/consume.
- Docs (1h): Update `architecture/system-design.md` §9 deployment topology with anything learned.
- Visibility (1h): Engage — comment substantively on 5 relevant posts (tag adaptive-orchestration authors where genuine).

### Friday 19 Jun — WP0 close + LangGraph first contact
- Learning (2h): LangGraph state machines + supervisor/router pattern (this becomes a baseline).
- Dev (4h): Minimal 2-node LangGraph (planning → execution) running inside the stack; `make test`/`make lint` green.
- Docs (1h): **WP0 DoD checklist** in project plan — tick what's done; note gaps. Draft **Post 2**.
- Visibility (1h): **Publish Post 2 — Research question explainer.**

**Weekend:** finalise literature notes; confirm M1 gate (WP0 DoD) nearly met; plan Week 2.

---

## Week 2 — WP0 finish → WP1 formal model (Mon 22 – Fri 26 Jun)

### Monday 22 Jun — Context-signal taxonomy (C1)
- Learning (2h): MAPE-K / autonomic computing; revisit how AdaptOrch/CARD define their signals.
- Dev (4h): Draft data contracts in code — `TaskSpec`, `ContextSignal`, `ContextSnapshot` (from system-design §5).
- Docs (1h): Start `research/taxonomy.md` — enumerate signal kinds (complexity, agent health, latency, failure, cost, result quality).
- Visibility (1h): Draft **Post 3 — the honest research gap.**

### Tuesday 23 Jun — Strategy-as-data
- Learning (2h): Coordination patterns (sequential/parallel/hierarchical/hybrid); how MASFly/CARD represent topologies.
- Dev (4h): Define strategy template schema; author first 2 templates under `strategy_library/` + schema validation.
- Docs (1h): `architecture/adr/ADR-001-library-granularity.md` (DQ1 initial position).
- Visibility (1h): **Publish Post 3.**

### Wednesday 24 Jun — Triggers / mismatch detection
- Learning (2h): Failure handling (Saga/compensation); threshold vs. learned triggers.
- Dev (4h): Implement `signal_interpreter` aggregation + first trigger conditions; unit tests on synthetic snapshots.
- Docs (1h): Document trigger conditions in `taxonomy.md`; note DQ2 for later ADR.
- Visibility (1h): Engage — respond to Post 3 comments.

### Thursday 25 Jun — Hypotheses + evaluation skeleton
- Learning (2h): CLEAR (arXiv:2511.14136) in depth — what it measures and what it omits (coordination path).
- Dev (4h): 2 more strategy templates (now ≥4); wire interpreter → (stub) controller decision.
- Docs (1h): `research/hypotheses.md` — formalise H1–H4 (incl. H4 overhead); start `research/evaluation-design.md` metric list.
- Visibility (1h): Draft **Post 4 — architecture diagram** (export closed-loop diagram).

### Friday 26 Jun — M1 gate + freeze taxonomy v0
- Learning (2h): Free reading — fill any gaps from the week.
- Dev (4h): Integration check — signals flow agent → observer → Kafka → interpreter; durability smoke test (kill worker, resume).
- Docs (1h): **Tick WP0 DoD (M1)**; freeze `taxonomy.md` v0; update README roadmap.
- Visibility (1h): **Publish Post 4 — architecture diagram.**

**Weekend:** consolidate; if M1 gate not fully met, carry remainder into Mon. Do **not** start WP2 until M1 DoD is green.

---

## Week 3 — WP1 deepen (Mon 29 Jun – Fri 3 Jul)

- **Mon 29:** Meta-planner I/O contract design (no LLM yet) + grounding plan (ADR-003 stub, DQ3). Docs: strategy-reasoning §2.2 refinements.
- **Tue 30:** Decision Ledger schema + append-only store; unit tests. Docs: ledger examples.
- **Wed 1 Jul:** Adaptation Governor bounds spec (latency/cost/frequency, anti-oscillation, fallback) — interfaces + tests with forced violations. ADR-002 (triggers, DQ2).
- **Thu 2:** Controller `Protocol` + `StaticController` (expert-tuned fixed directive). Wire into runtime end-to-end.
- **Fri 3:** **WP1 DoD review** (taxonomy, ≥4 templates, triggers tested, ADR-001 committed). Update project plan. Visibility: milestone post **M2-in-progress** (formal model taking shape).

---

# PHASE 2 — BASELINES & CORE (Weeks 4–13) · weekly blocks

## Week 4 (Jul 6–10) — WP1 finish → WP2 start
- Finish/freeze formal model toward **M2** (target Wk 5).
- Implement Context Observer fully (all signal kinds → Kafka).
- Begin `SupervisorController` (LLM router/handoff per step) as the strong baseline.

## Week 5 (Jul 13–17) — M2 gate: formal model frozen
- Complete WP1 DoD; tag `model-v1`.
- Three agents behind uniform interface: planning, execution, retrieval (Qdrant RAG).
- Visibility: **milestone post — formal model + taxonomy published.**

## Week 6 (Jul 20–24) — WP2 orchestration runtime
- LangGraph runtime compiles `OrchestrationDirective`; `StaticController` runs S1 end-to-end.
- Prometheus instrumentation: latency, invocations, failures.

## Week 7 (Jul 27–31) — WP2 durability + supervisor baseline
- Temporal: replan-as-signal plumbing (even with adaptive stubbed); deterministic resume test.
- `SupervisorController` runs S1; Grafana dashboard of coordination metrics.

## Week 8 (Aug 3–7) — M3 gate: Baseline MVP
- Both baselines run S1 to completion; durability test passes; signals visible end-to-end.
- **Tick WP2 DoD.** Visibility: **milestone post — baseline MVP + first dashboard** (the "you can't claim improvement without a baseline" post).

## Weeks 9–10 (Aug 10–21) — WP3 meta-planner
- LLM Meta-Planner with structured output + library grounding; low-temp/seed determinism.
- Selection-consistency probe (repeated identical inputs → stability score) — RQ3.
- ADR-003 (prompt context budget, DQ3).

## Weeks 11–12 (Aug 24 – Sep 4) — WP3 governor + ledger + adaptive controller
- Adaptation Governor enforcing all bounds; forced-violation tests assert veto/fallback.
- Decision Ledger 100% trace coverage; `AdaptiveController` runs S1–S3.

## Week 13 (Sep 7–11) — M4 gate: Adaptive engine working
- Bounded adaptive controller completes S1–S3; governor + fallback verified.
- **Tick WP3 DoD.** Visibility: **milestone post — first runtime strategy switch demo** (short GIF/diagram), emphasise *bounded* + *auditable*.

---

# PHASE 3 — EVALUATION (Weeks 14–21) · weekly blocks

## Weeks 14–15 (Sep 14–25) — WP4 metrics + harness
- Implement C3 metric suite from Decision Ledger + telemetry (incl. adaptation overhead for H4).
- Experiment harness: controller × scenario × failure-rate × seed runner; deterministic fault injection.

## Week 16 (Sep 28 – Oct 2) — M5 gate: harness ready
- Scenarios S1–S3 finalised (grounded in claims-processing-style domain w/ SLOs).
- Smoke matrix (all controllers × S1 × 3 seeds) → comparison table; ADR-004 (metric set).
- **Tick WP4 DoD.** Visibility: **milestone post — how I measure coordination quality (beyond CLEAR).**

## Weeks 17–19 (Oct 5–23) — WP5 full experiments
- Run full matrix (controllers × S1–S3 × failure rates × ≥5 seeds); archive + checksum raw results.
- Statistical analysis (effect sizes, CIs); begin figures.

## Weeks 20–21 (Oct 26 – Nov 6) — WP5 findings + M6 gate
- Explicit H1–H4 verdicts, **including the H4 overhead/boundary result — reported, not hidden.**
- `research/findings.md`; figures regenerate from scripts.
- **Tick WP5 DoD.** Visibility: **milestone post — key findings teaser (incl. where adaptation didn't help).**

---

# PHASE 4 — WRITING & DISSEMINATION (Weeks 22–24) · weekly blocks

## Week 22 (Nov 9–13) — Draft I
- Paper: Abstract, Introduction (acknowledge 2025–2026 prior art), Related Work (from gap-analysis §2.5), Problem Formulation.

## Week 23 (Nov 16–20) — Draft II
- Paper: Architecture, Evaluation Methodology, Results; generate final figures/tables.

## Week 24 (Nov 23–27) — M7 gate: draft + artifact
- Paper: Discussion (when adaptation is/ isn't worth it), Limitations, Conclusion, references (30–40, incl. all 2026 papers).
- Reproducibility: clean checkout regenerates ≥1 headline figure; repo public-ready (license, no secrets).
- **Tick WP6 DoD → project DoD.** Visibility: **milestone post — preprint live + OrionAI lab.** arXiv submission.

---

## Daily discipline rules

1. **Start every morning with reading, not coding.** Research identity is built on literature fluency — and on knowing the 2025–2026 adaptive-orchestration work cold.
2. **Respect the gates.** Do not start a work package before the previous milestone's Definition of Done is green. Half-finished layers compound.
3. **Commit to GitHub daily** — including docs and ADRs. The graph and the decision trail both matter.
4. **Never skip the docs block.** README, gap analysis, taxonomy, and ADRs are what postdoc supervisors read first.
5. **The paper is the goal, and honesty is the method.** Every experiment serves a hypothesis — including H4. A clean negative result is a publishable result.
6. **Post per milestone, not per day.** After the launch sequence (Posts 1–4), one substantive milestone post beats forced weekly content.
