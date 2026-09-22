# Coding-agent instructions

## Mission

Implement the v0.1 research specification incrementally. The goal is a reproducible test of candidate-based action selection, not a demonstration engineered to favor one model.

Read, in order: [decisions](docs/decisions.md), [architecture](docs/architecture.md), [contracts](docs/contracts.md), [benchmarks](docs/benchmarks.md), [experiments](docs/experiments.md), and [implementation](docs/implementation.md).

## First scope

Implement P0, then P1 Lift. Do not begin with a VLA integration, Jev API, world model, ROS deployment, distributed framework, or learned scorer. Stack follows a passing Lift harness. A working closed loop is part of P1.

## Non-negotiable invariants

- Runtime policies must not receive hidden simulator snapshots, future branch outcomes, test labels, or evaluator-only success flags.
- Candidates are compiled once per observation into immutable, timestamped plans. Scorers cannot modify them. Candidate IDs are opaque identifiers, not rank hints.
- Preserve frame, unit, controller, horizon, and gripper semantics. Never silently clip an out-of-contract command and score the unclipped version.
- A `HOLD` command is not a safety certificate. An empty candidate set, NaN score, timeout, or stale observation has an explicit bounded failure path.
- Closed-loop branches cannot generally share later states. Reuse initial states and exogenous perturbation schedules, not another method's future observation stream.
- Split by source episode before creating snapshots, candidate labels, augmentations, or prompts. Repeated decisions are not independent samples.
- A softmax over candidates is not the probability of task success. Human-selected actions do not establish that every unselected action fails.
- No paid APIs, external data uploads, unreviewed remote code, hardware motion, or automatic large downloads by default.
- No force-push, unrelated repository rewrites, fabricated citations, or invented successful tests.

## Delivery discipline

Use English documentation and comments. Keep the core CPU-testable and imports free of simulator/model side effects. Add type checks and tests with each interface. Keep each implementation commit coherent.

Every experiment report must identify the code commit, resolved config and lockfile hashes, environment/assets, initial-state manifest, seeds, observation regime, controller, execution prefix, hardware, costs, failures, and command. Missing results remain `not_run`, never zero.

Freeze dependency versions and source revisions only after a real local compatibility test. Do not claim that this documentation revision validated a simulator installation.

Change a settled decision only with a short amendment in `docs/decisions.md`, the reason, and affected experiments. Improve implementation details autonomously; do not silently change the research question, data split, or test gate.
