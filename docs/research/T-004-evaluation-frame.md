# T-004 Evaluation Frame

- Status: APPROVED at Gate G1 with adjustments (Product Owner, 2026-11-25, D-007) | Author: Claude Code
- Scope: how technology options will be judged. No stack is assessed or recommended here.
- Evidence levels: `VER` verified here | `DOC` official documentation | `SEC` secondary source | `HYP` assumption.
- Sources: [PROJECT.md](../../brain/PROJECT.md), Product Owner instructions (2026-11-25), local read-only queries (section 6). No web research was done; weights and thresholds are judgement calls, not research findings.
- Changes to weights or thresholds need Product Owner approval and show the ranking before and after.

## 1. Method
1. **Option** = a complete combination (language, UI framework, capture integration, persistence, packaging), not a single library.
2. **Knock-out check** (section 2). One confirmed failure removes an option. An unknown or unverified capability is **not** a failure: mark it open, investigate it specifically, and resolve it before G4.
3. **Weighted scoring** (section 3), 0-5 per criterion. Points = sum(weight x score / 5), maximum 100.
4. **Floors:** a finalist needs at least 3 in C1, C2, C3 and C4. A high total cannot compensate. An unscored criterion does not fail a floor but stays open and must be scored before G4.
5. **Evidence caps:** `HYP`/`SEC` at most 3, `DOC` at most 4, 5 only with `VER`. No evidence = "unscored", listed by name; it is not counted as 0.
6. **No false precision:**
   - Totals are rounded to whole points and always shown with **evidence coverage** (share of weight backed by `DOC` or `VER`).
   - With any unscored criterion, show a range instead of a total: lower bound (unscored = 0) to upper bound (unscored = 5).
   - Below 60 % evidence coverage, label the option "insufficient evidence" and do not rank it.
   - Overlapping ranges are not ranked.
   - The 60 % threshold only permits a **preliminary** ranking. A final architecture decision additionally requires all decision-relevant knock-out criteria to be sufficiently clarified.
7. **Sensitivity** (once single totals exist): recompute with each weight changed by -50 % and +50 % (others rescaled to 100), and with C5 (cross-platform support) set to 0 and to 20. A lead under 5 points, or a lead that flips, is a tie.
8. **Tie-break:** higher C2, then C4, then the option with fewer languages and moving parts.
9. **Stages:** T-008 provisional scores (`DOC`/`SEC`) -> T-009 updates with `VER` -> T-013 final.

Score anchors: 0 impossible | 1 only with severe workarounds | 2 notable gaps or risk | 3 meets the requirement with known, manageable limits | 4 meets it well with mature, documented support | 5 meets it well, proven in our own tests.

Item 6 details (range display, 60 % threshold) implement the G1 instruction to avoid pseudo-precise totals. The author proposed them; the Product Owner accepted them on 2026-11-25 with the final-decision condition above.

## 2. Knock-out criteria
| ID | Condition to pass | Proven by |
|---|---|---|
| KO-1 | Windows 11: system-wide press/release capture in the background; focused-window input giving physical position and character; fullscreen on the 2560x720 secondary display. Hardware or firmware limits that exist with every stack (e.g. physical keys that send no unique OS event) do **not** exclude an option | T-005..T-007, T-009 |
| KO-2 | Capture is passive: normal typing is not blocked, delayed noticeably or altered | T-007 |
| KO-3 | Normal use needs no administrator rights, custom driver or weakened OS security (limits for protected windows are documented instead) | T-005, T-007 |
| KO-4 | Privacy: no capture registered until enabled; PS-1..PS-9 achievable and testable; no mandatory network traffic or telemetry that cannot be fully disabled | T-008, T-009 |
| KO-5 | No mandatory cost: license, API, hosting, account, or code signing for M1/M2 | T-008, T-012 |
| KO-6 | Every shipped dependency is redistributable from a public repository under section 5 | T-008, T-012 |
| KO-7 | Core components are maintained and viable long term, judged on actual maintenance, security fixes, compatibility with current OS versions, maturity and long-term usability. A stable release within the last 12 months is an indicator, not an exclusion criterion | T-008 |

- Stack-independent hardware/firmware limits (KO-1) are documented with their possible effect on product requirements (T-006) and raised to the Product Owner.
- macOS and Linux support is deliberately not a knock-out.

## 3. Weighted criteria
| ID | Criterion | Weight | What is assessed | Risks |
|---|---|---|---|---|
| C1 | Windows input reliability | 18 | Complete and correct events, simultaneous keys, recovery from lost events, device and knob handling | TR-1, 3, 7 |
| C2 | Privacy and security | 17 | Isolation of capture code, data path separation, attack surface, dependency count and supply chain, leak paths (logs, crash dumps, IPC) | TR-4 |
| C3 | UI/UX capabilities | 17 | Visual quality and animation, 2560x720, touch, no focus stealing, multi-monitor and mixed scaling, theming, font size, i18n, accessibility | TR-5 |
| C4 | Maintainability | 13 | Simplicity, number of languages, maturity, governance and longevity, upgrade churn, documentation | - |
| C5 | Cross-platform support | 12 | Section 4 result; two sub-scores: C5a macOS (8), C5b Linux (4) | TR-2 |
| C6 | Testability | 8 | Fake input source, automated unit and UI tests, tests in CI | TR-11 |
| C7 | Performance and resource efficiency | 10 | Latency, frame rate, CPU, memory, behaviour while unfocused, effect on other applications | TR-6 |
| C8 | AI-agent development efficiency | 3 | Documentation and examples, readable errors, command-line build and test, build time, context needed | TR-11 |
| C9 | Distribution and installation | 2 | Windows install, size, runtime prerequisites, update path, antivirus and signing friction | TR-10 |
| | **Total** | **100** | | |

C1-C4 (65) carry the stated priorities: quality, privacy, reliability, maintainability, UI/UX. "Reliable and fluid" is additionally enforced by the C1 and C3 floors and KO-2.

## 4. "Reasonable effort" for macOS and Linux
Long-term maintainability and product quality outrank short development time. Assessed per platform.

| ID | Criterion | Threshold | Kind | Assessed |
|---|---|---|---|---|
| E1 | One shared UI and domain code base | No per-OS copy of UI, training logic, statistics or storage | Required | T-008, T-009 |
| E2 | Platform code is contained | Only behind these adapters: input capture, permissions, window/display, packaging | Required | T-009 |
| E3 | Platform-specific code share | Guide value 15-20 % of non-test source lines | Guide | T-013 estimate |
| E4 | Extra development work | Guide value 25-30 % per additional platform on top of the Windows-only plan, counted as estimated work packages of similar size | Guide | T-013 estimate |
| E5 | Safe capture path | Public, documented APIs; at most normal user permission prompts; no root at runtime, no broad device-group access, no disabled OS protection | Required | T-005 |
| E6 | Cost | No mandatory cost for the supported level | Required | T-012 |
| E7 | No significant Windows penalty | Windows passes all knock-outs and floors; a drop of more than 1 point in any of C1-C4 versus the best Windows-only reference counts as significant | Required | T-009, T-013 |
| E8 | Verifiable | A real, suitable test system is required for officially promised support; not for experimental support or research | Required for `Supported` only | now: macOS yes, Linux no |

- A platform is reasonable if all required criteria for the claimed level pass and the guide values are met.
- Guide values are not hard limits: a justified overrun is submitted to the Product Owner for approval.
- E3 and E4 are low-precision estimates, re-checked when real code exists. E5 may be rated separately for X11 and Wayland.

### Support levels
| Level | Meaning |
|---|---|
| Supported | Officially promised: acceptance checks passed on a real, suitable system; maintained |
| Experimental | Builds and at least partly tried; no guarantee; limits and test environment documented; real hardware not mandatory |
| Unverified | Possibly portable according to sources; no promise |
| Not supported | Known blocker, or not reasonable under E1-E8 |

Reachable today: Windows and macOS `Supported`, Linux at most `Experimental`. Current Linux label: `Unverified` (research first). M1/M2 acceptance stays Windows-only.

Prototypes: macOS variants of T-007 and T-009 are part of the plan if prior research justifies them; Windows first, then macOS; installation and execution each need separate approval.

## 5. Cost and license framework
### Cost (Product Owner decision, 2026-11-25)
- No additional recurring cost for M1/M2: no mandatory paid API, hosting, central accounts, cloud infrastructure or code signing; no AI spend beyond existing subscriptions.
- Free hosted services (e.g. CI) only if optional and replaceable.
- A paid option with a significant advantage or technical necessity is submitted separately, including one-off costs.

### Dependency and content licenses (accepted at G1; engineering policy, not legal advice)
| Class | Examples | Rule |
|---|---|---|
| Allowed | MIT, BSD-2/3, Apache-2.0, ISC, Zlib, BSL-1.0, Unicode; fonts OFL-1.1; content CC0, public domain, CC BY | Use; record license and attribution |
| Case by case | MPL-2.0, LGPL, EPL, content CC BY-SA, database licenses, dual or revenue-limited licenses, free proprietary runtimes shipped by an OS vendor | Document obligations; Product Owner approves |
| GPL / AGPL | Libraries, frameworks or code under GPL/AGPL | May be researched; integration into the shipped product only with explicit Product Owner approval after the license obligations were examined |
| Excluded by default | CC NC or ND, no or unclear license, non-redistributable assets | Only if the Product Owner changes this policy |

The project license stays open. The obligations of each actual dependency are verified in T-008/T-012.

## 6. Test hardware and constraints
`VER`: read-only Windows queries on the development PC, 2026-11-25. `PO`: stated by the Product Owner, 2026-11-25. Serial numbers and device instance IDs are intentionally omitted. Open hardware questions are tracked in [STATUS.md](../../brain/STATUS.md).

| Item | Value | Evidence |
|---|---|---|
| OS | Windows 11 Pro 26H2, build 10.0.26300.9974, 64-bit (primary development platform) | VER |
| CPU / RAM | AMD Ryzen 7 9800X3D (8C/16T), about 31 GB | VER |
| GPU | AMD Radeon RX 9070 XT (plus integrated Radeon) | VER |
| Primary display | LG UltraGear+, 5120x2160, 165 Hz, 125 % scaling | VER |
| Secondary display | Corsair Xeneon Edge, 2560x720, 60 Hz, 100 % scaling, desktop origin (-2560, 1440) | VER |
| Touch | Does not work (PO). Windows reports no digitizer (`SM_DIGITIZER` = 0, 0 touch points). Cause unknown | PO, VER |
| Lemokey P1 Pro | 75 %, ISO-DE, rotary knob; devices "Lemokey P1 Pro" and "Lemokey Link" were connected | PO, VER (names) |
| Kinesis Advantage360 SmartSet (not Pro/ZMK) | Owned; not detected during the query | PO |
| ZSA Voyager | Owned; not detected during the query | PO |
| ZSA Moonlander | **Not available** for hardware tests; profile stays "not hardware-verified" | PO |
| macOS | Real Mac with Apple M4; exact model and macOS version to be added before practical tests | PO |
| Linux | No confirmed test system | PO |

Constraints:
1. Touch is a **hardware test blocker for later touch tests** (T-009 onwards), not for research.
2. The Windows PC is high-end: performance results are optimistic. Judge resource use in absolute values; no low-end Windows reference exists.
3. Mixed refresh rates and scaling and negative coordinates are useful real test conditions; other scaling values need deliberate test runs.
4. The Windows build may be a preview channel (`HYP`). Behaviour on older or stable Windows 11 builds would then be unverified.
5. Virtual machines do not count as verification for capture, touch or performance.

## 7. History: Gate G1
Proposal 2026-11-25; approved the same day with these changes:

| Topic | Proposed | Approved |
|---|---|---|
| KO-1 | No statement on device limits | Stack-independent hardware/firmware limits do not exclude an option; document them |
| KO-7 | Release within 12 months required (small libraries could be justified) | Release age is an indicator; overall maintenance and viability decide |
| Unknowns | Unscored counted 0 | Not a failure; open, investigated; ranges instead of pseudo-precise totals |
| Weights C4 / C5 / C7 / C8 / C9 | 15 / 10 / 8 / 4 / 3 | 13 / 12 / 10 / 3 / 2 |
| C5 split macOS / Linux | 7 / 3 | 8 / 4 |
| E3 / E4 | Hard limits 15 % / 25 % | Guide values 15-20 % / 25-30 %, justified overruns possible |
| E8 | Real system for any claimed level | Required only for officially promised support |
| GPL / AGPL | Excluded by default | Research allowed; integration needs explicit approval |
| macOS prototypes | Proposed | Approved for the plan, conditional on research; separate approvals |

Accepted unchanged: KO-2..KO-6, floors, evidence levels and caps, sensitivity analysis, tie rules, C1-C3 and C6 weights.

Follow-up approval (2026-11-25): the author's concretizations (range display, 60 % threshold, E7 indicator) were accepted; the 60 % threshold applies to preliminary rankings only (section 1, item 6).
