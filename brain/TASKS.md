# Tasks

> Single source for: task status, priorities and acceptance criteria.

> IDs: `T-###`, never reused. Priority: P1 (high) - P3 (low).
> Status: `todo` | `active` | `review` | `blocked` | `done`.
> Move finished tasks to "Done" with their acceptance criteria and verification results (see AGENTS.md, section 5).

## Active
- none

## Backlog
### T-002 Define project goals and requirements
- Priority: P1 | Status: todo | Owner: user + agent
- Acceptance criteria:
  - [ ] Purpose, goals, constraints, success criteria in [PROJECT.md](PROJECT.md) approved by user

### T-003 Decide tech stack and initial architecture
- Priority: P2 | Status: blocked (by T-002) | Owner: OPEN
- Acceptance criteria:
  - [ ] Stack and components in [ARCHITECTURE.md](ARCHITECTURE.md), rationale in [DECISIONS.md](DECISIONS.md)

## Done
### T-001 Set up shared agent memory
- Status: done (2026-11-24, closure accepted by user) | Owner: Claude Code (implementer), Codex (reviewer)
- Acceptance criteria: The original criteria were not retained in the repository and are not reconstructed here. The user accepted closure based on the verification below.
- Verification (2026-11-24):
  - `AGENTS.md` holds the shared rules; `CLAUDE.md` contains only `@AGENTS.md` and was loaded by Claude Code.
  - All 7 brain files exist; all relative Markdown links in `brain/` resolve.
  - `git check-ignore -v`: `brain/.obsidian/`, `.env`, `.env.*` ignored; `.env.example` not ignored.
  - Independent review by Codex: no severe findings. P2 resolved (Obsidian approval in D-001; single source per information in STATUS/TASKS/HANDOFF). P3 (merge LESSONS.md categories) deferred by user.
  - Finalization: English documentation language (D-002), task history rules and shared workflow commands added to `AGENTS.md`.
  - No project or build tests exist.
