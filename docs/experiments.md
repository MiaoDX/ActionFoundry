# Experiment plan

Status: **planned, not run**. Numeric budgets and gates below are preregistered engineering/research choices, not measured performance.

## 1. Questions and attribution

The main hypothesis is that explicit selection can improve a matched system's behavior, efficiency, or failure diagnosis. It may be false. Separate four axes: candidate support, representation, scoring/training, and execution timing. Do not change all four and attribute the difference to a decision head.

The nearest method-level precedents include QT-Opt, Implicit Behavioral Cloning, and V-GPS. SayCan addresses a coarser skill-selection setting. The baseline labels below mean mechanism comparisons unless the original code and protocol are actually reproduced [R04](references.md#r04) / [R05](references.md#r05) / [R06](references.md#r06) / [R07](references.md#r07).

## 2. Baseline matrix

| ID | Method | Phase | What it controls |
| --- | --- | --- | --- |
| S0 | Seeded uniform over admitted candidates | P1 | Whether the pool alone makes the task easy |
| S1 | Deterministic geometric/phase heuristic | P1 | What explicit state and task logic already solve |
| D0 | Direct-scripted command from the same phase machine | P1 | Candidate machinery versus the underlying controller |
| O0 | Reference-rollout best candidate | P1 diagnostic | Limited-horizon opportunity in a fixed pool; privileged |
| S2 | Small pointwise rollout-utility regressor | P2 | Benefit of learned evaluation with fixed support |
| S3 | Shared encoder with masked set-aware ranking | P2 optional | Benefit of comparing alternatives jointly |
| D1 | Direct continuous/chunk behavior cloning | P2 | Matched-data direct action generation |
| S4 | Value-guided reranking / offline-Q comparator | P2/P3 | Relation to V-GPS-style value guidance |
| F0 | Frozen model constrained action-ID generation | P3 optional | Language/vision decision baseline |
| F1 | Same frozen backbone, candidate token-logit selection | P3 optional | Decoding/interface overhead, not new training |
| F2 | Learned typed action/success heads | P3 optional | Effect of representation, supervision, and auxiliary targets |
| J0 | Hosted Jev with matched text-state information | Optional | External service comparison, not the main thesis |

S1 scores agreement with the phase machine's desired **executed-prefix endpoint**, with explicit distance, motion-cost, and grip-mode terms. Normalize terms on development data, log the equation and coefficients, and use exactly the same information as D0. D0 emits one capped pose target without ranking; it shares compiler, controller, checks, and tick budget.

D1 uses the same source episodes and observations as learned selectors. Log the additional counterfactual labels/compute available to S2–S4. A secondary equal-label-budget comparison supplies identical labels where the formulations permit it. Do not claim identical supervision merely because both methods use the same initial episodes.

## 3. Ordered experiments

| Experiment | Design | Primary evidence |
| --- | --- | --- |
| E00 | Mock compiler, selection, replay, failure and privilege tests | Contract correctness |
| E01 | Lift S0/S1/D0; then Stack; native closed-loop trials | Behavior and implementation sanity |
| E02 | Fixed decision snapshots, branch all candidates using a frozen continuation | Coverage and empirical selection regret |
| E03 | Same compiled plans, alternate semantic/relative/chunk encodings | Representation without action-support confounding |
| E04 | Nested K=8/16/32 pools; generator/phase-proposal ablation | Proposal bottleneck and compute scaling |
| E05 | Fixed pool S1/S2, D1, then optional S3/S4 | Learned selection versus direct control |
| E06 | Candidate order, IDs, duplication, paraphrase, and source shift | Shortcut sensitivity and generalization |
| E07 | H/prefix and inference-delay ablations | Responsiveness, oscillation, and latency tradeoffs |
| E08 | LIBERO policy alone versus its candidates plus reranker | VLA transfer under matched information |
| E09 | Measured abstention/reproposal/recovery; optional predicted futures | Incremental value of recovery or dynamics |

E03 with identical compiled plans and numeric S1 should produce identical decisions. This is a sanity test; encoding effects become meaningful with an encoding-sensitive scorer. E07 varies planned horizon H in {4,8,16}, execution prefix in {1,4,H}, and injected inference delay in {0,50,100,200} ms in the separate latency-aware mode. Each row declares its physical action support and speed limits.

### The single-token null test

For one unchanged model, prompt, candidate token set, and constrained greedy decoding, taking `argmax` of candidate logits and generating that single constrained token should choose the same action, up to ties/numerics. It is not evidence of improved intelligence to remove the decoder wrapper. Check candidate tokenization, multi-token scoring, masks, temperature, and context equality before interpreting any difference.

Typed multi-head models require an explicit loss and supervision. Do not call cross-entropy plus Brier loss a reproduction of an undisclosed RLCD recipe. Independently chosen axis/gripper outputs must be compiled into a joint feasible command; per-axis validity does not establish joint feasibility.

## 4. Counterfactual labels and oracle limits

At a frozen snapshot s, execute the same prefix length n that the evaluated runtime would execute for candidate c_i. Then apply the frozen reference continuation policy pi_ref for L=40 controller ticks, or until the episode budget ends. Store the prefix length, continuation revision, L, and all random seeds.

Use two labels:

- `y_i`: whether native task success occurs during the prefix plus continuation.
- `q_ref_i = y_i + mean(r_norm)`, where `r_norm = clip(r_shaped / reward_scale, 0, 1)` is the explicitly recorded normalized task reward over that branch.

This is a **reference-rollout utility proxy**, not optimal Q, a Bellman-consistent critic, or unrestricted task success. A failure under pi_ref may still be recoverable by a better continuation. Early infrastructure errors are missing labels with status codes, not zeros. Missing-label coverage is reported.

For eligible states with complete labels, define empirical opportunity `mean_s max_i y_i`, regret `mean_s (max_i q_ref_i - q_ref_selected)`, and conditional selection success over states with at least one successful branch. The domain is the admitted fixed pool at those states. O0 is not a mathematical upper bound on all closed-loop policies or future proposal strategies. Abstentions have no selected-candidate regret: report their coverage and actual fallback outcome separately, and never improve a metric by dropping them silently.

Start with one branch per candidate in the deterministic fixture. On a prespecified 10% development audit subset, repeat with five recorded disturbance seeds to inspect sensitivity. A single deterministic label can support prediction across a held-out state population; it does not estimate repeat-trial physical reliability for a particular action.

## 5. Data and first learned model

Collect a mixed state distribution from S0, S1, and D0, including failures. Cap the first pilot label set at 5,000 decision snapshots total, stratified by task, phase, outcome, and source episode, with no more than 20 snapshots per episode. Use K=16 before scaling. Apply the split in [benchmarks](benchmarks.md) before generating branches.

S2 begins with a shared numeric context/candidate encoder and two hidden layers of width 128, ReLU, scalar utility regression. Train three seeds {0,1,2}; AdamW starting learning rate 1e-3; batch size 256; maximum 50 epochs; validation early stopping patience 5. These are initial development settings, not a mandatory claim of optimality. Standardization uses training data only. No candidate ID or generation index is a predictive feature.

Add a separate sigmoid success head with binary cross-entropy only when `y_i` has usable class support. Report Brier score and log loss on the calibration/test protocol. The utility loss is MSE against q_ref; it is not itself a probability objective. P2's first report compares S2 with S1 and D1 before adding set-aware capacity, multimodal backbones, or auxiliary heads.

If training snapshots do not cover visited learned-policy states, measure the shift. A later aggregation round may collect new training-seed rollouts only; freeze a new dataset revision and never harvest test rollouts for fitting.

## 6. Metrics and statistical units

Primary: per-task `ever_success` and paired difference against the prespecified primary comparator S1. Also report final success, abort/error/timeout rates, operational violations, simulated ticks, wall-clock duration, actual K, and candidate/selection diagnostics.

Report runtime latency p50/p95/p99, proposal/scorer/compiler/execution timings, observation age, cold versus warm runs, and achieved closed-loop throughput separately. Include oracle labeling, training, and model download costs outside runtime totals. A cached/offline scorer cannot claim online latency.

Calibration: independent success probabilities use Brier/log loss and a reliability table; ECE uses 10 fixed equal-width bins with counts and uncertainty. Candidate softmax calibration is a different target and is not reported as success calibration. Measure both all labeled candidates and selected candidates; deployment selection changes the evaluated distribution. Threshold tuning uses the held-out calibration split only [R20](references.md#r20).

Use episodes/initial states as resampling units, not timesteps. For learned methods report all three training seeds and a hierarchical bootstrap (training seed, then paired initial state within task) with 10,000 replicates and 95% intervals. For deterministic baselines use paired episode bootstrap; Wilson intervals are acceptable for single success rates. Report per-task results plus an equally weighted task mean. Repeated windows from the same episode do not increase independent sample count.

Pilot: 30 development episodes/task/method. Confirmatory B1: all 300 held-out ID states per task and enabled method; OOD runs use the separately frozen 300-state manifest. Do not stop early when a favorable result appears. Fewer runs remain exploratory and report wider intervals. Adjust confirmatory sample size using pilot paired discordance and a declared target effect before touching test outcomes.

Multiple optional ablations are exploratory unless preregistered. Do not claim success because one of many settings happens to win. ID/OOD and state/vision regimes have separate denominators and tables.

## 7. Gates and stop conditions

**G0 — Implementable contract.** CPU fixture passes frame composition, quaternion, gripper, round-trip, masking, immutability, finite-score, stale-plan, bounded-retry, and privilege-isolation tests.

**G1 — Trustworthy simulator.** Native Lift and Stack success is independently checked; replay error is within the declared tolerance and discrete outcomes agree; every attempted episode produces a trace or explicit infrastructure-error record. Smoke runs are not a performance gate. If restoration fails, E02 is blocked—not fabricated.

**G2 — A measurable selector problem.** On development states, opportunity is at least 70% under the declared reference continuation and there is at least a 10-percentage-point gap between opportunity and unconditional selected-branch success, measured on the same snapshot denominator. These are project triage thresholds. If opportunity is low, inspect proposer/continuation first; if S1 is near the diagnostic ceiling, record a saturated slice rather than force learning.

**G3 — Evidence worth expanding.** On the confirmatory protocol, either (a) learned selection has a positive paired-success lower confidence bound versus S1 and at least a 5-point estimated gain, or (b) success is non-inferior within a prespecified 2-point margin and measured runtime decision cost is at least 20% lower. Predeclare which route is primary. These are expansion criteria, not guarantees that current sample sizes can establish them. No increase in observed operational violations is allowed for promotion; finite tests do not establish hardware safety.

**G4 — Transfer justified.** Reproduce the selected visual policy baseline locally; then add the reranker under matched observation, action, and evaluation contracts. A state-only gain does not bypass this gate.

A gate failure produces a report and a scoped revision, not a requirement to make the method win. Do not repeatedly inspect test results while tuning; any reopening uses a new holdout and versioned protocol.

## 8. Resource guardrails

P0 requires no GPU, network service, simulator assets, or model downloads. P1 state simulation is designed to avoid a GPU requirement; actual platform compatibility must be tested locally. Default diagnostics stop at 5,000 snapshots and K=16; first run a 100-snapshot throughput/storage pilot and estimate the full cost before expanding. Budget is expressed in episodes, branch-control ticks, model calls, disk bytes, and GPU-hours where applicable—not promised wall-clock time.

P2 starts small. P3/P4 model downloads, substantial GPU jobs, and paid API usage require an explicit run budget and local opt-in. A missing service/checkpoint is reported as skipped, never replaced silently by a different model.
