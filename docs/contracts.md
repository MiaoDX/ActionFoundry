# Runtime and data contracts

Normative v0.1 design. `MUST` describes a requirement for the future implementation. The names below are to be implemented as typed Python structures and explicit JSON encodings; they are not APIs currently available in this repository.

## 1. Coordinates and time

Use SI units: meters, seconds, radians. Internal rotations are unit quaternions in `xyzw` order; sign canonicalization and shortest-path interpolation must be tested. JSON numbers must be finite. Internal control arithmetic uses float64 until an adapter intentionally converts it.

`T_B_E` maps end-effector coordinates into the fixed robot-base frame. An anchored relative target `T_E0_Ek` is composed as:

`T_B_Ek = T_B_E0 @ T_E0_Ek`.

Every future waypoint in a relative chunk uses the same initial anchor `E0`. Sequential body increments instead compose as `T_B_E(k+1) = T_B_Ek @ delta_T_k`. Base-frame translation increments use base axes and are not interchangeable with either convention. Do not add Euler angles to compose orientation.

Allow initial input frames `robot_base` and `ee_anchor`; an object-relative proposal must name the object observation and resolve its transform at proposal time. Camera/left/right semantics require an explicit calibrated transform or a declared interpreter convention. No implicit frame inference from token names.

Gripper modes are `OPEN`, `CLOSE`, and `KEEP`. They are categorical commands, not assumed force or width values. Each robot adapter declares and tests its mapping; holding a pose does not imply preserving grip force. Controller-normalized action vectors are an adapter detail, never SI coordinates.

Use integer control ticks and `dt_s` for relative plan times. Log `sim_time_s`, UTC event time, and monotonic wall-clock durations separately.

## 2. Core records

| Record | Required content |
| --- | --- |
| `Context` | `schema_version`, `episode_id`, `decision_id`, `observation_id`, time, task/instruction, allowed observations, robot state, bounded history, observation regime |
| `CandidateSpec` | Opaque ID, kind, typed payload, frame and anchor, proposer revision, proposal seed, generation index |
| `ExecutablePlan` | Candidate ID, anchor observation ID, controller contract hash, `dt_s`, planned/executed step counts, canonical waypoints, gripper modes, plan digest |
| `ConstraintResult` | Candidate ID, each check's status `pass/fail/unknown`, thresholds, measured margins, method, explicit admission decision |
| `Evaluation` | Candidate ID, utility, score kind, optional success probability and its target, optional uncertainty, scorer revision, timing, status |
| `Decision` | `execute/replan/abort/done_request`, nullable candidate ID, reason code, observation ID, decision budget |
| `ExecutionResult` | Requested and completed ticks, realized state references, termination/truncation/error, observed check failures, timing |
| `DecisionTrace` | Context reference, full pool, compiled plans, masks/evaluations, decision, actual execution, configuration and artifact hashes |

Variable-length candidate sets are supported. Duplicate physical plans are deduplicated by digest before scoring while preserving all provenance references. A scorer MUST return one status for every admitted candidate, including errors. Missing or NaN utility cannot win selection.

A pose-plan waypoint is `{tick, position_m: [3], quaternion_xyzw: [4], gripper}`. Plan waypoints start at tick 1; tick 0 is the anchor. There must be a target for every planned controller tick in the initial compiler. Interpolation happens in the compiler, not separately in each scorer or executor.

Initial spec kinds: `local_move`, `relative_pose_target`, and `ee_pose_chunk`. `local_move` includes a direction/interpreter ID, magnitude, duration, and gripper mode; `relative_pose_target` includes a transform and duration; `ee_pose_chunk` includes the full sequence. General skill programs and joint-space chunks are extensions, not untyped payload escape hatches.

## 3. Interface signatures

```python
# Signatures to implement, not an importable package in this revision.
class Proposer(Protocol):
    def propose(self, context: Context, budget: ProposalBudget,
                rng: Generator) -> Sequence[CandidateSpec]: ...

class Compiler(Protocol):
    def compile(self, context: Context, candidate: CandidateSpec,
                control: ControlContract) -> ExecutablePlan: ...

class Scorer(Protocol):
    def evaluate(self, context: Context, plans: Sequence[ExecutablePlan],
                 features: Optional[PredictionBatch]) -> Sequence[Evaluation]: ...

class Selector(Protocol):
    def select(self, context: Context, checks: Sequence[ConstraintResult],
               evaluations: Sequence[Evaluation]) -> Decision: ...

class Executor(Protocol):
    def execute(self, observation_id: str,
                plan: ExecutablePlan) -> ExecutionResult: ...
```

Implement analogous `EnvironmentAdapter`, `ConstraintChecker`, `DirectPolicy`, and offline-only `SnapshotAdapter`/`BranchEvaluator` protocols. Use explicit constructor injection; no global mutable environment or model registry. RNG streams for environment, proposals, selection, and training are independent.

## 4. Score semantics

`utility` is a ranking quantity with larger values preferred. An energy model declares the conversion `utility = -energy`; a cost model declares `utility = -cost`. Units and normalization belong to the scorer manifest. Arbitrary scorer utilities are not comparable across methods.

`choice_probability` is conditional on a particular candidate set, prompt, and normalization. It changes when alternatives change. It is not physical success probability, and its margin/entropy is not automatically calibrated uncertainty.

`success_probability` is an independent estimate for each candidate with a named target, execution prefix, continuation policy, time horizon, and disturbance distribution. Multiple candidates may all succeed. Do not force these probabilities to sum to one.

`unknown` is distinct from zero probability. A missing uncertainty estimate is `null`, not a confident prediction. Selection thresholds are learned/tuned on development or calibration splits only. Store the calibration artifact hash.

## 5. Snapshot contract

Snapshotting is an offline diagnostic capability, not part of `Context`. Restore more than `qpos/qvel`: physics integration state and time, actuator/control state, mocap/user data as relevant, task counters, controller targets and integrators, gripper state, environment/proposer RNG, and observation-history buffers. Store the model/assets and controller fingerprints.

A simulator adapter may advertise branch evaluation only after restore-replay tests pass. For each of at least 20 development snapshots, replay the same prefix and continuation three times and report max state error plus equality of success/termination flags. Initial same-host target: maximum absolute joint-state error <= 1e-6, with no discrete-outcome differences. This is a test target, not an asserted property of MuJoCo. Record numeric tolerances and actual discrepancies.

If branching fails, continue ordinary closed-loop evaluation but mark counterfactual metrics unavailable. Never label a partial state reset as an exact oracle. A development fix cannot quietly relax tolerances after test failures.

## 6. Files and identity

Each run writes `manifest.json`, `episodes.jsonl`, `decisions.jsonl`, `metrics.json`, and artifact references. Large arrays use `.npz`; images/video and simulator snapshots are separate files with SHA-256 digests. Never use untrusted pickle or deserialize arbitrary Python objects from downloaded runs.

The manifest contains code revision/dirty state, config and lockfile hashes, dependency versions, environment/task/assets/controller revisions, model revision, hardware/OS, seeds, split IDs, timing mode, observation regime, and runtime/diagnostic compute separately.

Candidate identity is a stable digest of decision identity plus canonical plan and proposer revision. Randomize presentation order independently; do not encode expert status or label in IDs. Canonical serialization uses UTF-8, sorted keys, compact separators, finite values, and a schema version. Test round trips; do not promise cross-platform bitwise dynamics identity.

Labels belong in `diagnostics/labels.jsonl`, keyed by decision and candidate ID. Train/calibration/test exports must preserve parent episode IDs. Runtime prompts and model inputs are saved in an allowlisted, redacted form; credentials and personal data are excluded.
