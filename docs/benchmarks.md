# Benchmark specification

These are ActionFoundry protocol choices, not reported results or upstream benchmark defaults. Source links are in [references](references.md).

## B0 — Contract fixture

Implement `MockPlanarReach` using NumPy only: a point end effector in a bounded 2D workspace, a target, deterministic dynamics, and optional axis-aligned obstacles. Support save/restore, known costs, empty valid sets, forced timeouts, and exact replay. This is an engineering fixture, not evidence about manipulation.

## B1 — Primary manipulation slice

Use **robosuite Panda Lift**, followed by **Panda Stack**, state observations, fixed robot embodiment, and fixed controller configuration. robosuite documents both tasks, object/proprioceptive observations, and a composite-controller interface [R01](references.md#r01) / [R02](references.md#r02).

Freeze the following before test collection:

| Setting | v0.1 decision |
| --- | --- |
| Observation regime | `state`; declared robot pose/joints/gripper, object poses/sizes, permitted contact cues, task goal |
| Controller | BASIC composite controller with arm OSC_POSE semantics verified by adapter tests |
| Environment control rate | 20 Hz; physics timestep logged separately |
| Planned horizon / executed prefix | 8 / 4 controller ticks |
| Episode budget | 500 controller ticks; nominal 25 simulated seconds |
| Candidate pool | Default K=16; K=8/32 for a separate support ablation |
| Initial orientation | Fixed task-appropriate gripper orientation; orientation search deferred |
| Rendering | Off for state experiments; optional audit videos outside latency timings |
| API/model cost | None in P0/P1 |

Use native task success for the primary `ever_success` flag over the episode, and also report final-state success and maximum consecutive success ticks. Do not let a model's `DONE` token establish success. Early abort counts as unsuccessful unless success was already recorded; report both ever and final outcomes. Do not drop failed rollouts. Keep the full tick budget unless environment termination, explicit abort, or infrastructure error intervenes.

The 500-tick protocol is a local study, not a claim to reproduce an upstream leaderboard. Thresholds for the heuristic, workspace envelope, controller gains, and gripper mapping are selected only on pilot/development states, recorded, and frozen. Do not assume that a normalized controller action represents meters or that collision-free kinematics establishes safe contact dynamics.

### Initial candidate generator

Compile all candidates to eight-tick anchored pose plans. Start with:

- Twelve translation candidates: each of the six signed base axes at 1 cm and 3 cm total planned displacement, fixed orientation, gripper `KEEP`.
- One pose-maintenance candidate and two grip-only candidates (`OPEN`, `CLOSE`).
- One task-relative proposal toward the next geometric target from a documented state-based phase machine.

This yields up to 16 distinct plans; deduplication may reduce actual K and must be logged. The phase machine uses only the declared state track. Lift phases are approach-above, descend, close, and lift. Stack additionally includes transfer-above, lower, release, and retreat. Phase changes use visible geometry, gripper/contact observations, and bounded history—not future rewards or the evaluator's success flag. Candidate generation and a direct-scripted comparator share the exact same phase logic and thresholds.

A 32-candidate reservoir appends 16 seeded local target perturbations within the same displacement envelope. K=8 selects a seeded subset of the 16-pool while retaining maintenance and grip commands. Preserve nesting and record exact pool membership. Permute scorer presentation independently of candidate generation.

The operational checker initially verifies finite values, identity/freshness, units, workspace bounds, per-tick displacement/rotation bounds, and declared gripper/controller support. Full collision/contact feasibility is not claimed unless a separately tested checker is enabled. Log unsupported checks as unknown. The admission profile states which checks are mandatory; unknown mandatory checks reject the plan.

### Representation study

For the matched-support comparison, encode the **same compiled physical plans** as semantic incremental commands, anchored relative targets, and explicit pose sequences. Match H, executed prefix, controller, observations, and gripper behavior. Verify plan digests agree after round-trip compilation. Complex trajectory support is a later, separately labeled expansion; do not pretend every arbitrary chunk can be losslessly represented by one local-move token.

## B2 — Visual transfer validation

After B1, implement **LIBERO-Spatial** as a distinct worker environment. Use the suite's task definitions and supplied initial-state interface [R03](references.md#r03). Audit the actual pinned task list, dataset hashes, controller scaling, horizons, and reset behavior before starting. Do not assume robosuite 1.5 and LIBERO's original dependency stack can coexist; the upstream LIBERO README describes an older Python environment.

First run 10 smoke episodes on each task in the pinned suite. For a confirmatory study use all available official initial states under the selected evaluation recipe, not a hand-picked subset. Record task names, state IDs, and model-specific evaluation conventions. Do not claim a standard LIBERO score when using a custom horizon or split.

The first visual proposal source should be an existing policy with a validated adapter; compare its default action/chunk against selection from its own sampled candidates. Use the same weights, action normalization, context, and execution prefix. A checkpoint is pinned only after a local reproduction of its baseline succeeds. No VLA checkpoint or GPU is required to complete P0/P1.

Report three regimes separately:

| Regime | Information available to the whole pipeline |
| --- | --- |
| `state` | Explicit simulator/object state is permitted |
| `vision_assisted` | At least one component uses exact simulator geometry or privileged proposal features |
| `vision_only` | Runtime uses declared camera/proprioceptive/task observations only; no hidden object poses |

A reranker using vision does not make a privileged proposer vision-only. A pretrained model may already have seen LIBERO data. Record known training overlap; “held out from this scorer's training” is not “unseen by the foundation model.”

## Split and perturbation protocol

For B1, use per-task initial-state seeds: training/development 0–999, validation 1000–1199, calibration 1200–1399, held-out ID test 2000–2299, and OOD test 3000–3299. Ranges are inclusive. Pilot tuning uses 0–49 only. Persist initialized states and hashes; seeds alone are insufficient for reproducing resets after dependency changes.

All derived snapshots, branch labels, augmentations, and near-duplicate frames inherit the parent episode split. Test seeds never enter scorer fitting, temperature fitting, prompting, candidate design, or threshold selection. Train on both B1 tasks for the first controlled selector study; cross-task transfer is a different protocol.

Predeclare perturbation families independently: object-placement shift within validated reachable bounds; sensor noise/occlusion; dynamics or friction shift; exogenous object displacement; candidate-source shift. Freeze magnitudes on development states and store exact schedules before test use. Do not pool impossible resets with selector failures. Report invalid-reset counts and reasons without silently resampling favorable scenes.

Visual background/viewpoint shifts belong to B2 or a visual B1 extension, not state-only generalization claims. Cross-embodiment transfer, tactile input, deformable objects, and real hardware are deferred.
