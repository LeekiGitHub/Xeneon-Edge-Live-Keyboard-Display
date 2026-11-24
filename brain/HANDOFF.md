# Handoff

> Single source for: latest session summary and the next action. Overwrite on each handoff.
> Separate facts from assumptions. Task status belongs in [TASKS.md](TASKS.md), current state in [STATUS.md](STATUS.md).

- Date: 2026-11-24
- From: Claude Code (requirements author) -> To: planner/implementer for T-003
- Task completed: T-002 (approved and closed by the user). Next: T-003 Decide tech stack and initial architecture

## Work Completed
- Final approval of [PROJECT.md](PROJECT.md): scenarios and UI note confirmed, markers updated to `[C]`; first-launch UI language decided (OS language if supported, else English; D-005). T-002 moved to Done with criteria and review history.

## Verification
- Documentation-only; markers and links re-checked by search. No code or tests exist. Nothing committed.

## T-003 Input (see PROJECT.md section 8 for the full list)
- Facts: Windows mandatory (M1/M2); cross-platform preferred if feasible; opt-in system-wide capture with strict separation from training data (NF-Privacy, D-003); aggregates only unless approval; performance targets preliminary.
- T-003 must deliver for approval: stack recommendation with independent review (D-004), feasibility of global input capture per OS, event model and lost-event handling, performance measurement method with reference hardware and proposed limits, aggregate sufficiency for adaptive training, storage format, keyboard-profile format (KLE evaluation), "reasonable effort" criteria, dependency license constraints.
- Remaining `[O]`/`[P]` items are intentionally deferred to their phases.
- Next action: user starts `plan` for T-003. Recommended: Advanced tier, high effort (cross-platform input capture feasibility), independent review by the other agent.