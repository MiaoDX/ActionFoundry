# ActionFoundry

**A research workbench for candidate-based robot action selection.**

ActionFoundry explores an alternative to treating robot control as a single monolithic action-generation problem. The central question is:

> Given the current observation, task context, and a structured set of candidate actions or trajectories, how should a robot evaluate, select, and execute the next action?

The project is deliberately staged: first build a reliable candidate/action interface and evaluation harness, then establish non-learned baselines, and only then introduce learned scoring, selection, and planning components.

<p align="center"><img src="docs/assets/architecture.svg" alt="ActionFoundry architecture" width="900"></p>

## Research thesis

Modern robot policies often collapse perception, reasoning, action generation, and control into one learned mapping. ActionFoundry separates these concerns around an explicit **candidate set**. Candidates may come from motion primitives, trajectory samplers, planners, policies, retrieval, or learned proposal models. A selector evaluates them using task relevance, feasibility, safety, progress, value, uncertainty, or other signals before execution.

This decomposition gives us a clean place to ask:

- Can explicit candidate selection improve robustness or sample efficiency relative to direct action generation?
- Which candidate representations are useful: discrete actions, motion primitives, short-horizon trajectories, or hybrids?
- How much can deterministic/model-based scoring achieve before learning is required?
- Can a learned judge/value/ranker generalize across proposal mechanisms?
- When can a fast selector act, and when should a slower replanning path take over?

## Architecture

The intended loop is:

`observation + task → context → candidate proposal → constraints / scoring → selection → execution → feedback`

Proposal, evaluation, and selection are independently replaceable and benchmarkable. See [Architecture](docs/architecture.md).

## Roadmap

| Phase | Goal | Main output |
| --- | --- | --- |
| **0 — Contract** | Define observations, candidates, evaluations, traces, and metrics | Stable interfaces and schemas |
| **1 — Harness** | Make candidate generation and evaluation reproducible | Evaluation harness + deterministic baselines |
| **2 — Learned selection** | Learn to score/rank/select candidates | Judge/value/ranker baselines |
| **3 — Closed loop** | Re-select under execution feedback | Receding-horizon decision loop |
| **4 — Extensions** | Compare proposal sources, representations, and reasoning regimes | Ablations and research results |

The sequencing principle is **measure first, learn second**.

## Design principles

1. **Explicit candidates.** Make action alternatives inspectable.
2. **Proposal/selection separation.** Candidate generation and evaluation need not be the same model.
3. **Strong non-learned baselines first.** Establish what geometry, constraints, heuristics, and planning already solve.
4. **Trace every decision.** Preserve candidates, scores, rejection reasons, selected actions, latency, and outcomes.
5. **Closed-loop evaluation.** Offline ranking accuracy is insufficient; measure downstream task behavior.
6. **Architecture before model branding.** The repository should survive changes in VLM/VLA/judge/planner choices.

## Documents

- [Architecture](docs/architecture.md) — module boundaries, contracts, metrics, and staged implementation.
- [Research Notes](docs/research-notes.md) — synthesis of the motivating discussion and deep-research direction.

## Status

Early research scaffold. The repository currently captures the working hypothesis and implementation plan before committing to a specific learning stack.

## License

MIT. See [LICENSE](LICENSE).
