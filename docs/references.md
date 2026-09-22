# Reference landscape and evidence register

Primary-source check: September 22, 2026. This is a scoped reading map for the v0.1 design, not an exhaustive survey or an independent reproduction of the listed results. Project pages, abstracts, official documentation, and the identified repository READMEs were inspected. Paper-level claims should be rechecked against the full experimental protocol before a reproduction claim is made.

## How to use this map

**Direct mechanism** means an explicit overlap with proposal, scoring, selection, or action interfaces. **Adjacent** means a useful representation, control, or training precedent. **Infrastructure** means an implementation/evaluation dependency. None of these labels means the work implements Jev's undisclosed internals.

No success-rate table mixes numbers across these sources: their embodiments, observations, data, controllers, and task distributions differ. Read R04–R07 before proposing candidate reranking as a novel algorithm; read R08/R11–R17 before attributing benefits to an action representation; read R20 before interpreting confidence.

## Infrastructure

### R01

**robosuite environment documentation — infrastructure.** [Official documentation](https://robosuite.ai/docs/modules/environments.html).

The documented environments include Lift and Stack, robot/object observations, configurable episode/control settings, and task success checks. This supports the initial benchmark choice, not the untested claim that our snapshot/replay adapter is correct. ActionFoundry's horizons, splits, and metrics are local protocol choices.

### R02

**robosuite controller documentation — infrastructure.** [Official documentation](https://robosuite.ai/docs/modules/controllers.html).

Composite controllers distribute commands to body-part controllers, including OSC_POSE. The adapter must validate scaling, frame, gripper mapping, and restored controller state. An upstream controller name does not prove our normalized-to-SI conversion is correct.

### R03

**LIBERO: Benchmarking Knowledge Transfer for Lifelong Robot Learning (2023) — infrastructure.** [Repository](https://github.com/Lifelong-Robot-Learning/LIBERO), [paper](https://arxiv.org/abs/2306.03310).

The inspected README supplies task-suite and initial-state APIs and an older Python installation recipe. Code and dataset licensing are separate. Use it for the later visual-transfer protocol, with isolated dependencies and exact task/state manifests. README inspected at blob `96cf5b87eb7bb42be1269581256cc85bd7e21b6f`; the blob hash is not an executable environment pin.

## Candidate evaluation and selection

### R04

**QT-Opt: Scalable Deep Reinforcement Learning for Vision-Based Robotic Manipulation (2018) — direct mechanism.** [Paper](https://arxiv.org/abs/1806.10293).

Q-based closed-loop manipulation is an essential predecessor to learned action evaluation. It prevents equating “choose an action using a learned score” with a new model category. Its original large-scale real-robot learning setting is not reproduced by our small state-based harness.

### R05

**Implicit Behavioral Cloning (2021; CoRL proceedings 2022) — direct mechanism.** [Project](https://implicitbc.github.io/), [paper](https://arxiv.org/abs/2109.00137).

Energy-based implicit policies provide a precedent for selecting actions rather than directly regressing one output. Their energy is not automatically a success probability. Compare model capacity, observations, supervision, and inference search cost before claiming an advantage over explicit policies.

### R06

**V-GPS: Steering Your Generalists (2024) — direct mechanism.** [Project](https://nakamotoo.github.io/V-GPS/), [paper](https://arxiv.org/abs/2410.13816).

V-GPS reranks generalist-policy actions with an offline-RL value function at deployment. This is a particularly close comparator to proposal-plus-selection. A new candidate scorer needs to distinguish its supervision, candidate-set generalization, calibration, or compute tradeoff from ordinary value guidance.

### R07

**SayCan: Do As I Can, Not As I Say (2022) — direct at skill level.** [Project](https://say-can.github.io/).

SayCan combines language relevance with skill affordance/value estimates to choose executable skills. It motivates separating semantic desirability from feasibility. Its skill-level decisions are not the same temporal or physical interface as fine-grained relative moves.

### R08

**Show-Harness: Just a VLM Agent Can Play Robots (2026) — direct action-interface reference.** [Project](https://showlab.github.io/Show-Harness/), [repository](https://github.com/showlab/Show-Harness), [paper](https://arxiv.org/abs/2609.10522).

The project describes discrete semantic action units and embodiment-specific interpreters, with GUMI data collection. It motivates a semantic-interface baseline, not replacing the entire research program. Its interface, controller, context plugins, and training regime must be matched to isolate action-token versus scoring effects. Performance numbers here are deliberately not imported.

### R09

**TypeSafe: Introducing System One Models & Jev (September 15, 2026) — vendor interface inspiration.** [Official release](https://typesafe.ai/blog/introducing-system-one-models-and-jev).

The vendor describes structured probabilistic decisions and parallel outputs, and names RLCD. These statements establish the exposed product framing, not a public reproducible training recipe or calibrated robotic success probabilities. Schema-valid output can still select a physically wrong action. The state-based Doom example is not visual robot control.

### R10

**OpenRoboto Jev robot control — direct prototype.** [Repository](https://github.com/openroboto-ai/jev-robot-control).

The inspected README describes structured simulator observations, intent plus Cartesian-direction/gripper decisions, a shared executor, archived traces, and an offline verifier. It explicitly calls the comparisons single-seed recordings rather than success-rate estimates, and distinguishes its model probabilities from physical-success calibration. No camera-policy or hardware generalization claim is inherited. README blob: `0e749c38197e78c46dacbb1f61910ef2b244a823`.

## Action representation and execution

### R11

**Universal Manipulation Interface / UMI (2024) — adjacent representation and timing.** [Project](https://umi-gripper.github.io/).

UMI emphasizes relative-trajectory representation and inference-time latency matching. It supports examining reference frames and temporal execution separately from the choice model. Relative motion alone does not guarantee cross-embodiment transfer, and UMI is not merely a finite-action classifier.

### R12

**RT-2 (2023) — adjacent tokenized action policy.** [Paper](https://arxiv.org/abs/2307.15818).

RT-2 is a vision-language-action precedent for connecting token prediction to robot control. It informs the action-token comparator; it is not evidence that removing token decoding changes the decision made by an otherwise identical model.

### R13

**Perceiver-Actor / PerAct (2022) — adjacent discrete spatial action prediction.** [Project](https://peract.github.io/).

PerAct predicts discretized translation, rotation, gripper, and collision-avoidance actions from language and voxel observations. Discrete spatial prediction is a strong predecessor, but its voxel space is not inherently a local relative-move interface or a runtime-defined semantic candidate set.

### R14

**ACT: Learning Fine-Grained Bimanual Manipulation with Low-Cost Hardware (2023) — adjacent chunking baseline.** [Project](https://tonyzhaozh.github.io/aloha/).

ACT predicts action sequences and originally demonstrates joint-position control. Action chunking and relative end-effector coordinates are separate choices. An ACT-style direct comparator must preserve its action semantics rather than being silently converted to a different controller.

### R15

**Diffusion Policy (2023) — adjacent direct policy and proposal source.** [Project](https://diffusion-policy.cs.columbia.edu/).

Diffusion-based action-sequence prediction is a direct-policy baseline and a possible source of diverse candidate chunks. Sampling multiple chunks and selecting one changes inference compute; compare against the original policy and matched-budget alternatives. No claim is made that diffusion policies universally reduce to lookup or finite classification.

### R16

**VoxPoser (2023) — adjacent value-map planning.** [Project](https://voxposer.github.io/).

VoxPoser composes affordance/constraint maps used to synthesize trajectories. It suggests a non-neural or programmatic evaluator baseline. A spatial value map, a finite-set ranking score, and a calibrated success probability are not interchangeable quantities.

### R17

**ReKep (2024) — adjacent constraints and intermediate representation.** [Project](https://rekep-robot.github.io/).

ReKep turns keypoint relations into optimizable constraints for robot motion. It motivates explicit task/geometry checks rather than asking a neural judge to rediscover every invariant. Generated constraints themselves still require validation and are not certified safety rules.

## Alignment, generalist policies, and uncertainty

### R18

**GRAPE: Generalizing Robot Policy via Preference Alignment (2024 preprint) — adjacent training objective.** [Paper](https://arxiv.org/abs/2411.19309).

Trajectory-level preference alignment uses successful and failed behavior to improve VLA policies. It is a training-side comparison, not simply a test-time reranker. A demonstration's selected action does not label all other candidates as unsuccessful; explicit outcomes or defensible preferences are needed.

### R19

**OpenVLA (2024) — adjacent generalist policy.** [Project](https://openvla.github.io/).

OpenVLA is a potential later policy baseline/proposal source, subject to local checkpoint, action-normalization, and benchmark reproduction. The v0.1 spec does not mandate this model or assume that an untested checkpoint is compatible with the chosen worker environment.

### R20

**On Calibration of Modern Neural Networks (2017) — evaluation precedent.** [Paper](https://arxiv.org/abs/1706.04599).

Calibration and post-hoc temperature scaling motivate held-out calibration data and proper scoring metrics. Calibration is distribution- and target-dependent; fitting a temperature does not provide a physical safety guarantee, and candidate-choice confidence is not task-success confidence.

## World modeling and alternative interfaces

### R21

**Deep Visual Foresight for Planning Robot Motion (2016; ICRA 2017) — direct planning precedent.** [Paper](https://arxiv.org/abs/1610.00696).

Action-conditioned visual prediction combined with model-predictive control predates the current decision-model discussion. It motivates an optional candidate-outcome predictor but also requires separating model error, cost design, proposal quality, and selection quality.

### R22

**Embodied-R1 (2025; revised 2026) — adjacent pointing interface.** [Paper](https://arxiv.org/abs/2508.13998).

The paper describes pointing as an intermediate representation connecting visual reasoning and action primitives. This is an alternative to relative-move tokens, not evidence that all useful embodied interfaces should be discrete candidate classifiers.

### R23

**Language Movement Primitives (2026) — adjacent continuous primitive interface.** [Paper](https://arxiv.org/abs/2602.02839).

The current record connects VLM reasoning to dynamic movement primitive parameters. This preserves continuous motion in a compact interface. Earlier conversational task counts/performance values are not copied; the revised paper should be the source for any quantitative comparison.

### R24

**DSWAM: A Dual-System World Action Foundation Model (2026 project) — adjacent world/action architecture.** [Project](https://ds-wam.github.io/).

The authors describe a WAM executor and optional language-level planner; action inference need not explicitly generate future video. Therefore “uses world-model training” does not imply an available candidate-conditioned rollout API. ActionFoundry must test that capability rather than assume it from the WAM label.

### R25

**GR00T N1 (2025) — adjacent dual-system counterpoint.** [Paper](https://arxiv.org/abs/2503.14734).

Its VLM and diffusion action modules form a dual-system architecture trained together. A robotics “System 1” may generate continuous actions rather than typed decisions. Shared terminology is not evidence of identical architecture or an omitted component that every VLA needs.

## Qualified or unresolved references

### R26

**Action with Visual Primitives / AVP — withdrawn reference.** [arXiv record](https://arxiv.org/abs/2605.22183).

The record inspected on September 22, 2026 marks v4 withdrawn on September 20, 2026. The authors state that further experimental improvements are needed before publishing. Retain it only as historical context for an intermediate visual interface. Do not use its reported gains as positive evidence for this design or a reproduction target.

### Unverified leads from the earlier conversation

PrimitiveVLA (`arXiv:2605.28634`) could not be retrieved in this source pass; no mechanism or numerical result is treated as verified here. Several earlier-mentioned Jev replicas and robotics demos were not individually re-audited for this commit. Their earlier summaries are leads, not established evidence. This does not assert that those projects are absent or invalid.

Before adding a reproduction: verify the primary artifact, publication/revision status, code and weight availability, license, observation privileges, task protocol, and exact checkpoint/source revision. Preserve negative findings and withdrawals rather than silently replacing the evidence.
