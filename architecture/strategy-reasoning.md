# Strategy Reasoning Engine — Core Component Design

**Project:** ContextFlow
**Author:** Mahboub Parhizkar, PhD
**Lab:** OrionAI
**Document:** `/architecture/strategy-reasoning.md`
**Status:** Draft v0.1 — June 2026
**Parent:** [`system-design.md`](system-design.md)

---

## 1. Role in the system

The Strategy Reasoning Engine (SRE) is the **PLAN/DECIDE** stage of the adaptation control loop
and the project's primary research artifact. It receives an interpreted `ContextSnapshot` plus a
trigger, and emits an `OrchestrationDirective` (a strategy expressed as data) — or vetoes the
change and keeps the current strategy.

Everything around the SRE is proven infrastructure. The novelty lives here, in four design
decisions: **strategy-as-data**, **bounded adaptation**, **auditable decisions**, and a
**deterministic fallback**.

---

## 2. Sub-components

### 2.1 Strategy Library
A versioned catalogue of coordination strategies. Each entry is a **template**, not free-form
text, so the planner *selects and parameterises* rather than inventing structure.

```yaml
# strategy_library/hierarchical_recover.yaml
strategy_id: hierarchical_recover
version: 1
topology: HIERARCHICAL
agents: [planning, execution, retrieval]
coordination:
  supervisor: planning
  workers: [execution, retrieval]
recovery_policy:
  on_agent_failure: reassign_to_peer
  max_retries: 2
applicability:                 # used for grounding + offline analysis
  complexity_band: [2, 3]
  failure_pressure: high
cost_profile: medium
```

Templates cover the canonical topologies — `SEQUENTIAL`, `PARALLEL`, `HIERARCHICAL`, `HYBRID` —
each optionally paired with a recovery policy. The library is the shared vocabulary that makes
decisions comparable (contribution C1/C2) and is small enough that selection stays well-posed
(open question DQ1).

### 2.2 LLM Meta-Planner
Maps `ContextSnapshot + trigger + candidate templates → a single strategy choice with rationale`.

- **Input grounding:** the prompt includes the *current* strategy, the trigger that fired, a
  compact snapshot, and the *applicable* library templates (pre-filtered by the interpreter) —
  never the whole library, to keep grounding tight (DQ3).
- **Output contract:** structured JSON `{chosen_strategy_id, parameters, rationale,
  confidence}` validated against the library; malformed output is rejected by the Governor.
- **Determinism aids:** low temperature, fixed seed where supported, and a canonical prompt
  template versioned alongside the library so results are reproducible.

> RQ3 is answered empirically here: how prompt content + library grounding affect *consistency*
> of selection. We measure selection stability across repeated runs of identical inputs.

### 2.3 Adaptation Governor
The safety and cost gate. The planner *proposes*; the Governor *disposes*. It enforces:

| Bound | Meaning | Effect on violation |
|-------|---------|---------------------|
| `max_replans` | Cap on strategy switches per task | Veto further switches |
| `cost_budget` | Cumulative LLM + agent cost | Veto; fall back to cheapest viable strategy |
| `latency_budget` | Remaining SLA time | Veto if reasoning would breach SLA |
| `min_switch_interval` | Anti-oscillation window | Veto switch inside window |
| `schema_valid` | Output well-formed and in-library | Reject → fallback |

The Governor guarantees **safety** (no unbounded replanning, no budget breach) and **liveness**
(a deterministic last-known-good fallback always exists). It is the architectural answer to H4:
adaptation can never become unboundedly expensive.

### 2.4 Decision Ledger
Append-only log of every decision, accepted or vetoed:

```json
{
  "task_id": "T-1042",
  "ts": 1718480000.0,
  "triggered_by": "FAILURE",
  "snapshot": {"complexity_band": 2, "latency_remaining_ms": 4200, "failures": 2},
  "candidates": ["sequential_basic", "hierarchical_recover"],
  "chosen": "hierarchical_recover",
  "rationale": "execution agent failed twice; reassign under supervisor",
  "governor": {"verdict": "accepted", "replan_index": 1},
  "outcome": {"latency_ms": 180, "cost": 0.012}
}
```

The ledger is the substrate for **coordination-quality metrics (C3)** and full **reproducibility**:
a run can be replayed and audited decision-by-decision.

---

## 3. Decision algorithm

```
on trigger(snapshot, current_strategy):
    if not mismatch_detector.warrants_adaptation(snapshot):   # cheap guard
        return keep(current_strategy)

    candidates = library.applicable(snapshot)                 # pre-filter
    proposal   = meta_planner.decide(snapshot, current_strategy, candidates)

    verdict = governor.evaluate(proposal, snapshot, history)
    if verdict.accepted:
        ledger.record(proposal, verdict, outcome="accepted")
        return directive_from(proposal)                       # → workflow replan signal
    else:
        ledger.record(proposal, verdict, outcome="vetoed")
        return keep(current_strategy)                          # safe default
```

The expensive LLM path is reached **only** past the cheap mismatch guard — the core lever for
keeping adaptation overhead bounded and measurable.

---

## 4. Coordination-quality metrics produced here (C3)

Derived directly from the Decision Ledger + execution telemetry:

| Metric | Definition | Tests |
|--------|------------|-------|
| Adaptation latency | Time from trigger to directive applied | H1, H4 |
| Strategy-switch correctness | Did the switch improve the targeted signal vs. counterfactual hold? | H3 |
| Agent utilisation | Useful agent invocations / total invocations | H2 |
| Recovery boundedness | Replans per task vs. cap; oscillation rate | H1, safety |
| Adaptation overhead | Extra cost+latency attributable to reasoning | H4 |
| Decision auditability | % decisions with complete, valid trace | reproducibility |

These are the metrics CLEAR and benchmark-centric adaptive systems do **not** report, and are the
measurement contribution of the project.

---

## 5. Interfaces

```python
class StrategyReasoningEngine:
    def __init__(self, library: StrategyLibrary, planner: MetaPlanner,
                 governor: AdaptationGovernor, ledger: DecisionLedger): ...

    def decide(self, task: TaskSpec, snapshot: ContextSnapshot,
               current: OrchestrationDirective,
               history: DecisionHistory) -> OrchestrationDirective:
        """Returns a new directive (accepted) or `current` (vetoed/no-trigger)."""
```

The SRE implements the `OrchestrationController` protocol (see system-design §4) as the
`AdaptiveController`, so it is directly comparable to `StaticController` and
`SupervisorController` under the evaluation harness.

---

## 6. Validation plan (mapped to RQs)

- **RQ1 (taxonomy):** ablate signal kinds; measure decision quality with/without each signal.
- **RQ2 (architecture):** verify durability (crash mid-replan → deterministic resume) and bound
  enforcement under stress.
- **RQ3 (algorithm):** measure selection consistency vs. prompt/grounding variants; report
  stability across repeated identical inputs.
- **RQ4 (empirical):** run `AdaptiveController` vs. baselines across S1–S3 with fault injection;
  report the C3 metric suite, including the H4 overhead result honestly.

---

## 7. Open questions

Tracked as ADRs as they are resolved (see system-design §10): library granularity (DQ1), trigger
threshold strategy (DQ2), prompt context budget (DQ3), minimal metric set (DQ4).
