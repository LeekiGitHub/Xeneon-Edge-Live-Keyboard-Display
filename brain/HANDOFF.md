# Handoff

> Single source for: latest session summary and the next action. Overwrite on each handoff.
> Separate facts from assumptions. Task status belongs in [TASKS.md](TASKS.md), current state in [STATUS.md](STATUS.md).

- Date: 2026-11-25
- From: Claude Code (T-005 consolidation) -> To: researcher for T-006
- Current task: T-005 closed (accepted, D-009); T-006 ready, not started

## Work Completed
- Codex review findings consolidated into sections 1-10 of the [T-005 report](../docs/research/T-005-global-capture.md); review history kept (section 11), dispositions in section 12.
- T-005 moved to Done; D-009 recorded.
- Claude Code began a `review T-005` request on its own report (reading only); it was interrupted and produced no findings.

## Verification
- Documentation only. No installs, prototypes, code or tests; no `VER` results.
- Checkpoint (2026-11-25): changed files limited to `brain/` and `docs/research/`; review text unchanged apart from a marked history note; citation IDs, relative links, whitespace and personal-data patterns checked. Sources added by Codex were not re-fetched.
- Commit and push to `origin/main` approved by the user; confirm sync with `git status`.

## Open Issues
- Report section 9 (gaps) and section 7 (Product Owner findings, suggested for Gate G2).
- Wayland verdict recorded as `Unknown` and WP-13..WP-15, MP-08/MP-09 numbered by Claude Code (report section 12).
- Hardware questions: [STATUS.md](STATUS.md). Voyager and Advantage360 firmware versions matter for T-006.

## Next Action: T-006 (start only on user request)
- Recommended: Claude, Standard tier, high effort (multi-source research with requirement impact).
- Scope and completion criteria: T-006 in [TASKS.md](TASKS.md). Report: `docs/research/T-006-event-model.md`.
- Read first: T-005 report matrix rows C07-C13 and the status/privacy parts of section 7; PROJECT.md F1, F6, NF-Privacy. Load further sections only as needed.
- Carry over: model enabled / registered / receiving as separate states; lost events must clear highlights; raw repeat and injection behaviour are unverified.
- Use current official sources with URL and access date; nothing is `VER` without a test. No stack preference, installs or prototypes.
