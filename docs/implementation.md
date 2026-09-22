# Local development handoff

The next task is implementation, not further architecture brainstorming. Start with the frozen P0/P1 scope and use measurement to settle simulator-specific constants. All runtime commands below are **acceptance targets to implement**, not currently working commands.

## 1. Package boundaries

Target Python 3.10 for the initial core and robosuite worker. Use NumPy, standard-library dataclasses/Protocol/argparse, and a strict configuration/serialization layer. PyYAML may parse configs; unknown fields must fail validation. Add pytest and a linter/type checker for development. Torch, vision models, remote APIs, simulators, and OpenWAM are optional dependencies with lazy imports.

The exact dependency lock is created on the local machine after install and smoke tests. Save separate core, robosuite, and later LIBERO worker locks. LIBERO's upstream example uses Python 3.8.13; isolate that stack rather than downgrading the entire core. A worker protocol can exchange versioned JSON and array-file references over a local subprocess boundary. Do not copy the whole core into each dependency environment.

```text
src/actionfoundry/
  contracts.py          # typed records and runtime validation
  geometry.py           # frames, quaternion handling, plan compilation helpers
  config.py             # strict schema, resolved config, fingerprinting
  runner.py             # bounded synchronous closed loop
  cli.py                # doctor, run, replay, diagnose, summarize
  environments/         # mock and worker adapters
  proposals/            # local lattice, phase target, policy adapter later
  compilation/          # typed spec -> immutable executable plan
  constraints/          # declared checks and operational admission
  scoring/              # random, heuristic; learned adapters later
  selection/            # masked argmax, bounded abstention
  execution/            # control adapter and freshness checks
  diagnostics/          # snapshots, branch evaluator, offline labels
  tracing/              # JSONL, artifact hashes, manifests
  metrics/              # paired episode statistics and calibration
workers/                # isolated simulator entry points, added when needed
configs/                # runnable configs once implementation exists
tests/                  # unit, property, integration, isolation
reports/                # small verified summaries; large artifacts stay external
```

Create only modules needed by the current vertical slice. Avoid an abstract plugin framework, GPU scheduler, or unimplemented placeholder classes across every directory.

## 2. Ordered work packets

| Packet | Deliverable | Required acceptance evidence |
| --- | --- | --- |
| I0 | Package/config/contract validation | Invalid units, shapes, fields, nonfinite values, IDs rejected |
| I1 | Mock environment + compiler | Exact anchored transforms and deterministic save/restore |
| I2 | Proposer/checker/scorer/selector/runner | Complete traces; random and heuristic policies; bounded failures |
| I3 | Metrics/replay CLI | Known fixture coverage/regret and paired intervals reproduced |
| I4 | robosuite Lift adapter | Six direction probes, rotation probe, gripper polarity, reset and replay checks |
| I5 | Lift P1 experiments + Stack adapter | Pilot reports, resource profile, frozen split/config/assets manifests |
| I6 | Offline branch labels | Continuation and controller state restoration; privilege-isolation audit |
| I7 | P1 confirmatory report | S0/S1/D0, E02 diagnostics, explicit G0–G2 assessment |
| I8 | P2 model/data pipeline | Only after a measured selector gap; matched-data D1 comparator |
| I9 | OpenWAM worker adapter | Reproduce pinned checkpoint baseline; action/state mapping and provenance tests |
| I10 | WAM decision experiments | Frozen-feature head first; conditional prediction/joint training only after E10/E11 gates |

Each packet can be a small implementation commit. Do not run I8 merely because its module is easy to add. A failing pilot may legitimately lead to a proposer or controller fix before learning.

## 3. Test requirements

Unit tests must cover: base versus body-frame increments; anchor fixed across a chunk; 90-degree frame rotation; quaternion sign equivalence/normalization; grip `KEEP`; timestamps and stale plan rejection; candidate deduplication; absent/duplicate IDs; score masking; NaN/timeout behavior; all-invalid and empty sets; order-independent tie breaking; bounded reproposal; immutable compiled plans.

Isolation tests must install a policy that attempts to access hidden snapshots, reward functions, branch labels, or test manifests and prove access is rejected by construction. Source and artifact paths are not enough: passing an environment object indirectly still violates the boundary.

Replay tests distinguish exact mock replay from tolerance-based simulator replay. JSON round trips must preserve semantic values. Every executed command must have an observation identity, controller contract, plan hash, and trace event. Candidate encoding-only variants must compile to the same plan digest.

Statistics tests use small synthetic arrays with known coverage, regret, bootstrap pairing, missing-label rates, and all-abstain behavior. Missing and not-run are not represented as numeric zeros. Inject failures so error accounting is exercised before reporting success rates.

## 4. CLI acceptance targets

```bash
# Implement these commands in I0-I3; they do not exist in this docs-only revision.
python -m actionfoundry doctor --profile core
python -m actionfoundry run --config configs/mock.yaml
python -m actionfoundry replay --run runs/<run-id> --verify
python -m actionfoundry summarize --run runs/<run-id>

# Add these only after the simulator adapter passes I4.
python -m actionfoundry doctor --profile robosuite
python -m actionfoundry run --config configs/phase1-lift.yaml
python -m actionfoundry diagnose --run runs/<run-id> --max-snapshots 100
```

The example in [phase1-lift](examples/phase1-lift.yaml) supplies design values. Implement a strict parser, resolve and validate operational thresholds, then promote the example to a runnable config. A production/test run refuses unresolved thresholds, missing pins, non-frozen split manifests, or an invalid controller contract. `doctor` reports installed versions and capabilities, not an unconditional success banner.

Exit codes: 0 completed; 2 invalid configuration/contract; 3 unavailable dependency/capability; 4 runtime/infrastructure failure. Ordinary task failure is data in a successfully completed evaluation, not a CLI infrastructure error.

## 5. Reproducibility and safety of development

Commit resolved small manifests and result summaries, not API keys, raw personal data, model weights, videos, large datasets, or virtual environments. Store artifacts outside Git with checksums. Network services and paid requests are off by default; untrusted model repositories requiring arbitrary remote-code execution need separate review.

The MIT license covers ActionFoundry-authored material. Inventory third-party licenses before importing code, assets, or checkpoints; dataset licenses may differ from repository licenses. This specification does not authorize real-robot execution.

## 6. Definition of ready for P2

A new machine can install the pinned P0/P1 environments, run the documented fixture and simulator smoke test, replay a saved decision trace, reproduce S0/S1/D0 pilot reports, and see precisely why each G0–G2 gate passed or failed. Counterfactual metrics are present only if restoration is validated. All performance claims link to raw run manifests and commands.

## Suggested coding-agent opening task

> Read AGENTS.md and the v0.1 specs. Implement I0-I3 only: typed contracts, canonical action compilation, a deterministic NumPy mock environment, bounded proposal/selection/execution loop, append-only traces, replay, and tested metrics. Add passing CPU tests and a runnable mock config. Do not add model training, remote APIs, simulator downloads, or placeholder VLA integrations. Report actual test commands and failures, then identify the next concrete I4 dependency task.


## 7. OpenWAM integration boundary

Do not vendor OpenWAM into the core package. Use an isolated worker/environment with its own Python/GPU dependency lock and communicate through the same versioned local protocol used for simulator workers. Pin the upstream repository/checkpoint/config and preserve its resolved Hydra configuration with each experiment.

The first adapter supports inference only: reset/session identity, observation conversion, action/state normalization, action-chunk output, timing, and checkpoint provenance. Training remains an explicit OpenWAM job. If later experiments add an ActionFoundry decision head, keep the head/config/checkpoint separately identifiable so frozen-backbone and fine-tuned results cannot be confused.

Before any joint training, reproduce one official/local OpenWAM benchmark baseline within tolerance and audit action slots, masks, temporal horizon, camera preprocessing, normalization, and inference horizon [R27](references.md#r27). A failed reproduction blocks claims about ActionFoundry improvements but does not block P0–P2.
