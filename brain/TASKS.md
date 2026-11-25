# Tasks

> Single source for: task status, priorities and acceptance criteria.

> IDs: `T-###`, never reused. Priority: P1 (high) - P3 (low).
> Status: `todo` | `active` | `review` | `blocked` | `done`.
> Move finished tasks to "Done" with their acceptance criteria and verification results (see AGENTS.md, section 5).

## Active

### T-003 Decide tech stack and initial architecture
- Priority: P2 | Status: active (plan approved, D-006; Gate G1 passed, D-007; T-005 done, next T-006) | Owner: Claude Code (planner)
- Acceptance criteria:
  - [ ] Stack and components in [ARCHITECTURE.md](ARCHITECTURE.md), rationale in [DECISIONS.md](DECISIONS.md)
- Additional criteria (from [PROJECT.md](PROJECT.md) section 8 and D-004; approved with the plan 2026-11-25):
  - [ ] Every T-003 open item in PROJECT.md section 8 is answered, or explicitly deferred with Product Owner approval
  - [ ] Findings that drive the decision carry an evidence level and source
  - [ ] Independent review by the non-authoring agent, findings dispositioned
  - [ ] Product Owner approval of the recommendation

#### Plan rules (approved)
- One subtask per session, in dependency order. Stop at each gate for the Product Owner.
- Evidence levels: `VER` verified here (environment, steps, result) | `DOC` official documentation (URL, version, access date) | `SEC` secondary/community source | `HYP` assumption.
- Reports: `docs/research/T-###-<topic>.md` (English) with sources, access date, evidence level and limitations; `brain/` keeps summaries and links; load reports only when needed.
- No stack preference before T-008. Prototypes need explicit approval, live under `prototypes/`, are separate from the app and never its base.
- Installing anything needs approval first, with a list of what, why and size.
- Prototypes store no system-wide key histories or private input; capture prototypes are started manually and use no network; reports contain findings, not typed input.
- Real test systems: Windows and macOS. Without verification (currently Linux) no support is promised.
- Model plan is a flexible recommendation: Standard by default; agents recommend (never perform) a model switch, with a reason; respect both providers' usage quotas.

#### Technical risks (all `HYP` until researched; impact = effect on architecture or requirements)
TR-2, TR-3 and TR-4: desk research in the [T-005 report](../docs/research/T-005-global-capture.md); still unverified.

| ID | Risk | Impact | Task |
|---|---|---|---|
| TR-1 | Firmware-handled keys (layer/Fn, macros, onboard remaps) may send nothing or identical codes, so a faithful live view of every physical key may be impossible from OS events | Critical (may change requirements) | T-006, T-007 |
| TR-2 | Global capture differs fundamentally per OS: Wayland may offer no passive unprivileged path; macOS needs user permissions and blocks some input | Critical | T-005 |
| TR-3 | Windows capture limits: elevated windows, secure desktop, lock screen, silently dropped hooks, lost key-up, knob/media events | Critical | T-005, T-007 |
| TR-4 | Capture code resembles a keylogger: leaks via logs, crash dumps, IPC or dependencies; antivirus false positives | High | T-005, T-012, T-013 |
| TR-5 | Touch on a secondary display may steal focus or move the cursor; mixed DPI, fullscreen placement | High | T-008, T-009 |
| TR-6 | UI framework may throttle unfocused windows, miss latency targets or use too many resources | High | T-008, T-009 |
| TR-7 | Training path needs position plus character (dead keys, AltGr, IME); framework key APIs may be insufficient | High | T-006, T-009 |
| TR-8 | Aggregates may not suffice for adaptive training (privacy vs. function) | Medium-High | T-010 |
| TR-9 | Word lists, texts, fonts or dependencies may conflict with the public repo or the chosen license | Medium (word lists block M1) | T-004, T-012 |
| TR-10 | Signing/notarization costs and permission-sensitive packaging vs. "no mandatory paid services" | Medium | T-012 |
| TR-11 | Global capture is hard to test automatically; weak tooling raises AI-agent iteration cost | Medium | T-008, T-012 |

#### Subtasks (`todo` unless stated; Tier = recommended agent, model tier / effort)
Order: T-004 -> T-005, T-006 -> T-007 -> T-008 -> T-009 -> T-010, T-011, T-012 -> T-013. T-010 and the content part of T-012 only depend on T-004 and may be pulled forward.

T-004 and T-005 are done (see Done). Assess options only with the T-004 [evaluation frame](../docs/research/T-004-evaluation-frame.md).

##### T-006 Event model and keyboard behaviour
- Tier: Claude, Standard / high | Depends: T-004
- Questions: How do physical position, scan/HID code, logical key and character relate per OS? What do the profiled keyboards (Lemokey P1 Pro, Advantage360 SmartSet, Voyager, Moonlander without test hardware, generic TKL) actually send (layers, thumb keys, duplicates, knob)? What can the live view not know, and how should profiles express that? Which limits are stack-independent (KO-1) and affect product requirements? What does the focused training path need?
- Done when:
  - [ ] Stack-independent conceptual event model, incl. lost-event handling and the privacy boundary between both data paths
  - [ ] Limitation list per keyboard model; requirement conflicts raised to the Product Owner

##### T-007 Prototype P1: Windows capture
- Tier: Codex, Standard / high | Depends: T-005, T-006
- macOS variant (approved for the plan, D-007): only if prior research justifies it; after Windows; separate approvals for installs and for execution. Record Mac model and macOS version first.
- Questions: Verify TR-1 and TR-3: candidate APIs with a foreground app, elevated window, UAC/lock screen, key bursts, hold/repeat, AltGr, knob; behaviour of installed antivirus.
- Preference: use tools already shipped with Windows; no install unless shown necessary.
- Input from T-005 (D-009): Raw Input preferred test candidate, low-level hook as comparison; claims, trial order and test rules in [report section 8](../docs/research/T-005-global-capture.md#8-prototype-claims-and-test-method).
- Done when:
  - [ ] Test protocol with `VER` results on the user's hardware
  - [ ] Go/no-go and requirement clarifications decided (Gate G2)

##### T-008 Stack candidates (desk research)
- Tier: Claude, Standard / high | Depends: T-004, T-007
- Questions: Which candidates (web-based, native cross-platform, Windows-only as reference) pass the knock-out criteria? Maintenance, license, native integration, window/touch control, UI quality, i18n, testing, packaging, AI-agent suitability?
- Done when:
  - [ ] Longlist scored against T-004 criteria with sources
  - [ ] At most 2-3 finalists and a prototype install list approved (Gate G3)

##### T-009 Prototype P2: window, touch, rendering, performance
- Tier: Codex, Standard / high | Depends: T-008
- macOS variant (approved for the plan, D-007): only if prior research justifies it; after Windows; separate approvals for installs and for execution.
- Blocker for touch tests: Xeneon Edge touch does not work ([STATUS.md](STATUS.md)). Interpret performance results against the high-end test PC.
- Questions: Verify TR-5, TR-6, TR-7 with identical scenarios per finalist. How are latency, frame rate and resource use measured reproducibly?
- Done when:
  - [ ] Measurement method defined before measuring (endpoints, reference hardware, workload, sampling)
  - [ ] `VER` results per finalist on the Xeneon Edge
  - [ ] Numeric performance limits proposed to the Product Owner

##### T-010 Training data sufficiency
- Tier: Claude or Codex, Standard / high | Depends: T-004
- Questions: What evidence supports touch-typing and adaptive-practice methods? Which metrics do they need? Do aggregates suffice, and which granularity is data-minimal?
- Scope: only what affects data model and privacy; algorithm, lessons and finger assignment stay in M2 planning.
- Done when:
  - [ ] Sources rated by evidence strength
  - [ ] Aggregate data set proposed; any exception submitted for explicit approval

##### T-011 Persistence, profile format, i18n
- Tier: Claude or Codex, Standard / medium | Depends: T-006, T-008, T-010
- Questions: Storage technology, schema versioning, export/import? Own profile format vs. KLE compatibility? i18n mechanism?
- Done when:
  - [ ] Options compared and recommendation per item
  - [ ] Profile format shown to express split geometry, knob and the four independent concepts (F6) on paper

##### T-012 Licensing, distribution, testability
- Tier: Claude or Codex, Standard / medium | Depends: T-004 (content), T-008 and T-009 (distribution)
- Questions: Which DE/EN word lists, text sources and fonts are redistributable? Dependency license obligations? Windows install, signing, antivirus; effort for macOS/Linux? What can be tested in CI?
- Done when:
  - [ ] Source/license table with attribution duties
  - [ ] Distribution and test strategy for the finalists, incl. costs

##### T-013 Synthesis, review, approval
- Tier: Claude, Advanced / high if the extra effort is justified; final architecture review: Codex, Standard / high | Depends: T-004..T-012
- Done when:
  - [ ] Recommendation with alternatives, residual risks and confidence in ARCHITECTURE.md and DECISIONS.md
  - [ ] All decision-relevant knock-out criteria sufficiently clarified before the final decision (D-007)
  - [ ] Independent review dispositioned; Product Owner approval (Gate G4)

## Backlog
_None._

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

### T-004 Evaluation frame (subtask of T-003)
- Status: done (2026-11-25, Gate G1 approved with adjustments) | Owner: Claude Code (author), user (approver)
- Tier: Claude, Standard / medium | Depends: plan approval
- Questions: Which criteria and weights decide the stack? What is "reasonable effort" for macOS/Linux? Which dependency licenses and costs are acceptable? Which hardware and OSes are available for testing?
- Acceptance criteria:
  - [x] Criteria, knock-out conditions and measurable "reasonable effort" criteria approved by the Product Owner (Gate G1)
  - [x] Test hardware inventory recorded
- Result: [report](../docs/research/T-004-evaluation-frame.md) (G1 changes in its section 7); D-007, D-008.
- Verification (2026-11-25): hardware and display values by read-only Windows queries plus Product Owner statements; approved weights sum to 100. Weights and thresholds are judgement calls. No web research, tests or independent review (final review is part of T-013).

### T-005 Global capture feasibility (Windows, macOS, X11, Wayland; subtask of T-003)
- Status: done (2026-11-25, accepted by the Product Owner as a research task) | Owner: Claude Code (author), Codex (independent reviewer)
- Tier: Claude, Standard / high | Depends: T-004
- Questions: Which APIs allow passive background capture? Required permissions/privileges? What is invisible (protected contexts)? Press/release/repeat, device identity, rotary events? How is state recovered after lost events? Security implications?
- Acceptance criteria:
  - [x] Capability matrix per OS with evidence levels and sources
  - [x] Per OS: feasible / conditional / not feasible / unknown, with reason
  - [x] List of claims that need a prototype
- Result: [report](../docs/research/T-005-global-capture.md). Windows, macOS and native X11: conditional. Wayland: unknown (no general E5-compliant passive background path demonstrated for the live viewer). Test candidates: D-009.
- Verification (2026-11-25): desk research; [independent review by Codex](../docs/research/T-005-global-capture.md#11-independent-technical-review-codex-2026-11-25) with all findings adopted into sections 1-10 ([dispositions](../docs/research/T-005-global-capture.md#12-disposition-of-review-findings-claude-code-2026-11-25)). Citation IDs and relative links checked. No installs, prototypes or `VER` results: all capabilities remain practically unverified. Gaps: report section 9.
