# Handoff

> Single source for: latest session summary and the next action. Overwrite on each handoff.
> Separate facts from assumptions. Task status belongs in [TASKS.md](TASKS.md), current state in [STATUS.md](STATUS.md).

- Date: 2026-11-25
- From: Claude Code (researcher, T-006) -> To: Codex (T-007)
- Current task: T-007 (Windows capture prototype), not started
- Recommended: Codex, Standard tier / high effort (platform behaviour with privacy and reliability impact; escalate only with evidence)

## Work Completed
- T-006 accepted and closed (D-010). Added to the [report](../docs/research/T-006-event-model.md): reported keyboard configurations (12.1), preliminary decisions (10.1), configuration notes for scenarios (11).
- Updated TASKS.md, STATUS.md, DECISIONS.md, LESSONS.md, this file. Commit and push to `origin/main` approved by the user.

## Verification
- Documentation only. No installs, prototypes, code, tests or device or system changes; no `VER` results.
- No independent review of T-006 (waived).
- User statements on hardware are unverified.

## Inputs for T-007
- [T-005 section 8](../docs/research/T-005-global-capture.md#8-prototype-claims-and-test-method): claims, trial order, test rules 1-10; Raw Input preferred, low-level hook as comparison (D-009).
- [T-006 section 11](../docs/research/T-006-event-model.md#11-controlled-test-scenarios-for-t-007): WP-16..WP-28, MP-10, MP-11, priorities, configuration notes. Keep keyboards as configured.
- Configurations: [STATUS.md](STATUS.md). Constraints: [TASKS.md](TASKS.md) plan rules and T-007.

## Critical Assumptions to Check During Preparation
Check source and testability; do not treat as facts.
1. Capture leaves other apps' input and dead keys unaltered (T-006 3.3, WP-27; KO-2).
2. Position codes are stable across host layouts; US backslash and non-US `#` share a scan code (3.3, WP-16).
3. AltGr, Num Lock, Pause and Print Screen produce extra or multi-part codes; partly `SEC`/`HYP` (WP-17).
4. Knob and media signals reach each candidate path, possibly outside keyboard scan codes (WP-18, WP-23).
5. Key-state queries can clear stale keys; limits per T-005 (T-006 6.4, WP-25).
6. Device identity is reliable enough for input-device selection (RC-05, WP-26).
7. Duplicate codes on one device cannot be told apart (`HYP`, WP-22).
8. Lemokey vendor firmware source (`d3c413b`) matches the user's unit (unproven).
9. Learn mapping in a focused window yields the same codes as background capture (WP-28).

## Open Issues
- Unknown: all firmware versions, Advantage360 profile and remapped keys, Voyager bindings; ask before the respective tests.
- Report gaps (section 13): Kinesis default layout, official Lemokey manual, ZSA Oryx/factory layouts.
- Requirement conflicts without decision: RC-03, RC-07..RC-10; preliminary decisions become final at Gate G2.

## Next Action (only on user request)
1. Codex prepares T-007: scope, acceptance criteria, check of the assumptions above, test protocol, targeted questions to the user.
2. Wait for approval before creating prototype code, running it, installing anything or changing settings (including the OS input language). macOS variant only after Windows, with separate approvals.
