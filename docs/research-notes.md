# Research Notes

## Working hypothesis

A useful way to structure robot decision making is to separate **proposal** from **selection**. Instead of requiring a policy to directly generate the final action, the system can expose a bounded set of plausible actions or short-horizon trajectories and explicitly evaluate them before execution.

This repository treats that decomposition as the research object rather than committing to a particular model family.

## Synthesis

### The research question is larger than any one reference method

The motivating ideas touch vision-language-action policies, value functions and critics, model-predictive control, trajectory sampling, learning-to-rank, multimodal judging, and fast/slow decision systems. None of these labels should define the repository prematurely.

The durable question is whether an explicit **candidate → evaluate → select → execute** interface creates better controllability, diagnostics, and learning opportunities for embodied agents.

### Candidate quality and selector quality must be disentangled

A selector cannot recover an action that was never proposed. Experiments should report both whether a successful/near-optimal candidate existed in the set and whether the selector chose it. This motivates oracle-in-set evaluation and controlled candidate pools.

### Start with representations and an evaluation harness

The highest-leverage early work is not training a large judge. It is defining candidate schemas and coordinate conventions, deterministic proposal mechanisms, feasibility/safety filters, replayable decision traces, oracle and heuristic baselines, latency accounting, and closed-loop metrics.

### Non-learned baselines are part of the research

Geometry, collision checks, kinematic feasibility, goal distance, trajectory cost, and classical planning can be strong signals. They establish what information a learned evaluator actually adds.

### Learned selection admits multiple formulations

**Pointwise scoring.** Predict a scalar value/success score for each candidate. Simple and cacheable, but calibration and cross-set comparability matter.

**Pairwise ranking.** Compare two candidates. This may ease supervision but can be expensive at inference and can create non-transitive preferences.

**Set-aware selection.** Score/select candidates jointly, allowing relative context and diversity to matter. More expressive, but coupled to candidate-set size/distribution.

**Critic/value estimation.** Estimate downstream return or success conditioned on state/task and candidate. Natural for control, but target construction matters.

**Multimodal judging.** Use visual/language reasoning for semantic compatibility that geometry alone cannot capture. Latency, calibration, and grounding must be measured.

These are hypotheses to compare, not a predetermined progression.

### Closed-loop behavior matters more than offline ranking alone

A candidate can look locally optimal yet lead to a brittle future state. The evaluation stack therefore needs both offline diagnostics and closed-loop rollouts. Receding-horizon selection offers a practical bridge: select a short action/trajectory, execute a bounded portion, observe, and reconsider.

### Fast and slow pathways are an architectural hypothesis

A longer-term direction is a fast selector for routine, high-confidence decisions combined with a slower proposal/reasoning/replanning path when confidence is low, constraints conflict, or the candidate set is inadequate.

The engineering question is operational: **when can a cheap selector safely act, and when should the system spend more computation or regenerate candidates?**

## Key experimental questions

1. How does performance scale with candidate-set size and diversity?
2. Which action representation is easiest to evaluate while remaining executable?
3. How much does learned scoring improve over geometry/constraint/task heuristics?
4. Is ranking easier or more transferable than direct action generation?
5. Can a selector trained on one proposal distribution generalize to another?
6. How should uncertainty trigger abstention or replanning?
7. What is the latency/quality frontier for fast versus deliberative selection?
8. Which failures originate in context, proposal, scoring, selection, or execution?

## Proposed first benchmark slice

1. Pick one simulator/task family with deterministic replay.
2. Define one candidate representation, preferably short-horizon trajectories or parameterized motion primitives.
3. Generate candidate sets with at least one deterministic sampler/planner.
4. Implement hard validity checks and transparent heuristic scores.
5. Compute oracle-in-set performance.
6. Log complete decision traces.
7. Run closed-loop selection with deterministic baselines.
8. Only then add the first learned scorer.

The first learned experiment should hold the candidate generator fixed. This gives a clean answer to whether learning improves **selection** before proposal learning becomes another variable.

## Failure taxonomy

- **Context failure:** relevant state/task information is missing or incorrectly encoded.
- **Proposal failure:** no acceptable candidate was generated.
- **Constraint failure:** an invalid candidate was allowed through.
- **Scoring failure:** candidate utilities were estimated incorrectly.
- **Selection failure:** the decision rule chose poorly despite useful scores.
- **Execution failure:** the chosen candidate could not be realized as expected.
- **Recovery failure:** the closed-loop system failed to detect or correct a bad transition.

## Near-term deliverables

Optimize initially for experimental clarity rather than breadth: typed schemas, environment adapter(s), proposal interface, evaluator interface, trace format, deterministic baseline implementations, and a minimal benchmark runner.

Once those exist, the research can branch into learned ranking/value models, multimodal judging, proposal learning, and fast/slow control without rewriting the experimental foundation.
