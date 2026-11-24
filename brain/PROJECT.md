# Project

> Goals, requirements, constraints, success criteria. Change only with user (Product Owner) approval.
> Status: APPROVED baseline (T-002 closed 2026-11-24 by the Product Owner). Review findings and their handling: [TASKS.md](TASKS.md#independent-requirements-review-2026-11-24-codex).
> Markers: `[C]` confirmed by the Product Owner | `[P]` agent proposal, not confirmed | `[O]` open decision (owner and phase in section 8).
> A marker on a bullet overrides the section marker; unmarked bullets inherit it. Only `[C]` items are binding.
> Product requirements only. Tech stack, architecture and storage technology are decided in T-003.

## 1. Overview
- Working name: Xeneon Edge Live Keyboard Display (provisional); final name [O].
- Purpose: A high-quality app combining systematic touch-typing training, speed tests, adaptive learning and an interactive live visualization of physical keyboards. [C]
- Personal goal: Learn ten-finger typing and get used to ergonomic split keyboards (Kinesis Advantage360). [C]
- Motivation: Existing typing trainers and keyboard visualizers do not cover all needs; combine their best aspects in one polished app. [C]
- Audience: First the author; public GitHub repository so others can use it. License [O]. [C]
- Principle: Quality, reliability, maintainability over speed. No time pressure. [C]

## 2. Target Hardware, Display, Usage [C]
- Primary display: Corsair Xeneon Edge, 14.5", 2560x720, 32:9, 60 Hz, touch. UI designed for this format, using the whole area per mode.
- Other monitors/sizes: desirable; priority [O].
- Input: touch, mouse and keyboard.
- Setup A (primary, M1/M2): whole app, incl. training and optional keyboard view, fullscreen on the Xeneon Edge.
- Setup B (later): training on the main monitor; Xeneon Edge shows keyboard view and statistics; configuration on the main monitor also desired.

## 3. Functional Requirements
### F1 Live Keyboard Viewer [C]
- Shows the selected keyboard; only currently pressed keys are visualized (pressed, held, released).
- Standalone live mode without training; works system-wide while other apps are in the foreground; does not interfere with normal typing.
- App is started manually; system-wide capture is a separate opt-in (see NF-Privacy).
- Keyboard view toggleable during training; in-app exercises work without system-wide capture.
- Static, manually configured layouts first. Auto hardware detection, dynamic layers: later.

### F2 Speed Test [C]
- Random words, timed tests, WPM, accuracy, error evaluation, results/progress over time, word lists in German and English.
- Durations, error handling, options [O].

### F3 Endless Training [C]
- No time limit, continuously new content, statistics without a limit, user ends the session.

### F4 Text and Quote Training [C]
- Full sentences/texts; varying length and difficulty; multilingual; interesting content (texts, stories, quotes; film/series/anime themes only if rights allow).
- Only legally usable content with recorded source, license and attribution; no unclear copyrighted collections. Own texts: possible extension.

### F5 Touch Typing Academy (core feature) [C]
- Structured lessons of rising difficulty (single keys, combinations, words, sentences, special characters); fixed lessons plus adaptive training; targeted weakness training; complete beginner path.
- Respects keyboard geometries and layouts; supports transition to split keyboards; saved, traceable progress.
- Lesson explanations and instructions in German and English (see NF-i18n).
- The app never claims to detect which finger the user uses (no sensor).
- Visual aids/finger assignments for different geometries, methodology, scoring, adaptive algorithms: research [O].

### F6 Keyboard Profiles and Layout System [C]
- Vendor-independent. Prebuilt: (1) TKL with rotary knob (Keychron/Lemokey-inspired), ISO-DE and ANSI-US; (2) ZSA Moonlander; (3) ZSA Voyager; (4) Kinesis Advantage360.
- Faithful physical geometry (split boards, thumb clusters, varying key sizes, rotary knobs).
- Prebuilt profiles, manual legend remapping, saved custom profiles. App-only: never changes keyboard firmware.
- Four independently configurable concepts: physical geometry, key layout (legends), training language, finger assignment. Example: English text on a German QWERTZ board.
- Layouts from the start: QWERTZ-DE and QWERTY-US. Alternative layouts (Colemak, Dvorak, Neo): no prebuilt lesson paths now, but the layout system must allow adding them later.
- Later: auto detection, layout import, dynamic layer display, vendor integrations. Must not delay the core.
- Physical key position vs. produced character vs. OS key event, and their limits (duplicate functions, special keys, knobs): research [O].

### F7 Custom Keyboard Editor [C, later]
- Add/remove/arrange keys, size/position, legends, split/ergonomic geometries, save profiles. Reference: keyboard-layout-editor.com (KLE). Scope [O]. Evaluating import/export of an established format [P].

### F8 Statistics and Progress [C]
- Long-term local storage. Filters: keyboard profile, key layout, training language, mode, time range (all M2).
- Minimum metrics: speed, accuracy, development. Personal records, training time, error analysis, heatmaps [P].
- Calculations are consistent, documented, testable.
- Statistics are persisted as aggregates (see NF-Privacy).

## 4. Non-Functional Requirements
### NF-UX / Design [C]
- Minimalist (Monkeytype-like), modern, technical, premium-dashboard look; clear typography; low distraction; subtle animation with instant feedback; no cluttered gaming/RGB look; own visual identity.
- Light and dark mode; adjustable font size (font family optional); color themes wanted but not for v1.
- Good readability and touch usability; touch-target size defined in M2 planning.

### NF-Privacy and Data Lifecycle [C]
Two strictly separated data paths:
- **Live viewer (system-wide):**
  - Opt-in: off after every app start, explicitly enabled; state always clearly visible; visible pause button and configurable deactivation hotkey.
  - Input is processed only transiently for visualization: no recording, no persistent storage, no transmission, no text history.
- **Training capture (in-app):**
  - Training measurement data is captured only during an active, focused typing exercise, from the app's own focused window (no system-wide capture needed).
  - Loss of focus pauses the exercise automatically. While paused (focus loss or manual), no training data is captured; input in other applications never enters the evaluation.
- **Separation:** An enabled live viewer may keep visualizing while training is paused or active; it neither records input nor feeds training data, and training does not use live-viewer input. The two paths share no input data.
- **Retention:** Persist aggregated statistics, not full keystroke histories. Individual events and key timings are not retained after an exercise ends or is aborted. If T-003 finds that more detailed data is required for adaptive training, it must propose a data-minimal solution with justified scope and obtain explicit Product Owner approval first.
- No private input is sent to external services; no needless data transfer; respect OS protections/permissions.
- Automatic password-field detection: maybe later after feasibility check; never presented as a reliable protection guarantee. The UI communicates the limits of system-wide capture to the user.
- Personal data and local settings never end up in the public repo.

### NF-Performance [C]
- Binding: visualization is reliable, fluid and perceived as immediate; simultaneous key presses and pressed/held/released states are correct; no stuck highlights after lost events; low resource usage and no noticeable impact on other applications.
- Preliminary quality targets (not fixed acceptance limits): visual reaction about 30 ms after receipt of the key event; 60 FPS during active visualization.
- Measurement method, reference hardware and final numeric limits: T-003 researches and proposes; Product Owner approves.

### NF-i18n [C]
- UI languages: German and English (menus, settings, learning instructions, lesson explanations); UI language selectable independently of training language.
- i18n foundation (no hardcoded strings, more translations addable without major rework) exists from M1; complete DE/EN UI in M2.
- First launch: the UI language is chosen automatically from the OS language if supported (German, English); otherwise English (fallback). Manually changeable at any time.

### NF-Platforms [C]
- Windows: mandatory for M1 and M2. macOS and Linux: desired, not acceptance criteria for M1/M2.
- Cross-platform implementation is preferred if T-003 shows it is feasible with reasonable effort ([O]: T-003 proposes the criteria, Product Owner approves). No Windows-only stack chosen prematurely; no platform support promised before feasibility is verified.

### NF-Data [C]
- Local standalone app; no mandatory account or server. Stored locally: profiles, settings, results, statistics, learning progress. Import/export desirable.
- Optional sync/self-hosting is a later compatibility consideration: the architecture must not block it, but it does not drive M1/M2 design.
- The Obsidian `brain/` is not the app's data store.

### NF-Public Project [C]
- Public GitHub repo; clear docs, understandable installation, legally clean libraries/fonts/graphics/texts, suitable license (Product Owner decides), no personal data/credentials, ideally no mandatory paid services.

## 5. Scope and Milestones [C]
M1 is the first technical development state. M2 is the complete MVP and builds **cumulatively** on M1: all M1 functions and quality requirements remain part of M2 and are not re-listed. Build order within a milestone may be proposed by agents (T-003/planning).

| Item | Milestone |
|---|---|
| Basic modern minimalist UI, 2560x720 | M1 |
| System-wide live visualization (opt-in) | M1 |
| Profiles TKL (ISO-DE, ANSI-US) + Advantage360 | M1 |
| Manual legend mapping, persistent profiles | M1 |
| Speed Test DE/EN (minimum result, see M1 acceptance) | M1 |
| Local storage; i18n foundation | M1 |
| Moonlander + Voyager | M2 |
| Academy (fixed lessons + first adaptive exercises) | M2 |
| Endless Training; Text/Quote Training (legal content) | M2 |
| Long-term statistics with filters (incl. layout) | M2 |
| Light/Dark mode, font size, complete DE/EN UI | M2 |
| Install and usage documentation | M2 |
| Setup B, Custom Keyboard Editor, auto detection/dynamic layers/vendor integrations, sync, color themes, alternative-layout lesson paths | Later |

## 6. Success Criteria
> Verifiable = checkable by test, measurement or inspection. Subjective = judged by the Product Owner.
> All scenarios are `[C]`: PS-1..PS-9 by user decision, M1/M2 scenarios confirmed at final approval (2026-11-24).

### Privacy scenarios (apply to M1 and M2) [C]
- PS-1 Start: after every app start, system-wide capture is off and no system-wide events are processed.
- PS-2 Activation: explicit user action enables it; state is visible; pressed keys show while another app has focus.
- PS-3 Deactivation/pause: button and hotkey each stop visualization and event processing; state is visible.
- PS-4 Active exercise: only while an exercise is active and focused are training measurements captured.
- PS-5 Focus loss: the exercise pauses automatically; input in other apps changes no training result; nothing is captured while paused.
- PS-6 Exercise end/abort: only aggregates persist; storage inspection finds no individual events, key timings or entered text.
- PS-7 Parallel live view: with live viewer on, other-app keys are visualized while training stays paused and its results are unchanged; live-viewer input is never stored or transmitted.
- PS-8 Restart: capture is off after restart; no raw exercise data survives; aggregates persist.
- PS-9 No transmission: no system-wide or exercise input data leaves the device (inspection).

### M1 acceptance
Verifiable [C]:
- Windows: app starts fullscreen on a 2560x720 secondary display with a cleanly scaled UI, no clipped or overlapping elements, while another app has focus.
- Live view for TKL (ISO-DE, ANSI-US) and Advantage360 highlights pressed keys system-wide; normal typing in the other app is unaffected.
- Simultaneous keys and pressed/held/released states are correct; highlights clear after interruptions (e.g. focus change, lost events). Numeric targets (NF-Performance) are reported, not pass/fail.
- Manual legend mapping; custom profiles persist across restarts.
- Speed Test in German and English: random words, timed run, WPM, accuracy, error evaluation; result persists after restart.
- WPM and accuracy calculation documented and covered by automated tests.
- No hardcoded UI strings (i18n foundation).
Subjective: modern, minimalist look already apparent.

### M2 acceptance (all of M1, plus)
Verifiable [C]:
- Beginner path with fixed lessons and adaptive exercises; minimum path coverage defined in M2 planning; progress saved; after recorded weaknesses, a targeted exercise emphasizes the weak keys.
- Endless Training runs without time limit, keeps supplying new content and ends only when the user ends it.
- Text/Quote Training: every content item has recorded source, license and attribution.
- All four keyboard models; QWERTZ-DE and QWERTY-US; geometry, layout, language and finger assignment independently configurable.
- Keyboard view toggleable during exercises; exercises work without system-wide capture.
- Statistics filterable by keyboard profile, layout, language, mode, time range; trends over time shown.
- Light/Dark mode; adjustable font size; complete DE/EN UI selectable independently of training language; on first launch the UI language follows a supported OS language, otherwise English, and is manually changeable.
- All non-typing functions (navigation, settings, profiles, start/stop) operable by touch and mouse; exercise typing uses the physical keyboard; touch-target size per planning decision.
- Install and usage documentation present; installation from the documentation succeeds on a clean Windows setup.
- App makes no claim to detect the used finger.
Subjective: high-quality modern minimalist premium UI/UX (essential, not optional polish); comfortable touch/mouse operation; clear documentation; suitable visual learning aids and finger assignments.

## 7. Risks [P]
- R1: Large scope; mitigated by M1/M2 split.
- R2: System-wide key capture vs. privacy goal; OS limits (macOS permissions, Wayland).
- R3: Windows-mandatory vs. cross-platform preference; avoid premature stack lock-in.
- R4: Quote/film/anime content vs. public-repo licensing; prefer public-domain, own or openly licensed content.
- R5: Finger assignment on split/columnar boards differs from classic touch typing; needs researched, clearly labelled guidance.
- R6: Performance targets depend on OS event delivery and display pipeline; measurement method needed.

## 8. Responsibilities and Open Decisions
**Roles:** Product Owner (user): product requirements and final approvals, license. Coding agents: technical research, feasibility, proposals. Major architecture decisions: reasoned recommendation, independent review, Product Owner approval. Technical details inside an approved framework: agents decide.

| Phase | Open item |
|---|---|
| T-003 (research, then approval) | Cross-platform feasibility and "reasonable effort" criteria; tech stack; event model (position vs. character vs. OS event, lost-event handling); window focus/touch/multi-monitor incl. Setup B feasibility; performance measurement method, reference hardware, numeric limits; whether aggregates suffice for adaptive training (see NF-Privacy); storage technology, schema versioning, export/import format; keyboard-profile format incl. KLE evaluation; i18n mechanism; font/asset licensing; license constraints from dependencies (license itself: Product Owner) |
| M1 planning | Speed test durations/options/error handling; project name |
| M2 planning | Adaptive algorithm design; lesson content structure and minimum path coverage; finger-assignment aids; touch-target size |
| Later | Priority of non-Xeneon displays; Setup B; editor scope; password-field detection |
