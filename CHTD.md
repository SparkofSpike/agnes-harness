# CHTD — Co-Harness Targeted Distillation

Draft charter for review. Baseline `main@cde816b8`; development branch `feat/chtd`.

## Name and description

**Name.** CHTD — Co-Harness Targeted Distillation
**Chinese name (reference).** 共轨靶向蒸馏

**Description.** Teacher and student LLMs share one harness and run the same task loop, so the teacher's agentic capability is targetedly distilled into the student while distractors unrelated to that capability are automatically stripped.

**Description (short, for repository metadata).** Teacher and student LLMs share one harness: the teacher's agentic capability is targetedly distilled into the student, and unrelated distractors are stripped automatically.

## Problem

Agentic capability is usually distilled across harnesses. The teacher runs in its own scaffolding — its own tool schemas, step limits, retry policy and context assembly — and the student is trained on the transcripts that scaffolding produced. Two failures follow.

1. **The student learns the scaffolding, not the capability.** Trajectories carry the teacher harness's artifacts: its prompt scaffolding, its tool-call formatting, its stopping behaviour. Once the student runs behind a different harness, exactly those artifacts stop matching, and the transfer does not hold.
2. **The evaluation rewards the wrong thing.** Aggregate metrics — length, latency, token spend, success on unrelated subtasks — move together with the target capability. An improvement is then attributed to the wrong cause, and the distilled policy drifts off the axis the work claims to improve.

## Approach

Three commitments define CHTD.

1. **Co-harness execution.** Teacher and student run the *same* harness: same tool set, same loop, same step and stop rules, same context assembly. The only difference is the model behind the loop. Rollouts land on one shared, append-only ledger, so a teacher step and a student step are directly comparable.
2. **Targeted distillation.** The capability to transfer is named up front — multi-step decomposition, tool selection, error recovery, or another axis. The supervision signal is built from aligned segments of the shared loop, not from whole transcripts.
3. **Distractor stripping.** Metrics that correlate with the target capability without being caused by it are identified and removed, from both the training objective and the reported evidence. What remains is the signal the work claims to improve.

## Why this baseline

Agnes Harness already provides the pieces co-harness execution needs:

- **One loop, pluggable models.** Teacher and student are two providers behind the same runtime loop.
- **Append-only session records.** Trajectories are exact and replayable rather than reconstructed after the fact.
- **Fork and rewind.** The same prefix can be continued under a different policy — same-start counterfactuals without re-running the world.
- **Plugins and Skills.** Tasks and tools vary while the harness stays fixed.

The harness is therefore not merely a container for the experiment; it is the invariant that makes the comparison meaningful.

## Open questions

- What is the first target capability, and how is it measured?
- What form does the supervision signal take — segment-level labels, preference pairs, or something else?
- Where does "targeted" selection come from: a human-named capability, or an automatic ablation over candidate axes?
- How is distractor stripping validated — held-out probes, or an external capability benchmark?
- Does the student train inside the loop (online), or against recorded trajectories (offline)?

## Next step

Agree on the first target capability and its measurement before implementation begins.
