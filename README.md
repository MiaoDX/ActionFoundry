# ActionFoundry

**A research workbench for candidate-based robot action selection.**

ActionFoundry studies when explicit action proposals, evaluation, and selection improve robot manipulation—and when direct policies or ordinary control are the better solution. Jev and Show-Harness are references, not mandatory dependencies or a predetermined architecture.

> Given an observation, a task, and a set of executable alternatives, what is gained by deciding among those alternatives explicitly?

<p align="center"><img src="docs/assets/architecture.svg" alt="Observation, proposal, evaluation, selection and execution with feedback" width="900"></p>

## Status

**Design specification v0.1 · September 22, 2026.** This repository contains research and engineering specifications, not a working robot runtime. No ActionFoundry training runs or manipulation results are claimed. Example configurations describe the implementation contract; they are not executable yet.

## Design in brief

- Separate candidate proposal, representation, scoring, selection, and execution. Preserve a direct-policy baseline outside the candidate path.
- Begin with transparent, state-based simulation and no model training. Introduce learned selection only after replay and proposal quality are measurable.
- Keep deployable observations separate from privileged simulator diagnostics. Treat candidate-choice scores and physical-success probabilities as different quantities.
- Evaluate closed-loop behavior from the first simulator milestone. Treat learned world-action models as a parallel research axis once the basic harness is trustworthy: they may propose actions, expose predictive representations, or predict candidate-conditioned futures.

## Starting scope

| Stage | Deliverable | Exit evidence |
| --- | --- | --- |
| P0 — Contracts | CPU-only mock environment, schemas, frame conversions, traces | Unit, isolation, and replay tests |
| P1 — Harness | robosuite Panda Lift, then Stack; deterministic proposals and selectors | Paired rollouts, candidate coverage, failure attribution |
| P2 — Learned selection | Fixed-pool state rankers, direct behavior-cloning control | Held-out regret and closed-loop comparisons |
| P3 — Transfer and WAM | LIBERO visual-policy proposals plus OpenWAM integration | Matched policy baselines; action/world representation probes |
| P4 — Decision-aware world/action learning | Candidate-conditioned prediction, decision heads, recovery, optional joint training | Decision utility, predictive fidelity, and OOD/recovery ablations |

**Measure first, learn second.** A negative result is a valid outcome; candidate selection is the hypothesis, not the conclusion.

## Start here

[Local development handoff](docs/implementation.md) gives the work order and first acceptance tests. [AGENTS.md](AGENTS.md) defines the instructions for a coding agent.

| Document | Purpose |
| --- | --- |
| [Design decisions](docs/decisions.md) | Settled decisions, alternatives, and reopening conditions |
| [Architecture](docs/architecture.md) | Runtime, diagnostics, fallback, and module boundaries |
| [Contracts](docs/contracts.md) | Frames, types, score semantics, snapshots, and trace format |
| [Benchmarks](docs/benchmarks.md) | Environments, proposals, splits, and observation regimes |
| [Experiments](docs/experiments.md) | Baselines, ablations, labels, metrics, budgets, and gates |
| [Reference landscape](docs/references.md) | Primary sources, relevance, and evidence limitations |
| [Research notes](docs/research-notes.md) | Discussion synthesis, provenance, and corrections |
| [Results template](docs/results-template.md) | Required evidence for future experiment reports |

## Local checkout

```bash
git clone https://github.com/MiaoDX/ActionFoundry.git
cd ActionFoundry
```

There is no installation or experiment command to run until P0 is implemented. Start the coding agent with the handoff in `docs/implementation.md`; do not invent benchmark results to fill the documentation.

## License

[MIT](LICENSE). External code, datasets, checkpoints, and robot assets retain their own licenses. The project license does not relicense third-party material.
