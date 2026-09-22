# Architecture

ActionFoundry is organized around one explicit abstraction: at every decision step, the system operates on a **set of candidate actions** rather than requiring a single model to directly emit the final control command.

![Architecture](assets/architecture.svg)

## Decision loop

### 1. Observation and task context

Inputs can include robot state, images, language instructions, environment state, history, and task-specific constraints. A context encoder converts these heterogeneous inputs into a representation usable by proposal and evaluation modules.

### 2. Candidate proposal

A proposal source produces a finite candidate set `C_t = {c_1, …, c_K}`. The source is intentionally replaceable: hand-authored motion primitives, trajectory sampling, classical planners, policy rollouts, retrieval, or learned proposal models.

A candidate should carry enough structure to be evaluated and executed: action/trajectory payload, horizon, coordinate frame, provenance, optional proposal confidence, and auxiliary metadata.

### 3. Constraints and scoring

Candidates may first pass through hard feasibility/safety filters. Remaining candidates receive scores such as task progress, geometric feasibility, collision margin, learned value/success probability, semantic preference, uncertainty, and execution cost.

The architecture distinguishes **hard constraints** from **soft preferences**. A learned score should not silently replace a safety invariant.

### 4. Selection

The selector maps the evaluated candidate set to a decision. Initially this can be deterministic (argmax, lexicographic constraints, weighted objectives). Later phases can study learned ranking, pairwise comparison, calibrated value functions, or set-aware selection.

Abstention/replan is a valid decision: if no candidate is acceptable, request a new proposal set rather than force an action.

### 5. Execution and feedback

The selected candidate is handed to the robot controller or simulator. Execution produces observations and outcome signals, closing the loop. Receding-horizon operation follows naturally: propose, evaluate, execute a bounded prefix, observe, and reconsider.

## Core interfaces

The first milestone should stabilize four conceptual contracts:

```text
Context
  observation
  robot_state
  task
  history
  constraints

Candidate
  id
  action_or_trajectory
  horizon
  frame
  provenance
  metadata

Evaluation
  candidate_id
  hard_valid
  scores
  uncertainty
  reasons
  latency

DecisionTrace
  context_ref
  candidates
  evaluations
  selected_candidate
  execution_result
  timing
```

These are conceptual schemas, not yet an API commitment.

## Why this decomposition?

Direct action generation makes failure modes difficult to separate: bad context, weak proposals, incorrect value estimation, constraint violations, and controller failures. Candidate-based selection exposes intermediate alternatives and creates experimentally separable modules.

The same candidate pool can be scored by heuristics, a learned value model, a multimodal judge, or an oracle. Conversely, the same selector can consume candidates from multiple proposal mechanisms.

## Evaluation axes

| Axis | Example metrics |
| --- | --- |
| Candidate quality | oracle-in-set success, coverage, diversity |
| Selection quality | top-1 success, regret vs. oracle, ranking metrics |
| Calibration | reliability / expected calibration error where applicable |
| Safety | invalid-selection rate, constraint violations |
| Closed-loop behavior | task success, recovery rate, steps/replans |
| Efficiency | proposal latency, scoring latency, end-to-end decision latency |
| Robustness | perturbations, distractors, candidate-set shift |

**Oracle-in-set performance is a key diagnostic.** If no good action exists in the candidate set, improving the selector cannot solve the failure. This separates proposal failures from selection failures.

## Staged implementation

**Phase 0 — Contract.** Define schemas, logging, reproducibility conventions, and a minimal environment adapter.

**Phase 1 — Harness.** Build deterministic candidate sources, constraint filters, heuristic scorers, oracle analysis, and closed-loop evaluation. The goal is a trustworthy measurement system, not a neural model.

**Phase 2 — Learned selection.** Add learned scorers/rankers while holding proposal sets fixed where possible. Compare pointwise value prediction, pairwise ranking, and set-aware selection.

**Phase 3 — Closed-loop adaptation.** Introduce replanning, temporal context, uncertainty-aware abstention, and execution feedback.

**Phase 4 — Extensions.** Study learned proposal generation, richer multimodal context, hierarchical decisions, and fast/slow decision pathways.
