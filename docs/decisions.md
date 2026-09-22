# Design decisions

Status: accepted engineering direction for v0.1; scientific hypotheses remain unproven. Date: September 22, 2026.

## D01 — Workbench, not a Jev clone

The repository compares explicit candidate selection with direct action generation. No proprietary architecture, RLCD recipe, calibration property, or robotics capability is assumed to have been reproduced. Jev is an optional future external scorer; a local typed scorer is named for its implemented loss and architecture.

Why: Q-based selection, implicit policies, and value-guided reranking already exist. The reference map makes these precedents explicit. Reopen the project thesis only after a documented experiment, not after discovering another appealing paper.

## D02 — Small vertical slice first

P0 uses a deterministic planar mock. P1 uses robosuite Panda Lift followed by Stack with state observations. P3 uses LIBERO-Spatial for a separate visual-transfer study. Do not implement multiple simulator adapters concurrently.

Why: perception, action generation, and simulator integration should not all change in the first experiment. State-assisted success is not evidence of general visual manipulation. If local compatibility fails, document the failure and pin a compatible worker environment; do not silently substitute a different benchmark.

## D03 — One canonical execution plan, multiple representations

Compile semantic moves, anchored relative targets, and later trajectory proposals into a timestamped sequence of base-frame end-effector poses with gripper modes. Different representations may express the same physical plan. Their execution must be identical in the representation-only ablation.

Why: changing step size, horizon, or controller while changing action tokens does not isolate the representation. Joint-position policies require a separately declared adapter; do not convert them into end-effector commands without a validation study.

## D04 — Local synchronous baseline

Start with one decision at a time, one environment process, and bounded-prefix execution. A 20 Hz environment control interface and a four-tick execution prefix imply a nominal 5 Hz decision schedule, not an achieved real-time guarantee. No ROS or asynchronous scheduling in P0/P1.

Why: determinism and attribution are initially more valuable than throughput. Add concurrency only after traces, budgets, stale-result rejection, and state restoration pass.

## D05 — Separate execution permissions from scores

Structural validity and configured operational constraints are checked outside learned models. Task progress and predicted success are soft estimates. Unknown feasibility is represented explicitly, not converted into `safe=true`. Simulation diagnostics must not be described as hardware safety validation.

## D06 — No training in P1

Use random, deterministic geometric/phase, direct-scripted, and privileged diagnostic baselines first. P2 adds a small pointwise scorer and a direct behavior-cloning baseline before set-aware scoring, VLM judging, or large-scale training.

Why: a failed proposal set cannot be repaired by a better selector. A selector bottleneck is established on development data before allocating training compute.

## D07 — Privilege is a system-level property

Report `state`, `vision_assisted`, `vision_only`, or `oracle_diagnostic` for the entire pipeline. A vision scorer fed candidates generated from exact object poses is `vision_assisted`, not vision-only. A frozen pretrained checkpoint evaluated on familiar benchmark tasks is not called zero-shot without its training provenance.

## D08 — Two clocks and two evaluations

Measure simulator-control time separately from wall-clock time. Offline fixed-pool ranking and fresh closed-loop rollouts answer different questions. Paused simulation can measure algorithmic behavior but cannot demonstrate real-time control.

## D09 — WAM is a parallel research axis

A simulator and a learned world/action model have different jobs. Simulator branches provide privileged counterfactual labels after snapshot/restore validation. A WAM may provide action proposals, world/action representations, or candidate-conditioned predictions when its architecture actually supports that contract. OpenWAM is the first integration target because it exposes modular WAM architectures, training/fine-tuning, released checkpoints, and manipulation benchmark adapters [R27](references.md#r27).

P0/P1 do not depend on a WAM. After the basic harness is trustworthy, WAM experiments proceed in increasing commitment: frozen checkpoint as proposer/representation source; small decision head on frozen features; candidate-conditioned prediction where supported; only then optional joint WAM + decision training. Predictive fidelity and decision utility are reported separately.

## D10 — Simulator backend is an interface

MuJoCo/robosuite is the first mechanism-debugging backend, not a project-wide simulator commitment. LIBERO is the first visual-transfer benchmark, RoboTwin 2.0 is the preferred richer WAM-oriented follow-up, and CALVIN/ManiSkill/Isaac Lab remain conditional extensions. Each adapter declares snapshot/restore, determinism, observation privilege, controller, and rendering capabilities; unsupported branchability is never emulated silently.

## D11 — Honest handoff

This commit provides specifications and examples. It does not provide a trained model, a completed experiment, or a recovered copy of the separate Deep Research report. References were checked at the primary-source level described in the source map; reported paper results were not reproduced.

## Decisions left to local measurement

Exact compatible dependency pins, controller gains, operational workspace bounds, and scorer thresholds must be frozen during a development-only pilot. The protocol fixes their selection process and logging, not untested values. Changes after test evaluation require a versioned protocol amendment and a new holdout.
