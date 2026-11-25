# T-006 Event Model and Keyboard Behaviour

- Status: ACCEPTED by the Product Owner as completed documentation research (2026-11-25, D-010); no separate independent review; technical hypotheses remain open for T-007 | Author: Claude Code | Date: 2026-11-25
- Added at acceptance: user-reported hardware configuration (sections 5 and 12), preliminary product decisions (10.1), configuration notes for the scenarios (section 11). User statements are not verification results; evidence tags are unchanged.
- Scope: how a physical keyboard action reaches the app, what the profiled keyboards can send, and a **proposed**, technology-independent event and profile model.
- Not decided here: tech stack, capture API, storage format, UI presentation, requirement changes.
- **Nothing was tested.** No `VER` result. Nothing was installed, no code was written, no keyboard or OS setting was changed.
- All sources accessed 2026-11-25 (section 14). Builds on [T-005](T-005-global-capture.md).

## 1. Evidence rules
| Tag | Meaning |
|---|---|
| `DOC` | Official specification, vendor/OS documentation, or the vendor's own source code, retrieved |
| `SEC` | Secondary source (third-party mirror, archived personal vendor blog, search excerpt) |
| `HYP` | Author's inference or assumption; not stated by a retrieved source |
| `T:WP-xx` | Needs a practical test; scenario in section 11 |

- `DOC` describes what a source says. **No behaviour in this report was observed.**
- Firmware source code shows what that source version would do. It does not show what the user's unit runs.
- Specifications, Kinesis PDFs and firmware files were downloaded and the relevant passages read as text by the author. Web pages marked `(H)` in section 14 were read through a helper agent; quotes are as it returned them.
- "Control" = one operable element drawn on screen (key, rotary direction, rotary push, pedal).

## 2. Summary
1. **The app can only see what the OS receives.** Firmware decides what a physical control sends: nothing, one code, several codes, a delayed code, or a code shared with another control. This holds for every software stack (KO-1 stack-independent limit; TR-1 confirmed at documentation level).
2. **Four identities must stay separate:** control (where the user pressed), key code (what arrived), legend (what is drawn), character (what was typed). None determines another reliably.
3. **Key codes name positions on a reference keyboard, not on the user's board.** HID assigns usages by typical position and leaves characters to the host layout [H1]. Programmable firmware may put any code on any control.
4. **A highlight can only mean** "a signal bound to this control arrived". Profiles therefore need per-control detectability, many-to-many bindings and visible ambiguity.
5. **The live path needs no character translation.** The training path is character-based and takes text from its own focused window; the physical key is at most an inferred hint.
6. **Several approved requirements cannot be met literally** for all keys of the profiled keyboards. They are listed in section 10 for the Product Owner; nothing was changed.

## 3. From physical key to character
```
control -> switch/encoder -> FIRMWARE -> HID report -> OS driver / remappers -> platform key code
        (opaque: layers, tap-hold,    (usages)                                  |
         macros, combos, profiles)                       live path stops here --+
                                                                                 |
                     host layout + modifiers + dead keys / IME -> character  (training path only)
```

### 3.1 USB HID (as far as it matters)
- Usages are meant to be position-stable across languages: the closest equivalent position is used and only keycaps differ (example: German Z sends the usage named "Y") [H1] `DOC`.
- A keyboard report lists the keys currently pressed, not press/release codes. The host derives transitions by comparing reports; order inside one report is undefined; auto-repeat is implemented by the host [H2] `DOC`.
- Boot-style reports carry six non-modifier keys. Beyond the report capacity, or for an unrecognizable combination, the keyboard reports `ErrorRollOver` [H2] `DOC`. Devices with N-key rollover use other report formats `HYP`.
- Media functions (mute, volume) are usually on the Consumer usage page, separate from the keyboard page [H1, A3] `DOC`.
- Two controls sending the same usage on one device probably cannot be told apart or counted, because the report states which usages are down `HYP` (T:WP-22).

### 3.2 Per operating system
| Level | Windows 11 | macOS | Linux (not verifiable here) |
|---|---|---|---|
| Position-like code | Scan code with E0/E1 prefix; identifies the key "regardless of the active keyboard layout" [W1] `DOC` | Virtual key code, "hardware-independent" [M1] `DOC`; constants identify physical keys, `ANSI` names follow US positions [M5] `SEC` | evdev key code [L1] `DOC` |
| Layout-dependent key | Virtual-key code, defined by the system for the key's purpose [W1] `DOC` | No separate stable layer used here; `charactersIgnoringModifiers` is character data [M2] `DOC` | keysym [L2] `DOC` |
| Character | Character and dead-key messages produced via the layout [W1]; `ToUnicodeEx` [W3] `DOC` | `NSEvent.characters` (empty for dead keys) [M2]; layout translation with dead-key state and keyboard type [M3] `DOC` | Layout translation from key code to keysym [L2] `DOC` |
| Composed text / IME | Keyboard messages go to the focused window [W1] `DOC`; IME path not researched | Text input client protocol for text views [M4] `DOC`; relation to key events not retrieved | Not researched |
| Repeat | Host-generated [H2]; identification per capture API: T-005 C08 | autorepeat field (T-005 C08) | value 2 [L1] `DOC` |
| Lost events | Overrun make code in raw input [W2] `DOC`; see T-005 | See T-005 | `SYN_DROPPED`: discard and query state [L1] `DOC` |
| System-level remapping | Scancode map (system-wide, all keyboards, reboot) applied before virtual-key conversion [W7] `DOC`; PowerToys remaps through hooks [W8] `DOC`. What a capture API then sees: `HYP` | HID-usage remapping without special privileges, lost at restart [M6] `DOC` | Not researched |

W3C UI Events draws the same line between `code` (physical location) and `key` (locale and layout dependent) and notes that Fn typically generates no code [U1] `DOC`. It is cited as vocabulary support only, not as a technology choice.

### 3.3 Platform pitfalls relevant to the model
| Pitfall | Evidence | Consequence |
|---|---|---|
| Windows maps HID `0x31` (US backslash) and `0x32` (non-US `#`) to the **same** scan code `0x2B`; the ISO `<>` key (`0x64`) is `0x56` [W1] | `DOC` | Native code to HID usage is not unique; profile geometry must resolve it |
| Pause uses an `E1` sequence; Print Screen differs with Alt; Ctrl+Pause becomes Break [W1] | `DOC` | Adapter must normalize multi-part codes (T:WP-17) |
| AltGr arrives as left Ctrl plus right Alt on layouts with AltGr [W6] `SEC`, [W8] `DOC` | | Left Ctrl may light when only AltGr was pressed (T:WP-17) |
| Media buttons may carry no scan code [W2] (sample comment) | `DOC` | Bindings must allow another signal namespace (T:WP-18) |
| `ToUnicodeEx` changes keyboard state affecting dead keys unless flag bit 2 is set (Windows 10 1607+) [W3] | `DOC` | Do not translate captured events to characters in the live path |
| Keyboard layout is per thread [W4] | `DOC` | The app's layout is not proof of the foreground app's layout |
| Navigation and keypad keys share virtual keys depending on Num Lock; scan codes differ [W1] | `HYP` for the Num Lock effect | Bind position codes, not virtual keys (T:WP-17) |
| macOS: ISO section and grave key codes may appear swapped on some third-party ISO keyboards | `HYP`, no source retrieved | T:MP-10 |

## 4. Firmware: what the host cannot know
QMK feature behaviour [Q1-Q11] `DOC` applies to Lemokey (QMK source, [K1]) and ZSA [Z1, Z2]; only features actually configured take effect. Kinesis SmartSet offers comparable features [A1, A2]. Where "nothing arrives" follows from a function's purpose rather than an explicit statement, it is `HYP`.

| Feature | What arrives | Live-view effect |
|---|---|---|
| Momentary or toggled layer key, Fn | Nothing for the layer change itself [Q2, U1] | Layer key cannot light; active layer unknown |
| Layer content | The code of the active layer | Same control, different codes; same code from different controls |
| Tap-hold (mod-tap, layer-tap, tap-and-hold) | Events delayed until tap/hold is decided; default term 200 ms in QMK [Q1] | Delayed highlight; tap appears after release; a layer hold shows nothing |
| Macro, send-string, status report | A key sequence unrelated to the pressed control [Q7, A1] | Wrong or flickering highlights |
| Shifted or multi-modifier keycode (`(`, Hyper, Meh, shortcuts) | Several codes; shifted symbols are Left Shift plus the unshifted US code [Q11] | Several other controls light, not the pressed one |
| Combo, tap dance, key override, one-shot, auto shift, caps word | Replaced, added, delayed or time-dependent codes [Q3-Q6, Q8] | Control-to-code relation depends on timing and history |
| Encoder rotation | One press/release pair per step [Q9] | Momentary impulse; no "held" state |
| Firmware-only functions (lighting, radio, NKRO toggle, profile switch) | Nothing | Not detectable |
| Disabled key (e.g. Win lock) | Nothing | Looks like a key that was not pressed |
| Unicode input modes | OS-specific key sequences [Q7] | Many codes for one character |

Vendor tools may show the active layer (ZSA Keymapp does [Z8] `DOC`); how it obtains it is not documented on the pages read. Standard key reports carry only pressed usages [H2], so a generic app should assume **no layer information** (`HYP`). Vendor protocols are "later" scope (F1, F6).

## 5. Keyboard profiles and limitations
"Default" below means the cited vendor source. "User statement" is the Product Owner's report of 2026-11-25 (section 12); no device was inspected or tested.

### 5.1 Lemokey P1 Pro, ISO (M1)
Source: vendor firmware, branch `wireless_playground`, commit `d3c413b` (2024-08) [K1] `DOC`. keychron.com refused automated fetches (HTTP 429).

| Property | Finding |
|---|---|
| Geometry | `iso_encoder`: 83 matrix positions = 82 keys plus knob push; coordinates and widths in `info.json`; ISO Enter |
| German-relevant keys | All present as codes: grave, minus, equal, both brackets, semicolon, quote, non-US hash, non-US backslash |
| Layers | 4 defined (base, Fn, two spare that are transparent in the default keymap). No separate Mac base layer in this source |
| Knob, base | Rotate: volume down/up; push: mute. Delay between a step's down and up events configured as 2 ms [Q9] |
| Knob, Fn | Rotate: lighting only (nothing sent) |
| F-row | Base F1-F12; Fn: brightness, task view, file explorer, lighting, media, mute, volume |
| Composite output | Task view and file explorer send two-key shortcuts (Win+Tab, Win+E) |
| Disable | Holding the Fn+Win combination (3 s configured) toggles a Win key lock |
| Rollover | NKRO enabled in the definition; Fn toggle exists; wireless NKRO defined |
| Connections | USB, 2.4 GHz, Bluetooth (mode switch). USB vendor/product ID `362D:0304`, device version 1.0.2 in source |
| Other | Vendor raw HID channel enabled (configuration tools); out of scope |

Limitations
- L1 Fn sends nothing; Fn lighting, radio, battery and NKRO functions send nothing.
- L2 Fn+F3 would light Win and Tab, not Fn and F3.
- L3 Mute can come from the knob push or Fn+F10: ambiguous.
- L4 Knob steps are impulses; very short pairs can be invisible without a minimum display time. Delivery under fast rotation unknown (T:WP-18, WP-23).
- L5 Knob has three controls (counter-clockwise, clockwise, push) in the model.
- L6 Behaviour per connection mode and device identity unknown (T:WP-20).
- User statement: ISO-DE; factory keymap; connection changes between the available connection types; no Mac/Windows switch known to the user.
- Open: firmware version; shipped firmware vs. this source (a factory keymap does not prove they match); whether an OS-mode switch exists.

### 5.2 Kinesis Advantage360 SmartSet (M1)
Sources: manual 2022 edition (firmware 1.0.9) [A1], programming guide 2025-08 (1.0.79) [A2], token list [A3], quick start [A4]. Support page lists firmware 1.0.80 [A5] `DOC`.

| Property | Finding |
|---|---|
| Geometry | 76 keys, split, two thumb clusters [A6]; Space, Backspace, Enter among thumb keys [A6, A7]. **Exact positions and default codes are only in graphics that could not be read**; not recorded here |
| Profiles / layers | 10 profiles (0 fixed factory, 1-9 programmable); layers base, keypad, Fn, Fn2, Fn3; default layout uses the first three [A1] |
| Layer keys | Two Fn keys act momentarily; keypad key toggles [A1] |
| F-keys, keypad | On layers, e.g. Fn plus the `=` key gives F1 [A1] |
| Special keys | SmartSet key not programmable; four hotkeys blank by default [A1] |
| Programmability | Every other key in every layer; macros (about 300 actions, speed, repeat, delays), tap-and-hold, multi-modifiers, consumer and mouse actions [A1-A3] |
| Rollover | Default six simultaneous keys; NKRO by shortcut, reset on re-plug or v-Drive access [A1] |
| Regional variant | "Only being sold in a US-QWERTY configuration" per vendor FAQ [A7] (page may reuse Advantage2 text); status report shows "Advantage 360 US" [A1] |
| ISO extra key | Action for non-US backslash exists [A3]; a default key for it was not found |
| Host side | Generic OS drivers; remapping software may conflict [A1] |

Limitations
- A1 Fn keys and presumably the keypad and SmartSet keys send nothing (`HYP` for what is sent; layer behaviour `DOC`).
- A2 Active profile and layer are shown on keyboard LEDs [A1]; assume the app cannot read them (`HYP`).
- A3 Layer-dependent codes: a received F-key or keypad code identifies a control only if the profile knows the layer binding.
- A4 Remaps, macros and tap-and-hold in profiles 1-9 are invisible to the app and can make the prebuilt profile wrong.
- A5 With NKRO off, more than six non-modifier keys cannot be reported.
- A6 Onboard actions can emit output (status report types text) or change modes without host-visible keys.
- A7 For QWERTZ-DE, characters on the ISO `<>|` position need an assignment on this board; whether one exists in the user's profile is unknown.
- User statement: individual remaps or macros are in use; Windows keyboard layout "English, QWERTY"; physical US/QWERTY keycaps; unknown whether a separate key for `<`, `>`, `|` exists.
- Open: firmware version, active profile, which controls are remapped or carry macros, exact English layout variant (T:WP-16, WP-21, WP-22, WP-25). Because of the customisations, vendor defaults must not be taken as this unit's expected codes (A4).

### 5.3 ZSA Voyager (M2)
Sources: vendor firmware fork, branch `firmware25`, commit `35b6b2a` [Z1]; product page [Z3]; ZSA articles [Z5-Z7] `DOC`.

- Geometry: 52 keys, split, two thumb keys per half; no encoder in the firmware definition.
- Repository default keymap (3 layers; **not necessarily the Oryx or shipped layout**): thumb layer-taps and mod-taps; mod-taps on letters and edge keys; Shift/Backspace on two controls; caps word; layer 1 symbols as shifted US codes; layer 2 media and navigation.
- USB `3297:1977`; NKRO enabled as a feature; up to 32 layers advertised [Z3].
- Firmware sends codes; the OS decides characters. For German output the OS layout must match; Oryx offers language keycodes [Z5, Z6].

Limitations
- V1 Layer holds send nothing; layer state unknown.
- V2 Tap-hold keys: delayed, and one control yields two different codes.
- V3 Duplicate codes across controls and layers (e.g. Shift/Backspace twice; digits on two layers).
- V4 Default symbol keys send Shift plus a US code. On a German host they light two other controls and produce different characters than the US legend.
- V5 Fewer keys than a German layout needs: in the default keymap the Ü position exists only on layer 1 and the `<>|` code was not found.
- V6 Every user layout differs; a prebuilt profile is a starting point only.
- User statement: own Oryx layout with an individual, mixed key assignment. The repository default keymap (V4, V5) therefore does not describe this unit. Open: actual bindings, whether language-specific (German) keycodes are used, firmware version.

### 5.4 ZSA Moonlander (M2, not hardware-verified)
Source: [Z2] `DOC`; no test hardware, so every behaviour stays unverified.
- Geometry: 72 keys in the main layout; no encoder in the definition; hardware revisions A and B with different USB product IDs (`1969`, `1972`).
- Repository default keymap (3 layers): layer toggles on both halves, Hyper and Meh keys, mod-taps, duplicate Backspace and arrow codes, a vendor pairing key, a version-string macro.
- Limitations: as V1-V4 and V6, plus multi-modifier keys lighting several controls.

### 5.5 Generic TKL (M2)
- A nominal profile, not a device: variants (ISO/ANSI, knob) are open product decisions.
- Standard keys can be prefilled from HID usages and platform tables [H1, W1] `DOC`; key count and legends per variant are `HYP` until defined.
- Fn behaviour, media keys, rollover and extra controls are vendor-specific and unknown. Mark Fn "not detectable" by default (`HYP`).
- ANSI backslash and ISO `#` share a Windows scan code (3.3); geometry decides which control is drawn.

### 5.6 Comparison
| Limit | P1 Pro | Adv360 | Voyager | Moonlander | TKL |
|---|---|---|---|---|---|
| Controls that send nothing | Fn | Fn, likely kp/SmartSet | layer holds | layer keys | likely Fn |
| Layer-dependent codes | yes | yes | yes, heavy | yes, heavy | vendor |
| Delayed codes by default | none found | none documented as default | yes | yes | unknown |
| Duplicate codes by default | mute (knob, Fn) | unknown | yes | yes | unknown |
| Rotary | yes | no (not documented) | no | no | open |
| User can remap in firmware | yes | yes | yes | yes | vendor |

## 6. Proposed event model (concept, not a decision)
### 6.1 Principles
1. Observed signals only; never claim physical switch state or active firmware layer.
2. Position-like native codes are the identity; virtual keys and characters are not used for matching.
3. Ambiguity and unknown are states of their own, never silently guessed.
4. Two data paths with separate types; no conversion between them.

### 6.2 Live path
| Item | Fields (conceptual) | Notes |
|---|---|---|
| `KeySignal` | signal id (namespace, code, prefix); transition down/up; source ref (opaque, session-only, may be unknown); origin hardware/injected/unknown; receipt time | No character, no text, no virtual-key meaning. Never persisted |
| `ImpulseSignal` | as above, momentary | Optional: an adapter may instead deliver a normal down/up pair |
| `InputAnomaly` | kind: overrun/rollover, unrecognized | No key identity stored |
| `StreamReset` | reason: disabled, paused, permission or registration lost, session/desktop/lock transition, sleep/resume, secure input, device removed, overrun, profile or binding change | Clears state |
| `CaptureStatus` | intent (off/on/paused); registration (none/active/failed/known lost); observation (nothing yet / last receipt) | The three T-005 states; idle is not an interruption |

Signal namespaces are platform-specific (e.g. Windows scan code, macOS key code, HID consumer usage). A canonical id based on HID usages is proposed for profiles; the translation may return **more than one** candidate (3.3).

### 6.3 Pressed state and presentation
- State = set of `(source, signal)` currently down; in memory only.
- down while already down: repeat, not a new press. up without down: ignore.
- Presentation states: `pressed`, `held` (still down after a display threshold or repeat seen), `released` (short fade). Thresholds are UI parameters (T-009).
- **Minimum display time:** any down/up pair stays visible long enough to be perceived, without changing the logical state (knob steps, taps, macros).

### 6.4 Lost events and recovery
1. Clear everything on every `StreamReset`; start empty afterwards.
2. Clear a key when its up arrives, its source disappears, or a later down shows it was stale.
3. Candidate: while keys are believed down, reconcile them against the platform's key-state query and **only clear, never add**. Queries have limits (Windows returns zero when access is blocked; not per device) and never prove a healthy stream (T-005) (T:WP-25).
4. No unconditional timeout: long holds are legitimate. If testing shows undetectable losses, a stale-key rule needs a Product Owner trade-off.
5. Pause/disable must clear and stop processing immediately (PS-3).

### 6.5 Training path
| Item | Content | Source |
|---|---|---|
| `TextCommitted` | resulting characters, time | focused app window only |
| `Composition` | pending dead key or IME preview | focused app window |
| `EditAction` | delete, confirm, abort | focused app window |
| `FocusedKey` (optional) | position-like code, transition | focused app window; guidance and shortcuts |
| `FocusChange` | pauses the exercise (PS-5) | window |

- Evaluation compares **committed text** with expected text. Key presses and characters are not one-to-one (dead keys, AltGr, Shift, macros, combos, IME).
- Events and timings live only during the exercise; only aggregates persist (D-003). Sufficiency of aggregates: T-010.
- Expected-key guidance is derived: expected character -> (key layout) code chord -> (profile bindings) control(s), possibly "needs layer" or "not on this keyboard".
- Statistics per key are statements about the **expected** key, not the observed finger or switch.

### 6.6 Privacy boundary
| | Live path | Training path |
|---|---|---|
| Scope | system-wide, opt-in | own focused window, active exercise |
| Data | codes and transitions | text, composition, optional codes |
| Characters | never derived | yes |
| Persistence | none | aggregates only |
| Cross-use | never feeds training | never uses live data |

- Legends come from static layout data plus user overrides, never from translating captured input.
- No logging of signals, including unknown ones, in logs, crash dumps or IPC (TR-4).
- If both paths run in one process, they must not share the event channel; Windows allows one raw-input registration per device class per process (T-005 W5), so coexistence needs a test (T-005 WP-11).

## 7. Proposed profile model
| Entity | Content |
|---|---|
| `KeyboardProfile` | id, schema version, name, model, based-on, verification status per OS and firmware ("not hardware-verified"), references below |
| `Geometry` | controls with stable id, kind (key, rotary direction, rotary push, pedal), position, size, rotation, shape (ISO Enter), group (half, thumb cluster) |
| `BindingSet` | named configuration variant (factory base, Fn layer, user profile); informational activation note; the app does not know which is active |
| `Binding` | control id -> emission, with provenance `documented`, `recorded` (on this device) or `assumed`, per platform where needed |
| `Emission` | `code` / `codes` (chord) / `impulse` / `sequence` (not reconstructable) / `none` (firmware-only) / `unknown` |
| `KeyLayout` | code -> legends per modifier level and dead-key flag; also serves character -> chord for training. QWERTZ-DE, QWERTY-US; further layouts are data |
| `LegendOverride` | control id -> user label; changes drawing only |
| `FingerAssignment` | control id -> finger (M2, separate) |

- Geometry, key layout, training language and finger assignment stay independent (F6). Training language is not part of a profile.
- Four things called "layout" differ and need distinct names: **firmware keymap** (device), **host layout** (OS), **legend layout** (app setting), **training language** (content). The app cannot change the first two; a mismatch between host and legend layout makes legends wrong.

### 7.1 Resolving a signal to controls
| Candidates | Proposed behaviour |
|---|---|
| exactly one | highlight |
| several (duplicates, several binding sets) | mark all as ambiguous, or a user-chosen primary plus muted others (presentation open) |
| found only in a non-base set | highlight that control as "inferred"; do not claim the layer key or a persistent layer |
| chord binding | first version: no time-based grouping, each code resolves alone (grouping cannot separate firmware chords from real ones) |
| none | neutral "unmapped input" hint without showing or storing the code |

### 7.2 Manual mapping ("learn" mode)
1. Explicit mode in the focused app; user selects a control, then presses it.
2. Record the signal(s) in the **same namespace the live path delivers**; focused-window key events may differ or be intercepted by the OS (T:WP-28).
3. Several codes: store as chord or sequence and warn. Nothing observed: offer "not detectable", worded as "no signal observed".
4. Rotary: record each direction and push separately. Repeat per binding set (the user switches layers on the keyboard).
5. Persist only the confirmed `control -> code` configuration (see RC-06). Never write to the keyboard.

## 8. Layouts: QWERTZ-DE, QWERTY-US, custom
- Same position code, different legend and character: usage "Y" is Z on a German host [H1, Q10] `DOC`.
- US boards on a German host: the code, not the printed cap, decides the character. Boards that send US-oriented shifted codes produce other symbols (V4).
- DE needs positions that compact boards place on layers or omit (V5, A7). Training must handle "reachable only via layer" and "not available".
- Custom configurations are expressed as binding sets (what the firmware sends), legend overrides (what is drawn) and, for new character mappings, further `KeyLayout` data. Remapping in firmware (e.g. hardware Dvorak on a US host) is valid and needs bindings, not a new host layout.

## 9. Implications
**Visualization**
- Draw non-detectable controls as such, so silence is not read as an error.
- Show ambiguity and "inferred" distinctly; never animate an assumed layer.
- Apply a minimum display time; knob direction needs its own visual.
- With several keyboards connected, input from all may arrive; filtering needs device identity from the chosen capture path.
- Latency can only be measured from event receipt; firmware delay is outside the app.

**Layout-dependent typing training**
- Character-based scoring is robust against geometry and firmware; position guidance depends on correct host layout and profile bindings.
- A setup check ("press these keys") can confirm that host layout, profile and keyboard agree, inside the focused app and without storing text (`HYP` design).
- The app cannot know which finger or, on programmable boards, always which switch was used.

## 10. Requirement conflicts for the Product Owner
No requirement was changed. Suggested for Gate G2, after T-007.

| ID | Requirement | Conflict | Proposal |
|---|---|---|---|
| RC-01 | F1, M1 "highlights pressed keys" | Fn, layer, SmartSet and firmware-only keys cannot be highlighted | Apply acceptance to controls with an OS-visible signal; mark others |
| RC-02 | F1 only pressed keys shown | Duplicates, layers, chords and macros can light several or other controls | Accept visible ambiguity; no layer guessing |
| RC-03 | NF-Performance "perceived as immediate" | Tap-hold and combos delay output in firmware | State that targets apply from event receipt |
| RC-04 | M1 simultaneous keys correct | Limited by device rollover (Adv360 default: six) | "Correct up to what the device reports" |
| RC-05 | F1 "selected keyboard" | Other keyboards' input is delivered too | Decide: show any input, or filter where device identity exists |
| RC-06 | NF-Privacy no recording | Manual and learn mapping stores key codes | Confirm: deliberate configuration is not input recording |
| RC-07 | F5/F6 DE and US on all geometries | Some characters need a layer or are unavailable on a configuration | Lessons must handle availability; no firmware changes |
| RC-08 | F6 independent legend layout | Legends can contradict the host layout | Allow, but warn in training |
| RC-09 | M1 pressed/held/released | Rotary steps and taps have no perceivable hold | Define impulse display |
| RC-10 | F5/F8 weakness analysis | Physical key and finger are inferred, not measured | Describe statistics as expected-key based |

### 10.1 Preliminary Product Owner decisions (2026-11-25, D-010)
Preliminary: the affected requirements are adjusted finally at Gate G2, using practical findings. [PROJECT.md](../../brain/PROJECT.md) is unchanged.

| ID | Preliminary decision | Still open |
|---|---|---|
| RC-01 | Hardware and firmware limits of detectability are accepted in principle | Acceptance wording; how non-detectable controls are marked |
| RC-02 | The app must not feign unambiguous physical key detection when received events are ambiguous | Presentation of ambiguity and "inferred"; the options in 7.1 must satisfy this rule |
| RC-04 | Hardware limits on simultaneously detectable keys are to be taken into account | Actual limits per board and connection (T:WP-21); wording |
| RC-05 | Selecting the active input device is desirable if technically reliably possible | Feasibility per capture path (T:WP-26); behaviour if not reliable |
| RC-06 | Storing manually defined key-code mappings as profile configuration is allowed; it is not recording input histories | Wording at G2 |

No decision yet: RC-03, RC-07, RC-08, RC-09, RC-10.

## 11. Controlled test scenarios for T-007
Test rules of [T-005 section 8](T-005-global-capture.md#8-prototype-claims-and-test-method) apply unchanged (rules 1-10). Additionally:
- Test the keyboards **as currently configured**. No firmware update, remap, v-Drive access, profile, NKRO, Win-lock or radio change. A scenario needing a settings change (including the OS input language) requires separate approval, or is skipped.
- Output per step: expected vs. observed code, flags, count, down/up pairing, coarse timing. Codes are shown only for the prescribed control (T-005 rule 4).
- P = priority. Boards: L = P1 Pro, K = Adv360, V = Voyager. Record firmware version and connection mode where known.

| ID | P | Boards | Scenario | Question / expected observation |
|---|---|---|---|---|
| WP-16 | 1 | L, K | Prescribed position keys (`^ ß ´ Ü + # < Ö Ä Y Z` positions); optionally repeat under US host layout | Scan code and flags stable across layouts while virtual key changes; match [W1] |
| WP-17 | 1 | L, K | AltGr, Shift+navigation, keypad vs. navigation with Num Lock on/off, Pause, Print Screen | Synthetic or extra codes, order, how to recognize them |
| WP-18 | 1 | L | One knob step each way, push, fast spin (volume will change) | Interface and code per direction; pairs per step; lost steps; both candidate paths |
| WP-19 | 1 | L | Fn alone; Fn+F3; Fn+F10 vs. knob push | Silence of Fn; composite codes; duplicate mute. Do not use Fn with Win, N, 1-4 |
| WP-20 | 2 | L | Cable vs. receiver vs. Bluetooth (needs switching approval) | Same codes; device identity; behaviour when switched with a key held |
| WP-21 | 2 | L, K | Hold increasing sets of prescribed keys | Maximum reported; overrun signal; state afterwards |
| WP-22 | 2 | any with duplicates | Two controls with the same code: hold both, release one | Events and resulting state |
| WP-23 | 1 | L, K | Shortest taps and knob steps | Down-to-up duration distribution, input for minimum display time |
| WP-24 | 2 | V, K if configured | Tap-hold and layer key: tap, hold, roll | Order and delay of arriving codes |
| WP-25 | 1 | L, K | Hold a key and unplug; sleep; toggle layer while holding | Missing or mismatched up events; whether state query clears them |
| WP-26 | 2 | two boards | Alternate input; count interfaces per physical board (no serials or instance IDs in output) | Device distinction on each candidate path |
| WP-27 | 1 | L | Dead-key and AltGr text in a foreground editor with capture on; same input in a test window | Capture does not alter other apps' text (KO-2); key events vs. committed characters |
| WP-28 | 2 | L, K | Learn mapping of prescribed controls, repeat after re-plug | Codes stable; focused-window and background codes comparable |
| MP-10 | 2 | L | macOS: ISO grave and `<` positions | Key codes and keyboard type |
| MP-11 | 2 | L, K | macOS: knob, Fn combinations, AltGr/Option positions | Delivered events and codes |

Not testable now: Moonlander (no hardware). Voyager needs to be connected. Kinesis scenarios depend on knowing the active profile.

Configuration notes from the user statements (section 12); scenarios and priorities are unchanged, T-007 preparation decides adjustments:
- Ask for firmware versions before the respective tests.
- Adv360: expected codes depend on the user's remaps and macros, which must be described first. The German position names in WP-16 need restating for this board (US keycaps, English host layout).
- P1 Pro: connections change in normal use, which bears on the priority of WP-20.
- Voyager: WP-24 needs the relevant bindings of the user's Oryx layout.

## 12. Information needed from the user
| ID | Question | Needed for |
|---|---|---|
| Q1 | P1 Pro: keymap changed with Launcher/VIA? Firmware version if shown? Usual connection? Any OS-mode switch on the device? | 5.1, WP-18 to WP-20 |
| Q2 | Adv360: active profile, own remaps or macros, firmware version. Optional read-only source: status report (SmartSet + Right Ctrl + Right Shift + `/` into an empty editor) [A1] | 5.2, WP-21 to WP-25 |
| Q3 | Adv360: which OS layout do you use with it, and is there a key producing `<>\|`? German keycaps? | 8, RC-07 |
| Q4 | Voyager: own Oryx layout or factory default? German keycodes? | 5.3, WP-24 |
| Q5 | Is remapping software active on the PC (PowerToys, AutoHotkey, vendor tools)? | WP-16, WP-17 |

### 12.1 Answers (Product Owner statement, 2026-11-25; not verified)
| ID | Reported | Still unknown |
|---|---|---|
| Q1 | ISO-DE; factory keymap; changing connections across the available types; no Mac/Windows switch known | Firmware version |
| Q2 | Individual remaps or macros | Active profile, affected controls, firmware version |
| Q3 | Windows layout "English, QWERTY"; physical US/QWERTY keycaps | Exact layout variant; whether a separate key for `<`, `>`, `\|` exists |
| Q4 | Own Oryx layout; individual, mixed assignment | Actual bindings, use of German keycodes, firmware version |
| Q5 | No active remapping software known | Not checked on the system |

Firmware versions may be requested before the respective tests.

## 13. Gaps and limitations
- Kinesis default layout and position maps are graphics; no renderer was available, nothing was installed. Thumb-key order and default codes are missing.
- Lemokey: no official manual retrieved; one firmware branch and commit read, not matched to the shipped device. Third-party manual excerpts were not used as evidence.
- ZSA: Oryx default and factory layouts not retrieved; repository defaults used. Keymapp transport not found.
- HID usage tables 1.5 used; a newer revision was not checked.
- Not researched: IME details on all platforms, macOS key code to HID mapping table, how Windows delivers consumer controls to each capture API, Raw Input behaviour with system remappers, Linux beyond the basics, accessibility sticky/filter keys, remote desktop.
- Section 6 and 7 are the author's proposals (`HYP` as design), not decisions. The Product Owner waived a separate full review; critical assumptions are to be checked during T-007 preparation. Requirement changes still need approval.

## 14. Sources
`(H)` = read through a helper agent.

| ID | Source | Version / date |
|---|---|---|
| H1 | [HID Usage Tables](https://usb.org/sites/default/files/hut1_5.pdf) | 1.5 |
| H2 | [HID Device Class Definition](https://www.usb.org/sites/default/files/hid1_11.pdf), appendices C and F | 1.11, 2001 |
| W1 | [About keyboard input](https://learn.microsoft.com/en-us/windows/win32/inputdev/about-keyboard-input) incl. scan code table (H; table rows also read directly) | 2026-07-08 |
| W2 | [RAWKEYBOARD](https://learn.microsoft.com/en-us/windows/win32/api/winuser/ns-winuser-rawkeyboard) (H) | 2024-06-26 |
| W3 | [ToUnicodeEx](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-tounicodeex) (H) | 2025-07-01 |
| W4 | [GetKeyboardLayout](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getkeyboardlayout) (H) | 2025-07-01 |
| W6 | [Hot keys and AltGr](https://learn.microsoft.com/en-nz/archive/blogs/murrays/hot-keys-and-altgr), archived blog (H) | 2015-04-20 |
| W7 | [Keyboard and mouse class drivers, scan code mapper](https://learn.microsoft.com/en-us/windows-hardware/drivers/hid/keyboard-and-mouse-class-drivers), archived (H) | 2024-07-19 |
| W8 | [PowerToys Keyboard Manager](https://learn.microsoft.com/en-us/windows/powertoys/keyboard-manager) (H) | 2026-07-30 |
| M1 | [CGKeyCode](https://developer.apple.com/documentation/coregraphics/cgkeycode) (H) | - |
| M2 | NSEvent [characters](https://developer.apple.com/documentation/appkit/nsevent/characters), [charactersIgnoringModifiers](https://developer.apple.com/documentation/appkit/nsevent/charactersignoringmodifiers) (H) | - |
| M3 | [UCKeyTranslate](https://developer.apple.com/documentation/coreservices/1390584-uckeytranslate) (H, partly truncated) | - |
| M4 | [NSTextInputClient](https://developer.apple.com/documentation/appkit/nstextinputclient) (H) | - |
| M5 | [Events.h, third-party SDK mirror](https://raw.githubusercontent.com/phracker/MacOSX-SDKs/master/MacOSX10.13.sdk/System/Library/Frameworks/Carbon.framework/Versions/A/Frameworks/HIToolbox.framework/Versions/A/Headers/Events.h) (H) | SDK 10.13 |
| M6 | [TN2450 Remapping keys](https://developer.apple.com/library/archive/technotes/tn2450/_index.html), archived (H) | 2017-08-21 |
| L1 | [Linux input event codes](https://docs.kernel.org/input/event-codes.html) (H) | 7.4.0-rc4 |
| L2 | [xkbcommon introduction](https://xkbcommon.org/doc/current/xkb-intro.html) (H) | current |
| U1 | [UI Events code values](https://www.w3.org/TR/uievents-code/) (H) | Rec. 2025-04-22 |
| Q1-Q11 | QMK docs at [commit 80973aa](https://github.com/qmk/qmk_firmware/tree/80973aa0b5413e9a5bbbf9293507f39e870f9aa3/docs): Q1 `tap_hold.md`, Q2 `feature_layers.md`, Q3 `features/combo.md`, Q4 `features/tap_dance.md`, Q5 `features/key_overrides.md`, Q6 `one_shot_keys.md`, Q7 `features/send_string.md` and `features/unicode.md`, Q8 `features/auto_shift.md` (caps word: seen only in [Z1]), Q9 `features/encoders.md`, Q10 `reference_keymap_extras.md`, Q11 `keycodes_us_ansi_shifted.md` | commit 2026-11-18 |
| K1 | [Keychron firmware, Lemokey P1 Pro](https://github.com/Keychron/qmk_firmware/tree/d3c413bfd0a727eded97bb2089904e869d504c25/keyboards/lemokey) (`p1_pro`, `common`) | `d3c413b`, 2024-08-21 |
| A1 | [Advantage360 SmartSet user manual](https://kinesis-ergo.com/wp-content/uploads/Advantage360-SmartSet-KB360-Users-Manual-v10-12-22.pdf) | 2022-10-12 |
| A2 | [Direct programming guide](https://kinesis-ergo.com/wp-content/uploads/Adv360-SmartSet-Direct-Programming-Guide-Version-8-8-25.pdf) | 2025-08-08 |
| A3 | [Action token list](https://kinesis-ergo.com/wp-content/uploads/Adv360-SmartSet-Direct-Programming-Action-Tokens-v3-31-23.pdf) | 2023-03-31 |
| A4 | [Quick start guide](https://kinesis-ergo.com/wp-content/uploads/Advantage360-SmartSet-QSG-v7-25-22.pdf) | 2022-07-25 |
| A5 | [Support page](https://kinesis-ergo.com/support/kb360/) (H) | firmware 1.0.80 titled 7-19-26 |
| A6 | [Product page](https://kinesis-ergo.com/adv360/) (H) | - |
| A7 | [Getting started and FAQ](https://kinesis-ergo.com/advantage360-getting-started/) (H) | - |
| Z1 | [ZSA firmware, Voyager](https://github.com/zsa/qmk_firmware/tree/35b6b2a8cc2b9ad25d5c00c06dec6f16128c9bad/keyboards/zsa/voyager) | `35b6b2a`, 2026-11-24 |
| Z2 | [ZSA firmware, Moonlander](https://github.com/zsa/qmk_firmware/tree/35b6b2a8cc2b9ad25d5c00c06dec6f16128c9bad/keyboards/zsa/moonlander) | same |
| Z3 | [Voyager page](https://www.zsa.io/voyager) (H, truncated) | - |
| Z5 | [Setting up international layouts](https://blog.zsa.io/international-setup/) (H) | 2023-03-07 |
| Z6 | [Keys not matching output](https://blog.zsa.io/keys-not-matching-output/) (H) | 2023-10-15 |
| Z7 | [Where is the Fn key](https://blog.zsa.io/where-fn-key/) (H): QMK has no macOS Fn key in Oryx | 2024-03-07 |
| Z8 | [Keymapp](https://www.zsa.io/keymapp) (H) | - |
