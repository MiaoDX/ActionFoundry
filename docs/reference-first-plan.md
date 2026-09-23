# Reference-first experimental plan

Status: accepted near-term engineering direction, September 23, 2026.

## 1. Decision

Near-term ActionFoundry work is **reference-project-first**.

Do not require the full P0 -> P4 stack to be implemented before obtaining experimental evidence. Start from a contemporary project that already provides a runnable policy, released checkpoints, a simulator evaluation path, and a reasonably local modification surface. Reproduce its native path first, then make one controlled change at a time.

ActionFoundry remains the place for experiment identity, comparison discipline, candidate/selection logging, metrics, and later reusable abstractions. It is not initially required to replace the reference project's policy runtime, controller, environment adapter, preprocessing, or training stack.

The existing P0-P4 specification remains useful as a capability map and longer-term research plan. Its contracts should be implemented only when an experiment needs them or when the same capability has been required by more than one reference integration.

## 2. Why

The first research objective is to obtain reliable lesson learned about explicit action selection, action interfaces, and world/action-model guidance. Building a new robotics stack first creates avoidable coupled failure modes: simulator integration, controller semantics, policy reproduction, candidate generation, evaluation, and learning can all block the first informative experiment.

A reference-first path reduces this dependency chain. Failure to reproduce one upstream project blocks that integration, not the entire research program. Missing snapshot/restore blocks exact counterfactual diagnostics, not ordinary paired closed-loop evaluation. Missing WAM internals blocks representation/prediction studies, not native-policy evaluation.

Do not treat upstream success claims as reproduced until they run locally.

## 3. Initial substrate bake-off

Audit three candidates before committing to a primary substrate:

1. **Physical Intelligence openpi / pi0.5 on LIBERO** — primary stable-policy candidate. It is attractive as a policy/inference/training substrate; ActionFoundry would add candidate sampling, scoring, selection, and comparison rather than replace the native action stack.
2. **Show-Harness on ManiSkill** — primary action-interface candidate. Its semantic action vocabulary, interpreters, simulator runners, released models/data, and plugin structure make it suitable for action representation, chunking, adaptive-step, history, and recovery experiments.
3. **tau0-VLA on LIBERO** — primary contemporary decision/WAM candidate. Its proposal plus world-model-guided test-time computation is close to ActionFoundry's selection question, but its recent release means local reproducibility and modification cost must be measured before adopting it.

Keep **OpenWAM** as the next WAM substrate once a simpler selection experiment establishes a reason to add world/action representations or predictions. Keep **V-GPS** as an important methodological/value-guided reranking baseline rather than the default 2026 engineering substrate.

The bake-off is an engineering-readiness audit, not a model leaderboard.

## 4. Bake-off acceptance matrix

For each candidate, pin the exact upstream commit and record:

| Check | Evidence |
| --- | --- |
| Clean install | Commands, OS/Python/CUDA, lock or resolved dependency list |
| Assets | Checkpoint/data download succeeds; hashes or immutable revision recorded |
| Single inference | One native observation produces a valid native action |
| Simulator startup | Official evaluation environment starts without ActionFoundry modifications |
| Native smoke evaluation | A small fixed task/initial-state set completes and records success/failure/errors |
| Reset control | Determine exactly how task initial states/seeds are controlled; do not infer pairing from a model RNG flag |
| Raw action access | Native pre-execution action/chunk is observable without changing semantics |
| Candidate access | Determine whether K alternatives can be sampled/generated and what randomness/temperature changes |
| Pre-execution hook | A scorer/selector can be inserted before native execution with a small, reversible patch |
| Logging | Observation identity, raw action, processed action, outcome, latency, and error can be recorded |
| Disable equivalence | With the experiment hook disabled, behavior matches the pinned native path within declared determinism limits |
| Resource profile | Measured GPU memory, disk, warm/cold inference latency, and setup friction |

Classify each candidate as **PASS**, **PARTIAL**, or **BLOCKED**, with concrete reasons. Do not spend substantial time repairing all candidates. The purpose is to select one substrate.

## 5. Selection rule

Prefer the substrate that:

- reproduces a native baseline on the available local hardware;
- has released weights and a maintained evaluation path;
- exposes the action/chunk before execution;
- permits candidate generation and a pre-execution selection hook without rewriting the policy;
- preserves native controller, preprocessing, normalization, and termination semantics;
- supports fixed or persisted initial states well enough for paired comparisons;
- has a small enough modification surface that the experiment can be disabled and audited;
- gives the shortest path to one controlled selection experiment.

Recency is useful but is not sufficient. A newly released project that cannot yet be reproduced locally does not outrank a slightly older but stable substrate.

Do not deepen more than one integration at the same time.

## 6. First experiment after substrate selection

First reproduce the native policy on a prespecified small task/state manifest. Then add logging and a disabled-by-default experiment hook. Verify disable equivalence before changing the decision rule.

The first study should be small and diagnostic:

### E-R1 — Candidate budget

Hold the native model, observation preprocessing, controller, action normalization, task states, and execution semantics fixed. Compare the native path with several candidate budgets where the substrate supports meaningful sampling, for example K in {1, 4, 8, 16}. Record actual candidate diversity as well as compute cost.

Do not equate K=1 sampling with the native policy unless they are verified to be behaviorally identical.

### E-R2 — Fixed-pool selection

For a fixed candidate-generation procedure and K, compare a small number of selection rules. Always retain the unmodified native policy as a first-class comparator. Candidate-random selection is useful for measuring whether the pool alone makes the task easy. Add a new scorer only when its inputs and supervision are explicit.

Primary pilot evidence:

- closed-loop task success/failure on the fixed initial-state manifest;
- paired outcome differences where pairing is valid;
- candidate diversity and duplicate rate;
- selection changes relative to the native/default action;
- warm inference and scoring latency;
- GPU memory and total decision cost;
- abort/error/timeout counts;
- representative failure traces.

A 20-30-state/task pilot is exploratory. Do not interpret small percentage differences as confirmatory evidence.

## 7. Evidence-driven next steps

Add infrastructure only in response to an observed question:

- If candidate samples are nearly identical, investigate proposal diversity before building a more complex scorer.
- If selection changes outcomes but attribution is unclear, add validated snapshot/restore and fixed-state counterfactual labels where the environment supports them.
- If a value/success scorer appears useful, compare it with the native direct policy under matched information and report its additional labels and compute.
- If action-interface behavior is the bottleneck, use Show-Harness-style representation/interpreter experiments rather than forcing all actions into a new universal representation.
- If selection is useful and a WAM may provide decision-relevant information, integrate a frozen OpenWAM or native WAM representation/prediction path before considering joint training.
- If the native policy is saturated or reranking only adds cost, preserve the negative result and change the research question rather than constructing a favorable benchmark.

## 8. What ActionFoundry owns initially

Keep the first ActionFoundry layer thin:

- upstream repository/checkpoint/config/task-manifest identity;
- experiment variant identity and disable switch;
- observation/action/selection/outcome logging;
- paired comparison and resource metrics;
- provenance and failure accounting;
- optional candidate/scorer hook.

Do **not** initially require a universal environment abstraction, canonical end-effector plan for every upstream policy, a new simulator runtime, a new VLA training stack, or a general plugin framework.

When two independent integrations need the same capability, consider promoting it into a shared ActionFoundry contract. Until then, keep substrate-specific code local.

## 9. Coding-agent opening task

Start with the bake-off, not with implementation of the previous I0-I10 sequence.

1. Read this document, AGENTS.md, and the existing design documents for integrity requirements.
2. Create isolated local checkouts/environments for openpi, Show-Harness, and tau0-VLA. Do not vendor their source into ActionFoundry.
3. Pin the inspected upstream commit before modifying anything.
4. Attempt the smallest official no-training inference/evaluation path for each candidate.
5. Record exact commands, resolved versions, required assets, measured resources, failures, and whether each row of the bake-off matrix is PASS/PARTIAL/BLOCKED.
6. Do not repair all three projects or start model training. Stop the bake-off once there is enough evidence to choose one primary substrate.
7. For the selected substrate, propose the smallest reversible patch that exposes candidate generation and a pre-execution selection hook while preserving the native path.
8. Do not claim benchmark reproduction from a smoke run. Do not silently substitute a different task, checkpoint, controller, or action normalization when an official path fails.

The next decision after the bake-off is which single substrate to deepen, not which universal ActionFoundry architecture to implement.
