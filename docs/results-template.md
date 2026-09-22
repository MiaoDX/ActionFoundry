# Experiment report template

Copy this template into a new report only when running an experiment. Values below are placeholders; this file is not a result.

## Identity

Status: `not_run | exploratory | confirmatory | blocked`.

Experiment ID, date, code commit, dirty state, command, config digest, dependency lock, environment/assets/controller/model revisions, hardware/OS, observation regime, timing mode, candidate representation/support, planned horizon/executed prefix, and runtime versus diagnostic compute.

## Protocol

State the question, prespecified primary comparator/metric, initial-state manifest, task list, source-episode split, training/calibration provenance, model training overlap, seeds, episode denominator, perturbation schedule, and exclusions. Link any protocol amendment made before test evaluation.

## Results

| Method | Attempted episodes | Ever/final success | Paired difference and 95% CI | Abort/error/violation counts | Wall time / simulated ticks |
| --- | --- | --- | --- | --- | --- |
| Not run | — | — | — | — | — |

Report per-task and per-regime tables. Include actual candidate counts, empirically labeled opportunity, regret/coverage, missing-label fraction, calibration target and metrics, scorer latency percentiles, observation age, and measured compute/storage costs where applicable. Preserve all seeds; do not show only the best run.

## Integrity checks

Record frame/gripper probes, replay tolerances and errors, label leakage audit, matching controller/support/data budgets, and whether diagnostic labels use hidden simulator information. Link representative failures as well as successes.

## Interpretation

Distinguish observed results from possible explanations. State which gate is satisfied, which is blocked, alternative explanations, and the next narrow action. Do not generalize state-only results to vision or finite simulator testing to hardware safety.

## Artifacts

Manifest and raw trace locations with hashes, metrics-generation command, logs, small audit samples, and the exact steps a second machine needs to reproduce the report.
