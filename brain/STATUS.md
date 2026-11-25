# Status

> Single source for: current verified state and blockers.
> Task status lives only in [TASKS.md](TASKS.md); the next action lives only in [HANDOFF.md](HANDOFF.md).

- Last updated: 2026-11-25 (Claude Code)
- Phase: T-003 (tech stack); T-004 done (Gate G1, D-007); T-005 done (D-009); T-006 not started
- Blockers: none for research. Hardware test blocker for later touch tests (T-009 onwards): Xeneon Edge touch does not work, cause unknown.

## Capture Feasibility (T-005, documentary only, nothing tested)
- Windows 11, macOS, native X11: conditional. Wayland: unknown; no general E5-compliant passive background path demonstrated for the live viewer. Linux stays `Unverified`.
- T-007 candidates (D-009): Raw Input preferred, low-level hook as comparison; macOS listen-only tap variant justified.
- Open until tested: delivery with an elevated foreground window (both Windows APIs), recovery after transitions, macOS secure-input behaviour on the target build.
- Known documented limits: Windows protected desktops, silent hook removal; macOS Secure Event Input. "Enabled" does not prove events are being received.
- Details: [report](../docs/research/T-005-global-capture.md) (sections 1-10 current).

## Open Hardware Questions
| Question | Needed before |
|---|---|
| Cause of the missing touch function (Windows reports no digitizer) | T-009 touch tests |
| Exact Mac model (Apple M4) and macOS version | macOS runs of T-007/T-009 |
| Is Windows build 26300 a preview channel; is a stable build testable? | T-007 interpretation |
| Advantage360 SmartSet and Voyager were not connected during the inventory; firmware versions unknown | T-006 (firmware), T-007 |

## Verified State
- `AGENTS.md` holds shared agent rules incl. task history rules and shared workflow commands; `CLAUDE.md` imports it via `@AGENTS.md`.
- Documentation language: English (D-002); all brain files are in English.
- `brain/` contains the 7 memory files.
- `.gitignore` excludes `brain/.obsidian/`, `.env*` (except `.env.example`), OS files.
- Git: setup, T-002, the T-003 plan, T-004 and T-005 are committed on `main` (user-approved commits and pushes, 2026-11-25).
- Research reports: [T-004 evaluation frame](../docs/research/T-004-evaluation-frame.md), [T-005 global capture](../docs/research/T-005-global-capture.md).
- No source code, build system or tests exist yet.
- [PROJECT.md](PROJECT.md) is the approved requirements baseline (all review findings handled, see [TASKS.md](TASKS.md)); tech stack and architecture undefined ([ARCHITECTURE.md](ARCHITECTURE.md), T-003).
