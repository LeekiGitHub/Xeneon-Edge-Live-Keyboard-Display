# T-005 Global Capture Feasibility

- Status: ACCEPTED as a research task by the Product Owner (2026-11-25) | Author: Claude Code | Independent reviewer: Codex
- The corrections of the [independent technical review](#11-independent-technical-review-codex-2026-11-25) are consolidated into sections 1-10. **Sections 1-10 are current**; section 11 is the unchanged review history.
- Scope: passive, system-wide keyboard visualization on Windows 11, macOS (Apple Silicon), Linux/X11 and Linux/Wayland.
- Out of scope: tech stack decision, keyboard firmware behaviour and the event model (T-006), prototypes (T-007).
- **Nothing in this report was tested.** There is no `VER` result. No software was installed and no code was written.
- All web sources accessed 2026-11-25. Source list: section 10.

## 1. Evidence rules for this report
| Tag | Meaning |
|---|---|
| `DOC` | Official documentation, specification or official source file that was retrieved |
| `DOC†` | Official source, but only available as a machine-generated summary; exact wording not confirmed |
| `SEC` | Secondary source: community, third-party project, press, or an individual forum answer (including Apple staff answers) |
| `SEC‡` | Secondary source seen only as a search-result excerpt; the page itself was not opened |
| `HYP` | Assumption or inference by the author; not backed by a retrieved source |

Retrieval limits: the author read pages through a fetch tool run by a helper agent. Quotes are as that tool returned them; raw page bytes were not independently checked. The search tool is US-only and returns machine summaries. Sources marked `(R)` in section 10 were retrieved by the independent review and not re-fetched by the author; their tags follow the review. A statement that something is "not documented" means: not found on the pages listed here.

`DOC` describes what a source says. It never means the behaviour was observed: **every capability in this report is practically unverified.**

## 2. Verdicts
| Platform | Verdict | Reason | Confidence |
|---|---|---|---|
| Windows 11 | **Conditional** | Documented user-mode background capture exists (Raw Input, low-level hook) [W1, W2, W7]. Protected desktops limit it; delivery while an elevated window on the same desktop has focus is unknown; nothing tested | Medium-high that the basic path works |
| macOS (Apple Silicon) | **Conditional** | Public passive API (listen-only event tap) [M2] under the user-granted Input Monitoring permission [M24, M6]. Secure input restricts delivery [M14]; behaviour on the target build is open | Medium |
| Linux / X11 (native session) | **Conditional** | Passive XInput2 raw-event subscription is documented [X8] for a client authorized on that X server [X12] with XI2 available. No test system; runtime particulars open; GNOME and Plasma retired their X11 sessions [X13, X6] | Low-medium |
| Linux / Wayland | **Unknown** | **No general E5-compliant passive background path demonstrated for the live viewer.** The reviewed routes do not provide one [L1-L6, L10]; compositor-specific extensions and policies were not surveyed exhaustively | Medium that the reviewed routes are unsuitable |

The Wayland finding is an E5 gap for **the system-wide live viewer only**. Input to the app's own focused window (training) is a documented use of the core protocol [L1].

**Windows test candidates for T-007:** Raw Input is the preferred candidate to investigate; the low-level hook is a possible comparison candidate (Product Owner, 2026-11-25, D-009). This is neither a stack nor an architecture decision.

**macOS prototype (T-007 macOS variant): justified.** A documented listen-only candidate exists that needs only a normal user permission, a real Mac is available, and the open points can only be settled on hardware. Windows first; record the actual Mac model and OS build; installation and execution need separate approvals.

**Linux:** support level stays `Unverified`; any support-label decision stays with the Product Owner. No prototype is recommended without a test system.

## 3. Capability matrix
`yes` / `no` / `partial` / `?` (unknown). Tags and sources per cell. No cell is practically verified. Wayland cells describe reviewed routes only.

| # | Capability | Windows 11 | macOS | Linux X11 | Linux Wayland |
|---|---|---|---|---|---|
| C01 | Candidate interface | Raw Input (preferred test candidate); `WH_KEYBOARD_LL` (comparison) `DOC` [W1, W7] | Listen-only `CGEventTap`; `IOHIDManager` `DOC` [M2, M9] | XInput2 raw events `DOC` [X8] | No general compliant path demonstrated `HYP` from [L1-L6] |
| C02 | Press/release while another app has focus | documented `DOC` [W2, W7] | documented, needs permission `DOC` [M24] | raw press/release on the connected X server `DOC` [X8] | not via core protocol `DOC†` [L1] |
| C03 | User permission | none documented `HYP` | Input Monitoring (listen-only tap, HID) `DOC` [M24, M6, M8]; `SEC` [M4] | client authorization on the X server `DOC` [X12]; no user prompt known `HYP` | n/a |
| C04 | Admin, root or driver needed | no requirement documented `HYP` | no root for session tap; HID-level tap is root-only `DOC` [M1] | no requirement found `HYP` | device nodes: root or `input` group `DOC` [L5] |
| C05 | Passive (cannot block typing) | Raw Input: asynchronous `DOC` [W7]. Hook: can suppress or delay input `DOC` [W7] | passive listener `DOC` [M2] | passive subscription `DOC` [X8] | n/a |
| C06 | Capture can stop or be suspended | Hook silently removed after timeout, app cannot know `DOC` [W7]. Raw Input: ? | timeout disable event `DOC` [M10]; secure input `DOC` (archived) [M14]. Enabled state during secure input: open (one report [M15] `SEC`) | grab behaviour changed between XI 2.0 and 2.1 `DOC` (draft) [X9]; actual grab/freeze behaviour ? | n/a |
| C07 | State recovery | `GetAsyncKeyState`: per virtual key, zero when access is blocked; **cannot prove a healthy stream or per-device state** `DOC` [W9] | `CGEventSource.keyState` `DOC` [M11]; limits ? | `XQueryKeymap`: logical state, may lag while event processing is frozen `DOC` [X11] | evdev only (`EVIOCG*`) `DOC` [L6] |
| C08 | Auto-repeat identifiable | legacy messages flag it `DOC` [W15]; Raw Input/hook: ? | autorepeat field `DOC` [M12] | raw events: ? | `repeated` state exists for focused windows `DOC†` [L1]; evdev value 2 `DOC` [L6] |
| C09 | Physical key code | scan code + E0/E1; virtual key `DOC` [W3, W8] | virtual keycode, keyboard type `DOC` [M12] | keycode in raw events `DOC` (draft) [X9] | evdev key code `DOC` [L6] |
| C10 | Which keyboard sent it | Raw Input: `hDevice` `DOC` [W1, W4]. Hook: no device field `DOC` [W8] | HID manager: per-device enumeration `DOC` [M9]; tap: only keyboard type seen `HYP` | `sourceid` from XI 2.1 `DOC` (draft) [X9] | evdev: per node `DOC` [L6] |
| C11 | Hot-plug | arrival/removal messages `DOC` [W6] | matching/removal callbacks `DOC` [M9] | device hierarchy events `DOC` (draft) [X9] | n/a |
| C12 | Knob / media keys | consumer collection opened shared `DOC` [W16]; no knob event guarantee | system-defined events or HID values `HYP` [M13] | ? | n/a |
| C13 | Fn / firmware-only keys | Only what reaches the OS (all platforms) `HYP`; macOS has an Fn flag `DOC` [M13]; see T-006 | partial `HYP` | partial `HYP` | n/a |
| C14 | Elevated / privileged foreground app (same desktop) | **? for both Raw Input and hook** (indirect hints only [W9, W18, W19]) | ? | ? | n/a |
| C15 | UAC, lock screen, login | protected desktop: treated as unavailable in normal use; hooks are same-desktop `DOC` [W11, W12]; transitions untested | not delivered `HYP` | screen-locker behaviour ? | n/a |
| C16 | Password / secure fields | no verified suppression; keys may be delivered `HYP` | Secure Event Input stops delivery to the described intercept mechanisms incl. taps `DOC` (archived) [M14], Terminal secure entry [M25]; modern tap and HID behaviour open | no guaranteed protection `HYP` | n/a |
| C17 | Injected input distinguishable | hook: injected flags `DOC` [W8]; Raw Input: ? | ? | ? | n/a |

## 4. Windows 11
### Interfaces
| Interface | Documented behaviour | Limits for this project |
|---|---|---|
| Raw Input (`WM_INPUT`): **preferred test candidate** | Opt-in registration; asynchronous background delivery with `RIDEV_INPUTSINK` (window required); identifies the source device; arrival/removal with `RIDEV_DEVNOTIFY`; registration can be removed [W1, W2, W4-W7] `DOC` | One registered window per device class per process; registering from a library may interfere with the host application [W5] `DOC`. A UI framework that registers Raw Input itself may conflict (`HYP`, TR-6). Repeat, injection, rotary events and recovery are open |
| `WH_KEYBOARD_LL`: **comparison candidate** (KO-2) | Global, same desktop; callback on the installing thread with a message loop; extended/injected/up flags; no device identifier [W7, W8, W10] `DOC` | Can suppress or delay input; on timeout (max 1000 ms) Windows 7+ removes the hook silently. Microsoft: "In most cases where the application needs to use low level hooks, it should monitor raw input instead." [W7] `DOC` |
| `GetAsyncKeyState` | Current down state per virtual key [W9] `DOC` | Polling aid only. Can return zero on a non-active desktop, under UIPI, or without desktop hook access [W9]. The "pressed since last call" bit is unreliable. Not a health check of the event stream |
| Direct HID access | Keyboard collections are opened exclusively by Windows; consumer controls shared [W16] `DOC` | Not usable for keyboard keys; a shared consumer collection does not guarantee knob events |

### Constraints and risks
- Protected desktops: there is a single active input desktop; consent prompts use the secure desktop by default and only Windows processes can access it [W12]; the Winlogon desktop is not accessible to applications; hooks only receive messages of their own desktop [W11] `DOC`. Treat capture on a protected desktop as unavailable in normal use and test transitions and recovery. Key releases during a switch may be missed, leaving stuck highlights (`HYP`).
- Elevated foreground window on the **same desktop**: delivery to a standard-user app is **unknown for both APIs**. Hints only: Microsoft describes specific PowerToys functions failing [W18], AutoHotkey describes its own hotkeys being blocked [W19] `SEC`, and `GetAsyncKeyState` documents restrictions of a separate polling API [W9]. None establishes Raw Input or low-level hook delivery.
- `uiAccess` is excluded: Microsoft reserves it for relevant assistive technology and requires trust (code signing) and installation in a protected location [W13] `DOC`. It is not needed for normal use. No claim is made that every signed deployment must carry a fee.
- A third-party report (2026-11-04) says that a framework's own Raw Input registration stopped the same process's low-level hooks from firing while its window had focus [W20] `SEC`. Single project, unconfirmed.
- Session change notifications exist (`WTSRegisterSessionNotification`, `WM_WTSSESSION_CHANGE`) [W14, W22] `DOC`; their use as a "clear highlights" trigger is `HYP` until verified.
- No 2026 Windows change restricting hooks or Raw Input was found; a narrow search is not proof of absence. A January 2026 credential-dialog change concerns injected input [W21] `SEC‡`.

### Privacy
- No consent prompt was found in the documentation: opt-in (PS-1..PS-3) must be enforced by the app itself, by registering only while enabled and removing the registration on pause (`HYP` design; removal flag exists [W5]).
- Microsoft's unwanted-software criteria require clear indication of activity, notice and consent for software that stores or transmits user activity, and no option to hide it [W17] `DOC`. PS-2 points the same way. Antivirus reaction to the app is unknown (TR-4).

## 5. macOS (Apple Silicon)
None of the API pages states anything specific to Apple Silicon; applicability to the M4 Mac is `HYP` until tested.

### Interfaces
| Interface | Findings | Assessment |
|---|---|---|
| `CGEventTap`, listen-only | "passive listener" [M2] `DOC`. Apple's Catalina-era guidance specifies Input Monitoring for a listen-only tap, distinct from the Accessibility authorization of an active tap [M24] `DOC`. Apple DTS (2022) corroborates, notes sandbox/App Store availability and describes `CGPreflightListenEventAccess` / `CGRequestListenEventAccess` [M3, M4] `SEC` | Primary candidate |
| `IOHIDManager` | Listening needs user-granted access (10.15+) [M8]; device matching, removal and value callbacks [M9] `DOC`. Seize option exists: **must not be used** (not passive) | Candidate for device identity and knob; never for recovering protected input |
| `NSEvent` global monitor | Observe only; key events need Accessibility trust; excludes own app's events [M7] `DOC` | Not preferred: broader permission |
| Active (default) tap | Triggers Accessibility permission [M5] `SEC` | Not needed; excluded for least privilege and KO-2 |

### Permission, secure input, reliability
- Users grant and revoke Input Monitoring per app under Privacy & Security [M6, M26] `DOC`. A persistent OS grant does **not** authorize starting capture on app launch (PS-1).
- **Legacy documentation conflict:** Apple's `tapCreate` page says key events require root or "access for assistive devices" and still refers to OS X 10.4 [M1]. This legacy wording conflicts with Apple's newer listen-only guidance [M24], which is used here as the documented model. Exact behaviour on the target build still needs a test.
- Secure Event Input: Apple's technical note (2007, archived, not updated) says keyboard events stop reaching the intercept mechanisms it describes, including event taps, while any process has it enabled; apps that leave it on affect everyone [M14] `DOC`. Apple also documents Terminal's secure keyboard entry [M25] `DOC`. Open runtime questions: behaviour of a modern listen-only tap, its enabled state (one 2026 report on macOS 26.6.2: reports enabled, receives nothing [M15] `SEC`), and especially non-seizing HID callbacks.
- Taps receive a timeout-disable event [M10] `DOC`. The author also retrieved the user-input disable event and `tapEnable` re-enabling descriptions [M10]; the review could not independently retrieve the `tapEnable` description.
- Modifiers have a separate "flags changed" event type [M12] `DOC`; how Caps Lock, Fn/Globe and media keys behave needs a test.

### macOS 27
- Release notes (beta-cycle text): no capture-related item in the verbatim part read; the middle was only machine-summarized and the end was not read [M19] `DOC†`, incomplete.
- Apple's macOS 27 user guide confirms the user-facing Input Monitoring control [M6, M26] `DOC`. It does not settle the exact tap behaviour on the target build.
- Third parties report that the Accessibility row is labelled "Device Control and Data Access" [M20, M21, M22] `SEC`, and one project merged a change on 2026-11-09 saying its key remapping works without a separate Input Monitoring grant [M21]. This concerns an app holding the broader permission and says nothing reliable about listen-only apps. A forum thread reports a managed-permission bug on 27.x, acknowledged by Apple DTS [M20] `SEC`.
- Consequence: permission behaviour is version-sensitive. Record the Mac's macOS version before testing.

### Signing and distribution (input for T-012)
- Apple DTS: ad-hoc signed code has no stable designated requirement, so permissions may not carry over between builds; use a stable identity [M16] `SEC` (thread about Screen Recording; only indirect evidence for Input Monitoring persistence, `HYP`).
- Apple Developer Program: 99 USD per year [M18]; needed to distribute with Apple credentials [M23] `DOC`. Unnotarized apps can be opened through a manual override [M17] `DOC`.
- Whether a free development identity is sufficient for local testing: `HYP`. M1/M2 acceptance is Windows-only, so no conflict with NF-Cost today.

## 6. Linux
### X11 (native session)
- A client can passively subscribe to raw press/release events on the root window of the connected X server [X8] `DOC`.
- A historical XI2 draft describes the XI 2.0 grab restriction and its change in 2.1, keycodes, `sourceid` from 2.1 and device hierarchy changes [X9] `DOC` (draft); a later specification copy in a personal fork corroborates the version history [X10] `SEC`. The current upstream specification and an issue reporting a synchronous-grab exception [X1] `SEC‡` were not available as primary evidence.
- `XQueryKeymap` is documented and yields logical state that may lag while event processing is frozen [X11] `DOC`.
- Access is not unconditional: X.Org documents restricted client authorization [X12] `DOC`. Broad statements such as "any client can read everything" or "no OS protection exists" are not supported; no guarantee of protection for sensitive input is claimed either. Under Xwayland, X11 clients are not isolated from each other by Xwayland itself and are isolated from Wayland clients [X3] `DOC`.
- **Open:** actual grab/freeze behaviour, raw repeat, multi-device behaviour, screen lockers. The RECORD extension was not researched from a retrieved source.
- Platform trend: Plasma 6.8 (2026-10-14) retired native X11 sessions [X6] and GNOME's release history confirms the same for GNOME [X13] `DOC`; see also [X5] `SEC`. Other desktops with X11 sessions were not researched.
- E5: plausible only for an authorized client in a native X11 session with XI2 available; untested.

### Wayland
Finding: **no general E5-compliant passive background path demonstrated for the live viewer.** This rules out the reviewed routes, not every possible compositor.

| Route | Finding | Usable? |
|---|---|---|
| Core `wl_keyboard` | Key events only while a surface of the client has keyboard focus [L1] `DOC†` | No for background |
| GlobalShortcuts portal | App is told only about its registered shortcuts [L2] `DOC` | No (could serve the pause hotkey, `HYP`) |
| InputCapture portal | Armed for later compositor activation (pointer barriers); input is redirected to the app during capture; no immediate activation [L3] `DOC` | No: not passive monitoring |
| Compositor `a11y.KeyboardMonitor` (GNOME) | Watches all keys without suppressing them; current source restricts it to an authorized service (sender name `org.gnome.Orca.KeyboardMonitor`, plus a debug override) [L4, L10] `DOC` | No: reserved for the screen reader. Claiming that name would bypass an access control. A third party reports this is possible and that KWin offers the interface too [L7] `SEC`. **Rejected** |
| Kernel evdev (`/dev/input/event*`) | Full events incl. repeat and loss recovery [L6]; udev assigns the nodes to group `input` [L5] `DOC` | No: violates E5 (root or broad device group) |
| logind device handover | Reserved for the session controller [L8] `DOC†` | No |
| Xwayland client | Isolated from Wayland clients [X3] `DOC`; remains subject to compositor delivery. KDE has a "Legacy X11 App Support" setting that can forward keys to X11 apps [L9] `SEC‡` | No general access to native Wayland input; depends on weakening isolation; not a support basis |
| Compositor-specific extensions/protocols | Not researched | `HYP` |

## 7. Privacy, security and compatibility with the knock-outs
### Cross-platform implications
- Each candidate path exposes **the keystrokes the OS actually delivers, potentially including input into ordinary password fields**. A granted permission does not enforce visualization-only use; the app's design must (TR-4, KO-4).
- PS-1 to PS-3 require visible opt-in and pause behaviour. A persistent OS grant does not authorize starting capture on launch.
- Proposals for later design (`HYP`, not decided): capture only while enabled; keep only the current set of pressed keys in memory; no event logging, history or character translation in the live path; consider crash dumps, IPC and third-party dependencies as leak paths.
- Sensitive contexts differ: macOS secure input and Windows protected desktops restrict delivery; for ordinary password fields on Windows and X11 no suppression is verified. The app must not promise password protection (matches NF-Privacy).
- Lost releases must be expected wherever delivery is suspended. The live view needs a defined reset/resync rule per platform (NF-Performance, TR-3).

### Status display: three separate states
| State | Meaning | What the app can know |
|---|---|---|
| Enabled | The user opted in and has not paused | Known (app's own state) |
| Registered / permitted | OS registration or permission is in place | Partly: query or error result at that moment |
| Receiving | Events are actually arriving | Only evidenced by observed events. **An idle keyboard cannot reveal an interruption** |

Show known interruptions and clear highlights on **verified** transition signals. Never present "enabled" as proof of reception. `GetAsyncKeyState` cannot establish a healthy stream or per-device state [W9].

### Knock-out and E5 status (documentary only; unknown is not a failure)
| Check | Windows | macOS | X11 | Wayland |
|---|---|---|---|---|
| KO-1 capture part (Windows only) | open, documented basis; T-007 | n/a | n/a | n/a |
| KO-2 passive | open; Raw Input favourable, hook needs care | open; listen-only favourable | open | open (no path) |
| KO-3 no admin/driver/weakened security | open; limits for protected windows get documented | open | open | open; device-node route excluded, no compliant route demonstrated |
| KO-4 no capture until enabled, PS-1..PS-9 | open; app-enforced | open; app-enforced plus OS permission | open; app-enforced plus X client authorization | open (no path) |
| E5 safe capture path (X11/Wayland, macOS) | n/a | plausible (`DOC`), untested | plausible for an authorized native X11 client (`DOC`), untested | **gap:** no compliant path demonstrated |

Knock-outs apply to technology options; KO-1 is specific to Windows. macOS and Linux affect only the optional-platform assessment. No option is assessed or scored here, and no stack preference follows. The approved Windows acceptance scope and KO-5 to KO-7 are unchanged.

### Findings to raise with the Product Owner
No requirement change is proposed now; decide after T-007 (Gate G2).
1. A complete live view is not achievable in all contexts (Windows protected desktops; macOS secure input). Elevated windows on the same desktop are an open question.
2. PS-2/NF-Privacy want an always clear state. Capture can stop without the app being told (Windows hook removal `DOC`), and reception cannot be proven while the keyboard is idle. See the three states above.
3. Linux: no general compliant Wayland path is demonstrated, and major desktops retired X11 sessions. Level stays `Unverified`; any support label is the Product Owner's decision.
4. Distributing a macOS build with Apple credentials carries a recurring fee (T-012).

## 8. Prototype claims and test method
All claims are unverified. Raw Input is the preferred candidate; the low-level hook is the comparison.

### Windows (T-007). P = test priority
**First trial:** standard user, both candidates, compared against foreground typing with capture off, including the privacy lifecycle (WP-01, WP-02, WP-06, WP-13, WP-14).

**Go/no-go basis:** does a candidate meet the capture scope of KO-1, plus KO-2 and KO-3? Delivery in protected contexts and parity between the two APIs are observations, not pass conditions. The focused-input and fullscreen parts of KO-1 need separate evidence; the lifecycle trial informs KO-4, while training-path isolation, persistence and transmission still need product verification.

| ID | P | Claim to verify |
|---|---|---|
| WP-01 | 1 | Raw Input with `RIDEV_INPUTSINK` delivers press/release while another normal app has focus, without admin or prompt |
| WP-02 | 1 | Same via `WH_KEYBOARD_LL`, as comparison |
| WP-03 | 1 | What each path receives while a raised-privilege window has focus; tested separately from UAC, lock and secure desktop |
| WP-04 | 1 | UAC prompt, Win+L, Ctrl+Alt+Del: press on the ordinary desktop, release during the transition; events received, releases lost, stuck state |
| WP-05 | 1 | Highlights clear and fresh events are handled correctly on return (also sleep/resume); `GetAsyncKeyState` per context; usable transition signals |
| WP-06 | 1 | Typing in the foreground app is unaffected under key bursts |
| WP-07 | 1 | Simultaneous keys, hold/repeat, left/right modifiers, AltGr on ISO-DE, extended keys, Pause, Print Screen |
| WP-08 | 1 | Knob rotate/press on the Lemokey P1 Pro: which path and codes |
| WP-09 | 2 | Device identity (cable vs. wireless receiver, second keyboard), hot-plug, injected input |
| WP-10 | 2 | Reaction of installed antivirus to the prototype |
| WP-11 | 2 | Coexistence with a window of the same process (one-registration rule, own focus) |
| WP-12 | 2 | Visibility of dummy keys in a test password field (wording of the UI limitation notice) |
| WP-13 | 1 | Privacy lifecycle (PS-1 to PS-3, capture part of PS-8): no registration at start; explicit activation; pause by button **and** hotkey; immediate clearing and no further processing; off after restart; repeated activation |
| WP-14 | 1 | The deactivation hotkey does not interfere with normal typing |
| WP-15 | 1 | Status: enabled, registration state and observed delivery are distinguishable; known interruptions are reported; an idle keyboard is not reported as interrupted or as proven receiving |

### macOS (T-007 variant)
Only after Windows, with the actual Mac model and OS build recorded and separate approvals. Approval and startup behaviour is version-sensitive.

| ID | P | Claim to verify |
|---|---|---|
| MP-01 | 1 | Listen-only session tap works with Input Monitoring only; expected states for permission denied, granted and revoked; prompt flow; restart needed? |
| MP-02 | 1 | Ordinary background press/release **before and after** Secure Event Input; modifiers, Caps Lock, Fn/Globe, function keys |
| MP-03 | 1 | Protection check with dummy input (test password field, Terminal secure entry): no delivery expected; detectability; recovery without stuck keys. Unexpected delivery is a protection finding (rule 9) |
| MP-04 | 1 | Tap disable/re-enable; permission revoked while running (no hang, no input disruption) |
| MP-05 | 2 | State resync after lock, sleep, user switch |
| MP-06 | 2 | HID manager outside protected contexts: device identity, knob, required permission |
| MP-07 | 2 | Permission persistence across rebuilds: ad-hoc vs. stable identity |
| MP-08 | 2 | Timing, repeat and media keys with controlled hardware input |
| MP-09 | 1 | Privacy lifecycle and status states as in WP-13 to WP-15; an existing OS grant does not start capture on launch |

### Linux (only if a test system ever exists)
LP-01 XInput2 raw events incl. grabs, repeat, device id. LP-02 `XQueryKeymap` resync. LP-03 Re-check portals and compositor interfaces for a sanctioned monitoring path.

### Controlled, data-minimal test rules (proposal; each prototype still needs approval)
1. Manual start, visible "capturing" indicator, automatic stop after a short time limit; no autostart, service or network use.
2. Scripted steps with prescribed keys ("press and hold left Shift, then Q"). The tool compares in memory against the expectation of the current step.
3. Output per step only: pass/fail, counts, timing summaries, API flags and device type. Events outside the expected set are counted without key identity. No event log, no typed text, no serial numbers or device instance IDs.
4. Exception for discovering unknown codes (knob, special keys): an explicit mode that shows the code of one deliberately pressed key on screen; saved only as a manually confirmed mapping "physical control -> code".
5. Close unrelated applications; never type real passwords; secure contexts use dummy keys; cancel UAC prompts instead of approving anything unneeded.
6. Never disable antivirus or OS protections; record detections as results.
7. Synthetic input may exercise the tool but never counts as hardware verification.
8. Afterwards remove granted permissions and prototype autostart traces; report environment, steps and results as `VER`.
9. Protections stay intact. If any route delivers input in a protected context, stop that route and keep only the outcome (no key identity). Never use any route, including HID callbacks, to recover protected input.
10. Report aggregate observations from prescribed dummy input only.

Whether tools shipped with Windows suffice (T-007 preference) was not researched; antivirus script scanning may affect a script-based prototype (`HYP`).

## 9. Gaps, contradictions and unavailable sources
| Topic | Status |
|---|---|
| Current upstream XI2 specification | Not retrieved (author: redirect not followed, mirror path 404; review: unavailable). Historical draft [X9] and a fork copy [X10] used |
| XI synchronous-grab issue [X1] | Not available as primary evidence |
| `XQueryKeymap` manual page, RECORD specification | Author session: fetch denied by the permission classifier, not retried. The review cites the Xlib specification for `XQueryKeymap` [X11]. RECORD remains unretrieved |
| `wl_keyboard` text | Author: only machine summaries [L1]; freedesktop GitLab refused automated access. The review confirms from the core XML that key delivery requires an active surface |
| Mutter sources | GitLab returned 406; read from the GitHub mirror instead. Access checker read by the review [L10] |
| Apple pages | Original input-monitoring guide URL returned HTTP 429 to the review (versioned guide [M26] used); `tapEnable` description not independently retrieved by the review |
| KWin source | Guessed mirror path returned 404; KWin claims are secondary [L7] |
| Microsoft "HID clients supported" page | 404; replaced by [W16] |
| Apple pages for `IOHIDCheckAccess`/`IOHIDRequestAccess` | 404; replaced by Apple's open-source header [M8] |
| Apple `CGPreflight/RequestListenEventAccess` pages | Retrieved, but contain no description |
| macOS 27 release notes | Partially read (section 5) |
| Elevated foreground window (both Windows APIs); Raw Input autorepeat and injected input | No source establishes the behaviour; test in T-007 |
| Long-running background behaviour (process throttling, power saving, App Nap), latency | Not researched; belongs to T-007/T-009 measurements |
| Remote desktop, virtual machines, games with exclusive fullscreen or anti-cheat | Not researched |
| Legacy `tapCreate` wording vs. newer Input Monitoring guidance | Resolved at documentation level [M24]; runtime behaviour open (section 5) |
| XI 2.0 vs. 2.1+ grab behaviour | Version change per historical draft [X9]; actual behaviour open. An older description [X4] `SEC‡` reflects 2.0 |
| X11 "any client" claim from a secondary article [X2] | Too broad given client authorization [X12]; withdrawn |
| GNOME X11 removal | Official GNOME 50 notes do not mention X11; plan [X7] and press report [X5] are confirmed by GNOME's release history [X13] |

## 10. Sources
Dates are page metadata or visible dates as reported by the fetch tool; "-" = none found. `(R)` = retrieved by the independent review (2026-11-25), not re-fetched by the author.

### Windows
| ID | Source | Date | Tag |
|---|---|---|---|
| W1 | [About Raw Input](https://learn.microsoft.com/en-us/windows/win32/inputdev/about-raw-input) | 2026-03-18 | DOC |
| W2 | [RAWINPUTDEVICE](https://learn.microsoft.com/en-us/windows/win32/api/winuser/ns-winuser-rawinputdevice) | 2024-06-26 | DOC |
| W3 | [RAWKEYBOARD](https://learn.microsoft.com/en-us/windows/win32/api/winuser/ns-winuser-rawkeyboard) | 2024-06-26 | DOC |
| W4 | [RAWINPUTHEADER](https://learn.microsoft.com/en-us/windows/win32/api/winuser/ns-winuser-rawinputheader) | 2024-02-22 | DOC |
| W5 | [RegisterRawInputDevices](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-registerrawinputdevices) | 2025-07-01 | DOC |
| W6 | [WM_INPUT_DEVICE_CHANGE](https://learn.microsoft.com/en-us/windows/win32/inputdev/wm-input-device-change) | 2025-07-16 | DOC |
| W7 | [LowLevelKeyboardProc](https://learn.microsoft.com/en-us/windows/win32/winmsg/lowlevelkeyboardproc) | 2025-07-16 | DOC |
| W8 | [KBDLLHOOKSTRUCT](https://learn.microsoft.com/en-us/windows/win32/api/winuser/ns-winuser-kbdllhookstruct) | 2024-02-22 | DOC |
| W9 | [GetAsyncKeyState](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getasynckeystate) | 2026-01-29 | DOC |
| W10 | [SetWindowsHookEx](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-setwindowshookexa) | 2025-07-01 | DOC |
| W11 | [Desktops](https://learn.microsoft.com/en-us/windows/win32/winstation/desktops) | 2025-04-15 | DOC |
| W12 | [How UAC works](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/user-account-control/how-it-works) | 2026-10-08 | DOC |
| W13 | [UI Automation security overview](https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-securityoverview) | 2026-02-19 | DOC |
| W14 | [WTSRegisterSessionNotification](https://learn.microsoft.com/en-us/windows/win32/api/wtsapi32/nf-wtsapi32-wtsregistersessionnotification) | 2025-07-01 | DOC |
| W15 | [About keyboard input](https://learn.microsoft.com/en-us/windows/win32/inputdev/about-keyboard-input) | 2026-07-08 | DOC |
| W16 | [Top-level collections opened by Windows](https://learn.microsoft.com/en-us/windows-hardware/drivers/hid/top-level-collections-opened-by-windows-for-system-use) | 2025-07-18 | DOC |
| W17 | [Microsoft criteria for malware and unwanted software](https://learn.microsoft.com/en-us/defender-xdr/criteria) | 2026-09-10 | DOC |
| W18 | [PowerToys and administrator mode](https://learn.microsoft.com/en-us/windows/powertoys/administrator) | 2026-04-22 | DOC |
| W19 | [AutoHotkey v2 FAQ, UAC](https://www.autohotkey.com/docs/v2/FAQ.htm) | - | SEC |
| W20 | [Lowk pull request 21](https://github.com/lifechangee/Lowk/pull/21) | 2026-11-04 | SEC |
| W21 | [Credential autofill restriction, January 2026](https://support.microsoft.com/en-us/servicing/os/windows/docs/2026/01/new-behavior-restricting-certain-applications-to-autofill-credentials-introduced-by-the-windows-janu) | 2026-01 | SEC‡ |
| W22 | [WM_WTSSESSION_CHANGE](https://learn.microsoft.com/en-us/windows/win32/termserv/wm-wtssession-change) (R) | - | DOC |

### macOS
| ID | Source | Date | Tag |
|---|---|---|---|
| M1 | [CGEvent.tapCreate](https://developer.apple.com/documentation/coregraphics/cgevent/tapcreate(tap:place:options:eventsofinterest:callback:userinfo:)) | - | DOC |
| M2 | [CGEventTapOptions.listenOnly](https://developer.apple.com/documentation/coregraphics/cgeventtapoptions/listenonly) | - | DOC |
| M3 | [CGPreflightListenEventAccess](https://developer.apple.com/documentation/coregraphics/cgpreflightlisteneventaccess()), [CGRequestListenEventAccess](https://developer.apple.com/documentation/coregraphics/cgrequestlisteneventaccess()) | 10.15+ | DOC |
| M4 | [Apple forum 707680, DTS answer](https://developer.apple.com/forums/thread/707680) | 2022-06 | SEC |
| M5 | [Apple forum 122492, community](https://developer.apple.com/forums/thread/122492) | 2019-2021 | SEC |
| M6 | [Mac User Guide: input monitoring](https://support.apple.com/guide/mac-help/control-access-to-input-monitoring-on-mac-mchl4cedafb6/mac) | 2026 | DOC |
| M7 | [NSEvent global monitor](https://developer.apple.com/documentation/appkit/nsevent/addglobalmonitorforevents(matching:handler:)) | - | DOC |
| M8 | [IOHIDLib.h](https://raw.githubusercontent.com/apple-oss-distributions/IOKitUser/main/hidsystem.subproj/IOHIDLib.h) | main branch | DOC |
| M9 | [IOHIDManager.h](https://raw.githubusercontent.com/apple-oss-distributions/IOKitUser/main/hid.subproj/IOHIDManager.h) | main branch | DOC |
| M10 | [CGEvent.tapEnable](https://developer.apple.com/documentation/coregraphics/cgevent/tapenable(tap:enable:)), [tapDisabledByTimeout](https://developer.apple.com/documentation/coregraphics/cgeventtype/tapdisabledbytimeout), [tapDisabledByUserInput](https://developer.apple.com/documentation/coregraphics/cgeventtype/tapdisabledbyuserinput) | - | DOC |
| M11 | [CGEventSource.keyState](https://developer.apple.com/documentation/coregraphics/cgeventsource/keystate(_:key:)) | - | DOC |
| M12 | CGEventField [autorepeat](https://developer.apple.com/documentation/coregraphics/cgeventfield/keyboardeventautorepeat), [keycode](https://developer.apple.com/documentation/coregraphics/cgeventfield/keyboardeventkeycode), [keyboard type](https://developer.apple.com/documentation/coregraphics/cgeventfield/keyboardeventkeyboardtype); [flagsChanged](https://developer.apple.com/documentation/coregraphics/cgeventtype/flagschanged) | - | DOC |
| M13 | [maskSecondaryFn](https://developer.apple.com/documentation/coregraphics/cgeventflags/masksecondaryfn), [systemDefined](https://developer.apple.com/documentation/appkit/nsevent/eventtype/systemdefined) | - | DOC |
| M14 | [TN2150 Secure Event Input (archived)](https://developer.apple.com/library/archive/technotes/tn2150/_index.html) | 2007-06-08 | DOC |
| M15 | [Ghostty discussion 14232](https://github.com/ghostty-org/ghostty/discussions/14232) | 2026-09-14 | SEC |
| M16 | [Apple forum 795739, DTS answer](https://developer.apple.com/forums/thread/795739) | 2025-08 | SEC |
| M17 | [Safely open apps on your Mac](https://support.apple.com/en-us/102445) | 2026-05-27 | DOC |
| M18 | [Developer Program enrollment](https://developer.apple.com/programs/enroll/) | - | DOC |
| M19 | [macOS 27 release notes](https://developer.apple.com/documentation/macos-release-notes/macos-27-release-notes) | beta text | DOC† |
| M20 | [Apple forum 858535](https://developer.apple.com/forums/thread/858535) | recent | SEC |
| M21 | [KeyPath pull request 1379](https://github.com/malpern/KeyPath/pull/1379) | 2026-11-09 | SEC |
| M22 | [Third-party permission guide](https://coreydaley.dev/guides/mac-permissions/) | 2026 | SEC |
| M23 | [Developer account overview](https://developer.apple.com/support/compare-memberships/), [Developer ID](https://developer.apple.com/support/developer-id/) | 2026 | DOC |
| M24 | [Apple WWDC19 session 701](https://developer.apple.com/videos/play/wwdc2019/701/) (R) | 2019 | DOC |
| M25 | [Terminal User Guide: secure keyboard entry](https://support.apple.com/en-asia/guide/terminal/trml109/mac) (R) | - | DOC |
| M26 | [macOS 27 user guide: input monitoring](https://support.apple.com/en-asia/guide/mac-help/mchl4cedafb6/27/mac/27) (R) | macOS 27 | DOC |

Apple reference pages M1-M3, M7, M10-M13 and M19 were read through their JSON data endpoints (`developer.apple.com/tutorials/data/...json`).

### Linux
| ID | Source | Date | Tag |
|---|---|---|---|
| X1 | [Xorg server issue 876](https://gitlab.freedesktop.org/xorg/xserver/-/issues/876) | - | SEC‡ |
| X2 | [Linux keylogging article](https://linuxsecurity.com/features/complete-guide-to-keylogging-in-linux-part-2) | - | SEC‡ |
| X3 | [Wayland book: X11 application support](https://wayland.freedesktop.org/docs/book/Xwayland.html) | - | DOC |
| X4 | [Who-T: XI2 recipes part 4](http://who-t.blogspot.com/2009/07/xi2-recipes-part-4.html) | 2009-07 | SEC‡ |
| X5 | [The Register: GNOME 50](https://www.theregister.com/software/2026/03/19/gnome-50-debuts-with-x11-axed-wayland-front-and-center/5224877) | 2026-03-19 | SEC |
| X6 | [KDE Plasma 6.8 announcement](https://kde.org/announcements/plasma/6/6.8.0/) | 2026-10-14 | DOC |
| X7 | [GNOME blog: X11 session removal](https://blogs.gnome.org/alatiera/2025/06/08/the-x11-session-removal/) | 2025-06-08 | SEC |
| X8 | [X.Org XISelectEvents](https://xorg.freedesktop.org/archive/X11R7.5/doc/man/man3/XISelectEvents.3.html) (R) | X11R7.5 | DOC |
| X9 | [Historical XI2 protocol draft](https://cgit.freedesktop.org/xorg/proto/inputproto/tree/specs/XI2proto.txt?h=multitouch-devel#n4540) (R) | historical | DOC (draft) |
| X10 | [Later XI2 specification copy, personal fork](https://raw.githubusercontent.com/rwiggins/xorgproto/master/specs/XI2proto.txt) (R) | - | SEC |
| X11 | [Xlib specification](https://www.x.org/releases/X11R7.6/doc/libX11/specs/libX11/libX11.html) (R) | X11R7.6 | DOC |
| X12 | [X.Org SECURITY extension](https://xorg.freedesktop.org/archive/X11R7.7/doc/xextproto/security.html) (R) | X11R7.7 | DOC |
| X13 | [gnome-session 50.0 release history](https://raw.githubusercontent.com/GNOME/gnome-session/50.0/NEWS) (R) | 50.0 | DOC |
| L1 | [wayland.app core protocol](https://wayland.app/protocols/wayland), [protocol XML mirror](https://raw.githubusercontent.com/wayland-mirror/wayland/main/protocol/wayland.xml) | - | DOC† |
| L2 | [GlobalShortcuts portal](https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.GlobalShortcuts.html) | v2 | DOC |
| L3 | [InputCapture portal](https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.InputCapture.html) | v2 | DOC |
| L4 | Mutter mirror: [a11y D-Bus XML](https://raw.githubusercontent.com/GNOME/mutter/main/data/dbus-interfaces/org.freedesktop.a11y.xml), [a11y manager source](https://raw.githubusercontent.com/GNOME/mutter/main/src/backends/meta-a11y-manager.c) | main branch | DOC |
| L5 | [systemd udev default rules](https://raw.githubusercontent.com/systemd/systemd/main/rules.d/50-udev-default.rules.in) | main branch | DOC |
| L6 | Kernel docs: [event codes](https://docs.kernel.org/input/event-codes.html), [input](https://docs.kernel.org/input/input.html) | 7.4.0-rc4 | DOC |
| L7 | [Linneman Labs post](https://linnemanlabs.com/posts/hello-my-name-is-orca/) | 2026-05-15 | SEC |
| L8 | [org.freedesktop.login1](https://www.freedesktop.org/software/systemd/man/latest/org.freedesktop.login1.html) | systemd 262 | DOC† |
| L9 | [KWin merge request 1950](https://invent.kde.org/plasma/kwin/-/merge_requests/1950), [KDE bug 474705](https://bugs.kde.org/show_bug.cgi?id=474705) | - | SEC‡ |
| L10 | [Mutter D-Bus access checker source](https://raw.githubusercontent.com/GNOME/mutter/main/src/backends/meta-dbus-access-checker.c) (R) | main branch | DOC |

## 11. Independent technical review (Codex, 2026-11-25)

> History note (added at consolidation): the review text below is unchanged and describes the state at review time, before consolidation and acceptance. Current wording is in sections 1-10; dispositions are in section 12.

**Recommendation. Accept T-005 as desk research after adopting the corrections below in the verdict, matrix and test plan.** This review supplies the corrected interpretation for downstream work. Product Owner acceptance is pending. All findings come from retrieved documentation or source, with inferences and runtime questions identified below. No `VER` results.

### Findings requiring correction

- **High — Windows elevation and protected desktops (C14–C16, W9/W11/W12/W18).** Mark delivery to a standard-user app while an elevated foreground window is on the **same desktop** as unknown for both APIs. W18 describes specific PowerToys functions. W9 documents restrictions on a separate polling API. Neither establishes Raw Input or low-level hook delivery in this case. Microsoft documents same-desktop scope for hooks, a single active input desktop, and default UAC prompts on the secure desktop. Treat capture on the protected desktop as unavailable in normal use and test the transitions and recovery. Ordinary password fields carry no verified suppression guarantee. Exclude `uiAccess` because Microsoft reserves it for relevant assistive technology and requires trust and protected installation. The source does not establish a mandatory fee for every signed deployment. [PowerToys](https://learn.microsoft.com/en-us/windows/powertoys/administrator) · [GetAsyncKeyState](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getasynckeystate) · [Desktops](https://learn.microsoft.com/en-us/windows/win32/winstation/desktops) · [UAC](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/user-account-control/how-it-works#securing-the-elevation-prompt) · [UI Automation security](https://learn.microsoft.com/en-us/windows/win32/winauto/uiauto-securityoverview)
- **High — Status and privacy (C06–C07, section 7).** Distinguish user opt-in, registration or permission state, and observed event delivery. An idle keyboard cannot establish an interruption. Report known interruptions and clear highlights on verified transition signals. `GetAsyncKeyState` returns state per virtual key and can return zero when access is blocked. It cannot establish a healthy stream or per-device state. Replace the claim of exposure to the “whole” stream with exposure to the keystrokes the OS actually delivers, potentially including ordinary password-field input. PS-1–PS-3 require visible opt-in and pause behavior. A persistent OS grant does not authorize starting capture on launch. [LowLevelKeyboardProc](https://learn.microsoft.com/en-us/windows/win32/winmsg/lowlevelkeyboardproc) · [GetAsyncKeyState](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getasynckeystate) · [Windows session notifications](https://learn.microsoft.com/en-us/windows/win32/termserv/wm-wtssession-change) · [Approved privacy scenarios](../../brain/PROJECT.md#privacy-scenarios-apply-to-m1-and-m2-c)
- **Medium — macOS permission and protection (M1/M4/M6/M14).** The legacy OS X wording in `tapCreate` conflicts with Apple's official Catalina guidance for a listen-only tap, which explicitly specifies Input Monitoring and distinguishes it from an active tap's Accessibility authorization. Use WWDC19 as `DOC` for that documented model. Apple DTS corroborates it and describes the preflight/request APIs. The macOS 27 user guide confirms the user-facing Input Monitoring control. It does not settle the exact tap behavior on the target build. TN2150 documents Secure Event Input for the intercept mechanisms it describes, including event taps. Apple also documents Terminal's secure-entry protection. Exact behavior of a modern listen-only tap, its enabled state, and especially non-seizing HID callbacks remain runtime questions. The Screen Recording signing example is indirect evidence for Input Monitoring persistence. The original M6 URL returned HTTP 429 here, and the versioned Apple guide was retrieved. The `tapEnable` description could not be independently retrieved here. The timeout event definition was retrieved. [Legacy API reference data](https://developer.apple.com/tutorials/data/documentation/coregraphics/cgevent/tapcreate%28tap%3Aplace%3Aoptions%3Aeventsofinterest%3Acallback%3Auserinfo%3A%29.json) · [Apple WWDC19](https://developer.apple.com/videos/play/wwdc2019/701/) · [Apple DTS](https://developer.apple.com/forums/thread/707680) · [macOS 27 user guide](https://support.apple.com/en-asia/guide/mac-help/mchl4cedafb6/27/mac/27) · [TN2150](https://developer.apple.com/library/archive/technotes/tn2150/_index.html) · [Terminal guide](https://support.apple.com/en-asia/guide/terminal/trml109/mac) · [Timeout definition](https://developer.apple.com/tutorials/data/documentation/coregraphics/cgeventtype/tapdisabledbytimeout.json) · [Signing thread](https://developer.apple.com/forums/thread/795739)
- **Medium — X11 evidence and scope (C02–C11, C16).** Retrieved X.Org material supports passive subscription to root-window raw press/release events on the connected X server. The archived XI2 draft describes the 2.0 grab restriction and 2.1 change, keycode, `sourceid` from 2.1, and device hierarchy changes. A later spec copy in a personal fork corroborates the version history. The current upstream spec and the cited synchronous-grab issue were unavailable as primary evidence here. Leave actual grab/freeze behavior, raw repeat, multi-device behavior and screen lockers open. `XQueryKeymap` is documented and yields logical state that may lag when event processing is frozen. Scope E5 plausibility to an authorized client in a native X11 session with XI2 available. X.Org documents restricted client authorization as well, so claims about any client or the absence of OS protections are too broad. GNOME's tagged release history and KDE's announcement confirm retirement of their native X11 sessions. Linux stays `Unverified`. [X.Org XISelectEvents](https://xorg.freedesktop.org/archive/X11R7.5/doc/man/man3/XISelectEvents.3.html) · [Historical XI2 draft](https://cgit.freedesktop.org/xorg/proto/inputproto/tree/specs/XI2proto.txt?h=multitouch-devel#n4540) · [Later spec copy](https://raw.githubusercontent.com/rwiggins/xorgproto/master/specs/XI2proto.txt) · [Xlib specification](https://www.x.org/releases/X11R7.6/doc/libX11/specs/libX11/libX11.html) · [X.Org SECURITY](https://xorg.freedesktop.org/archive/X11R7.7/doc/xextproto/security.html) · [GNOME release history](https://raw.githubusercontent.com/GNOME/gnome-session/50.0/NEWS) · [Plasma 6.8](https://kde.org/announcements/plasma/6/6.8.0/)
- **Medium — Wayland verdict (L1–L6/L8).** Use “No general E5-compliant passive background path demonstrated for the live viewer.” The core XML confirms key delivery requires an active surface. GlobalShortcuts reports registered shortcuts. InputCapture is armed for later compositor activation and redirects input during capture. GNOME's current source restricts its accessibility monitor to an authorized service. The default systemd device group and session-controller API are outside the approved normal-app route. Xwayland remains subject to compositor delivery and cannot establish general access to native Wayland input. These findings rule out the reviewed routes. Compositor-specific extensions and policies were not surveyed exhaustively. Focused training remains a distinct, documented use of the core protocol. Carry the finding as an E5 gap for this feature and leave the support-label decision with the Product Owner. [Core XML, project mirror](https://raw.githubusercontent.com/wayland-mirror/wayland/main/protocol/wayland.xml) · [GlobalShortcuts](https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.GlobalShortcuts.html) · [InputCapture](https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.InputCapture.html) · [Mutter monitor](https://raw.githubusercontent.com/GNOME/mutter/main/src/backends/meta-a11y-manager.c) · [Mutter access check](https://raw.githubusercontent.com/GNOME/mutter/main/src/backends/meta-dbus-access-checker.c) · [systemd device rules](https://raw.githubusercontent.com/systemd/systemd/main/rules.d/50-udev-default.rules.in) · [logind](https://www.freedesktop.org/software/systemd/man/latest/org.freedesktop.login1.html) · [Xwayland architecture](https://wayland.freedesktop.org/docs/book/Xwayland.html)

### Confirmed comparison and T-007 implications

- **Windows comparison.** Raw Input is the better-documented first test candidate for passive operation. `RIDEV_INPUTSINK` enables asynchronous background delivery to a registered window. `RAWINPUTHEADER` identifies the generating device, and registration supports device-change notifications and removal. Only one target window per device class can register within a process. A `WH_KEYBOARD_LL` callback runs on the installing thread with a message loop and can suppress or delay delivery. Its event structure has injection and release flags and no device identifier. Timeout can silently remove the hook. Treat it as a comparison for KO-2 and keep Raw Input repeat/injection behavior, rotary events and recovery open for testing. Direct keyboard HID collections are opened exclusively by Windows. Shared consumer-control collections establish no knob event guarantee. [Raw Input registration flags](https://learn.microsoft.com/en-us/windows/win32/api/winuser/ns-winuser-rawinputdevice) · [RegisterRawInputDevices](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-registerrawinputdevices) · [Raw Input header](https://learn.microsoft.com/en-us/windows/win32/api/winuser/ns-winuser-rawinputheader) · [LowLevelKeyboardProc](https://learn.microsoft.com/en-us/windows/win32/winmsg/lowlevelkeyboardproc) · [Hook data](https://learn.microsoft.com/en-us/windows/win32/api/winuser/ns-winuser-kbdllhookstruct) · [HID collections](https://learn.microsoft.com/en-us/windows-hardware/drivers/hid/top-level-collections-opened-by-windows-for-system-use)
- **Windows trial.** Keep WP-01–WP-12 as the test basis. Make the first trial a standard-user comparison against foreground typing with capture off. Check initial non-registration, explicit activation, both pause controls, immediate clearing and no further processing, restart to off, and repeated activation (PS-1–PS-3 and the capture part of PS-8). Check the deactivation shortcut for interference with normal typing. Test raised foreground privilege separately from UAC, lock and secure-desktop transitions. For recovery, press in the ordinary desktop and release during the transition, then check clearing and correct handling of fresh events on return. Treat protected delivery and hook parity as observations. Base the capture go/no-go on whether a candidate meets KO-1's capture scope, KO-2 and KO-3. Use aggregate observations and prescribed dummy input with protections intact.
- **macOS trial.** The documented listen-only candidate justifies the planned variant after Windows, subject to recording the actual Mac and OS build and obtaining the separate approvals. Specify expected permission-denied, granted and revoked states, ordinary background delivery before and after Secure Event Input, and recovery with no stuck keys. Any unexpected delivery in a protected context is a protection finding. Stop that route and retain only the outcome. Do not use HID callbacks to recover protected input. Test timing, device identity, repeat and media keys with controlled hardware input. The approval and startup behavior remains version-sensitive.
- **Gate scope.** The comparison leaves the Windows capture portion of KO-1 and the runtime behavior for KO-2/KO-3 open pending T-007. The focused input and fullscreen parts of KO-1 require separate evidence. The capture lifecycle trial informs KO-4. Training-path isolation, persistence and transmission still require product verification. macOS and Linux affect the optional-platform assessment. The approved Windows acceptance scope and KO-5–KO-7 stand. No option has been scored. [Approved evaluation frame](T-004-evaluation-frame.md)

## 12. Disposition of review findings (Claude Code, 2026-11-25)
The Product Owner accepted the corrected research results and approved closing T-005 as a research task (2026-11-25). All findings of section 11 were adopted. Sources added by the review were not re-fetched.

| Review finding | Severity | Adopted in |
|---|---|---|
| Elevated same-desktop delivery unknown for both APIs; protected desktop treated as unavailable; password fields without verified suppression; `uiAccess` reasoning corrected | High | Sections 2, 3 (C14-C16), 4, 7, 9 |
| Separate opt-in, registration/permission and observed delivery; `GetAsyncKeyState` is no health check; "whole stream" narrowed; OS grant does not start capture | High | Sections 3 (C07), 4, 7; WP-15, MP-09 |
| macOS: Input Monitoring for listen-only taps documented (WWDC19); legacy wording marked; secure-input runtime questions; signing evidence indirect; retrieval limits | Medium | Sections 2, 3, 5, 9, 10 |
| X11: documented sources added, scope limited to an authorized native session, broad claims withdrawn, runtime points open | Medium | Sections 2, 3, 6, 7, 9, 10 |
| Wayland: "no general E5-compliant passive background path demonstrated"; support label left to the Product Owner | Medium | Sections 2, 3, 6, 7 |
| Windows comparison and trial design; go/no-go basis; gate scope | - | Sections 2, 4, 7, 8 (WP-01 to WP-15) |
| macOS trial design and protection rule | - | Section 8 (MP-01 to MP-09, rules 9-10) |

Consolidation choices by the author, not worded this way in the review:
- The Wayland verdict is expressed as `Unknown` on the four-value scale of T-005, replacing `Not feasible`; the E5 cell reads "gap" instead of "not met".
- WP-13 to WP-15 and MP-08/MP-09 turn the review's trial text into numbered claims. Their priorities are the author's proposal.
