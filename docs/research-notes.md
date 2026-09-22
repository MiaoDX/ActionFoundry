# Research synthesis and provenance

## What the project preserves

The discussion began with Jev's structured decision interface, broadened to robot control, then examined relative moves, action chunks, value/critic selection, and intermediate interfaces including Show-Harness. The durable question is not whether one reference should replace every other architecture. It is whether making executable alternatives explicit helps a robot choose, abstain, or recover under controlled comparisons.

The accepted engineering choices are in [decisions](decisions.md). The research hypotheses remain open, and the [reference landscape](references.md) contains competing and adjacent approaches rather than a single claimed lineage.

## Evidence provenance

This revision is a new primary-source-checked synthesis of the visible discussion and repository. The separate Deep Research session's full report was not available as a retrievable artifact in this editing session; it has **not** been imported or represented as a recovered report. The earlier repository notes were a discussion synthesis, not a documented literature review. This source map supplies explicit traceable references without inventing missing research output.

No ActionFoundry runtime, training, or manipulation experiment was run for this design revision. Literature claims are attributed to their authors, not independently reproduced.

## Corrections that affect design

**Candidate selection is established prior art.** QT-Opt, Implicit Behavioral Cloning, SayCan, and especially V-GPS prevent treating proposal-plus-ranking alone as a novel contribution. The open question is the measured benefit of the particular interface, supervision, generalization behavior, or cost/quality tradeoff [R04](references.md#r04) / [R05](references.md#r05) / [R06](references.md#r06) / [R07](references.md#r07).

**Relative action is not one representation.** Sequential deltas, current-pose-anchored trajectories, discrete spatial targets, and semantic moves are distinct. Action chunking is a temporal choice independent of whether the coordinates are absolute or relative. ACT's original joint-position interface and PerAct's spatial action representation should not be relabeled as relative end-effector control [R11](references.md#r11) / [R12](references.md#r12) / [R13](references.md#r13) / [R14](references.md#r14).

**A typed probability head does not establish calibration.** Valid syntax is not correct semantics; choosing the most likely option is not predicting its physical success. Auxiliary success/risk heads need separate outcome labels and evaluation. Unchosen demonstration actions are not automatically negative examples [R09](references.md#r09) / [R18](references.md#r18) / [R20](references.md#r20).

**Fast/slow terminology is not a common implementation.** A WAM may infer action chunks without rolling out explicit future video; a robotic System 1 can be a diffusion action generator. Optional prediction and candidate scoring therefore require actual interfaces, not terminology-based assumptions [R24](references.md#r24) / [R25](references.md#r25).

**Show-Harness is a comparator, not a new mandate.** Its interpreter-based action interface belongs in the action-representation study. It does not displace direct policies, continuous chunks, or value-guided approaches in the workbench [R08](references.md#r08).

**Reference status matters.** AVP is withdrawn as of the source-check date, so its earlier cited performance is no longer used to support the design. Other unverified earlier claims remain leads rather than evidence [R26](references.md#r26).

## Remaining research questions

Where does failure originate: insufficient proposals, incorrect value estimates, missing observations, bad control interpretation, or recovery? Does a scorer transfer across proposal sources or merely learn their ordering? Does its benefit survive equal action support, controller, supervision, and wall-clock budgets? Does an apparent state-based advantage survive visual perception and model latency? How much of the useful signal should come from a WAM's action expert, its world representation, explicit candidate-conditioned prediction, or a separate decision head? These questions determine the next stage; the project is not required to produce a Jev-like model if simpler baselines solve the problem.

The phased plan starts with falsifiable contracts and diagnostics. Positive, negative, saturated, and blocked experiments are all legitimate outputs when accompanied by reproducible traces.


## World/action model correction

The earlier discussion used “world model” too narrowly as a learned substitute for simulator branch rollout. OpenWAM makes the broader design space explicit: a WAM can jointly organize video/world representations and action generation, with dedicated action capacity and explicit information flow between world and action components [R27](references.md#r27). ActionFoundry therefore treats WAMs as a parallel research axis rather than a late simulator replacement.

The simulator remains useful for privileged counterfactual labels because it can branch from a saved state. The learned WAM has different roles: proposal source, representation source, candidate-conditioned predictor where supported, and eventually a jointly trained backbone with decision supervision. These roles must be evaluated separately.

This also changes the environment view. robosuite/MuJoCo remains the first mechanism-debugging backend, not the only simulator. LIBERO is the first visual transfer benchmark; RoboTwin 2.0 is the preferred richer WAM-oriented follow-up; CALVIN is a later long-horizon/relative-action target; GPU-parallel systems such as ManiSkill or Isaac Lab are scale options rather than immediate dependencies.

A particularly important research distinction is **predictive fidelity versus decision utility**. A representation that predicts pixels or states accurately may still discard distinctions needed to rank actions, while a less faithful predictor may preserve decision-relevant structure. ActionFoundry should measure both instead of using reconstruction quality as a proxy for control value.
