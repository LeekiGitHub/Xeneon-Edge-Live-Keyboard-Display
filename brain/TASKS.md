# Tasks

> Single source for: task status, priorities and acceptance criteria.

> IDs: `T-###`, never reused. Priority: P1 (high) - P3 (low).
> Status: `todo` | `active` | `review` | `blocked` | `done`.
> Move finished tasks to "Done" with their acceptance criteria and verification results (see AGENTS.md, section 5).

## Active
_None._

## Backlog

### T-003 Decide tech stack and initial architecture
- Priority: P2 | Status: todo (unblocked, not started) | Owner: OPEN
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

### T-002 Define project goals and requirements
- Priority: P1 | Status: done (2026-11-24, approved by the Product Owner) | Owner: user + Claude Code (author), Codex (independent reviewer)
- Acceptance criteria:
  - [x] Purpose, goals, constraints, success criteria in [PROJECT.md](PROJECT.md) approved by user (final approval 2026-11-24)
- Verification: draft independently reviewed by Codex (7 findings); user decisions incorporated and all scenarios confirmed; explicit final approval given 2026-11-24. First-launch UI language decided (OS language if supported, else English). Documentation-only task, no tests.

#### Independent requirements review (2026-11-24, Codex)

The draft covers the main product goals and separates a first development milestone, an MVP and later work. It is suitable for identifying T-003 research questions. Resolve the high findings and record the disposition of the remaining findings before approving the draft as the baseline for T-003. The review used the consolidated draft and the author handoff as evidence for user confirmations.

**High**

1. **R-01 — Capture and data lifecycle are underspecified.** [NF-Privacy](PROJECT.md#nf-privacy-and-security-c) (lines 75–81), [NF-Data](PROJECT.md#nf-data-c) (97–100), and [acceptance](PROJECT.md#6-success-criteria) (137–138, 146, 149). The draft distinguishes live capture from exercises, permits individual key timings only inside active exercises, and requires saved results and progress. It leaves open whether individual timings captured during an exercise may survive its end, what data may be collected when both modes are active, and what pause or loss of exercise focus stops. The live-view checks cover visible state and absence of history, but the exercise boundary is missing from acceptance. Define allowed data per mode, including whether individual timings can persist, retention and deletion if allowed, what exercise start/end and focus loss mean, and the observable behavior of pause/restart. Add checks for disabled capture and exercise-only input, pause via button and hotkey, session end, and absence of stored or transmitted global input. Preserve the confirmed restriction and leave aggregate sufficiency for T-003. Decide the user-facing indication of the limits of global capture independently of any future password-field detection.
2. **R-02 — M1 performance pass/fail contradicts the stated target status.** [NF-Performance](PROJECT.md#nf-performance-c-preliminary-quality-targets-feasibilitymeasurement-in-t-003-not-hard-limits) (83–86) calls 30 ms and 60 FPS preliminary and explicitly says they are not hard limits. [M1 acceptance](PROJECT.md#m1-acceptance) (135–138) requires meeting the targets. Align the acceptance wording with the confirmed preliminary status. Ask T-003 to propose a reproducible measurement plan defining timing endpoints, reference hardware and refresh rate, workload, sampling and reporting. Keep observable correctness checks for simultaneous keys and clearing highlights after interruptions separate from numerical targets.

**Medium**

3. **R-03 — Milestone and acceptance coverage is uneven.** [Setup A](PROJECT.md#2-target-hardware-display-usage) (18–22), [F1–F2](PROJECT.md#3-functional-requirements) (25–34), [NF-Platforms](PROJECT.md#nf-platforms-c) (92–95), and [milestones and acceptance](PROJECT.md#5-scope-and-milestones) (108–155). Windows and fullscreen Setup A apply to both milestones but are absent as explicit acceptance conditions. The table leaves most M1 rows blank under M2, while acceptance says “M1 plus.” The M1 live-view check can pass without proving foreground-app operation or that normal typing is unaffected. The M1 speed-test check can pass without exercising timing, random words or error evaluation, while F2 also mentions progress and M2 owns long-term statistics. Declare the table cumulative and map confirmed requirements to milestone checks. Include a Windows fullscreen secondary-display scenario with another app focused, and define the minimum exercise result in M1 versus trends and filters in M2. Clarify what “all functions” operable by touch and mouse covers (152), since the exercises explicitly require physical typing.
4. **R-04 — Several M2 checks can pass with less than the functional requirements.** [F3–F5](PROJECT.md#3-functional-requirements) (36–50) and [M2 acceptance](PROJECT.md#m2-acceptance-m1-plus) (146–155). Naming Endless Training does not test continuous content or deliberate session termination. The text/quote check covers source and license for quotes only, while F4 covers all content and also requires attribution. A beginner path and some adaptive exercises may pass without a shared definition of a complete path. Add short, observable scenarios for the confirmed behavior and agree on the minimum path coverage and evidence of adaptation. Keep lesson methodology and algorithm design open where already delegated. Distinguish documentation presence or a successful install/launch check from the user's judgment of clarity.
5. **R-05 — Marker definitions and usage disagree.** [Legend](PROJECT.md#project) (5–7), [F8](PROJECT.md#f8-statistics-and-progress-c) (65–66), [NF-Privacy](PROJECT.md#nf-privacy-and-security-c) (78), [scope](PROJECT.md#5-scope-and-milestones) (128), [M2 acceptance](PROJECT.md#m2-acceptance-m1-plus) (150–152), and [open product details](PROJECT.md#9-open-product-details-decide-later) (176–177). The prior author handoff records four confirmations. The M2 layout filter still carries `[P: …]` in F8, the M1 i18n foundation is `[P]` in the scope note, and the agreement to define touch-target size during planning is `[P]` in M2 acceptance. Individual key-timing handling is already `[C]` in NF-Privacy and T-003 questions. Relabel the three stale occurrences `[C]` and remove the header override once corrected. Keep the actual touch-target size `[O]`, aggregate sufficiency open for T-003, and unconfirmed statistics metrics and KLE compatibility evaluation as proposals. The current legend defines `[C]`, `[P]` and `[O]`; `[A]` is neither defined nor used. Ask the user whether `[A]` is intended and what it means before introducing it. Clarify how section markers apply to subordinate bullets, where proposals and open decisions can coexist with confirmed features.
6. **R-06 — Define the decisions T-003 can rely on.** [NF-Platforms/NF-Data](PROJECT.md#nf-platforms-c) (92–100), [technical questions](PROJECT.md#8-open-technical-questions-for-t-003-not-decided) (165–174), and [product details](PROJECT.md#9-open-product-details-decide-later) (176–177). Windows is mandatory and other platforms are preferences. The draft leaves “reasonable effort” undefined, places license choice in both lists, and gives a later setup and optional sync broad architectural weight without an explicit limit. Record Windows plus opt-in capture and primary-display usability as decision constraints, cross-platform support as a comparison preference, and later items as compatibility considerations. Assign license approval to the user while T-003 can assess dependency and distribution constraints. Keep KLE evaluation, storage format and event feasibility as research questions. Treat any unresolved product choice affecting the stack as an explicit assumption or decision for the user.

**Low**

7. **R-07 — First-launch language default is ambiguous.** [NF-i18n](PROJECT.md#nf-i18n-c) (88–90) names English as the international default, while [open product details](PROJECT.md#9-open-product-details-decide-later) (176–177) leave the initial UI language open. Clarify whether English is a fallback or the first-launch default, and verify the agreed behavior and independent language selection in M2 acceptance.

#### Disposition of findings (2026-11-24, Claude Code)

| ID | Handling | Where |
|---|---|---|
| R-01 | Adopted per user decisions: two separated data paths, auto-pause on focus loss, aggregates only, no retention of events/timings, approval needed for exceptions; scenarios PS-1..PS-9 added | NF-Privacy, section 6 |
| R-02 | Adopted: 30 ms/60 FPS are preliminary targets, reported not pass/fail; reliable, fluid, immediate visualization binding; method and limits go to T-003 for approval | NF-Performance, M1 acceptance |
| R-03 | Adopted: table is cumulative; Windows fullscreen secondary-display scenario with other app focused; minimum M1 speed-test result vs. M2 trends/filters; touch/mouse scope limited to non-typing functions. Scenarios are `[P]` pending final approval | Sections 5, 6 |
| R-04 | Adopted as `[P]` scenarios: endless/continuous content and user-ended session, attribution for all content, observable adaptation, docs presence and clean-install check vs. subjective clarity. Minimum path coverage deferred to M2 planning | M2 acceptance |
| R-05 | Adopted: all confirmed items marked `[C]`, header override removed, `[A]` removed (unused, undefined), marker scope rule added | Header, F8, NF-i18n |
| R-06 | Adopted: roles and phase assignment of open items; license decided by user; sync/Setup B demoted to compatibility considerations; "reasonable effort" criteria assigned to T-003 | NF-Data, section 8 |
| R-07 | Adopted at final approval: first launch follows a supported OS language, else English; manually changeable | NF-i18n, M2 acceptance |

Final approval (2026-11-24, user): R-03/R-04 scenarios and the UI note on capture limits confirmed and relabelled `[C]`.
