# Cornix -> Orbee feature parity migration

Status: Phase 0 inventory complete; candidate mapping documented; implementation not started

## Goal

Port the current user-facing Cornix behavior set from `kanclaus/Cornix_ZMK` to Orbee while preserving Orbee as the source of truth for hardware-specific definitions.

This is **not** a wholesale copy of Cornix firmware.

- Cornix = source of behavior / user experience
- Orbee = source of hardware truth

## Frozen baselines

### Cornix

- repo: `kanclaus/Cornix_ZMK`
- branch: `main`
- baseline: `9fd24c56c8d00daed65efe78748bb0f47e218e44`

### Orbee

- repo: `kanclaus/zmk-config-Orbee_rev.2`
- branch: `main`
- baseline: `f627f76a11943c17ff6d11dcf801d66889593934`

If either baseline changes before implementation, record the new SHA and re-check the migration assumptions.

## Non-negotiable rules

1. Preserve Orbee matrix, pin, split, PMW3610, sensor and board definitions unless a later task explicitly requires a hardware change.
2. Do not modify Cornix; it is read-only reference material.
3. Do not silently drop Cornix functions because Orbee has fewer physical keys.
4. Build a physical-position mapping before editing Orbee keymap bindings.
5. Cornix hardware-specific LED/VBUS code must be adapted, not copied blindly.
6. Keep Orbee trackball functionality while adding Cornix keyboard-driven mouse layers where technically compatible.
7. Do not mix DYA/ZMK Studio enablement into this migration.
8. No direct pushes to `main`; implementation work must use a fresh branch and PR.
9. Keep implementation phased and buildable; avoid one giant migration PR.
10. Static/build success must not be reported as physical-device proof.

## Current Orbee baseline observations

- board: `seeeduino_xiao_ble`
- shields: `Orbee_L`, `Orbee_R`
- keymap: `config/Orbee.keymap`
- layout metadata: `config/Orbee.json`
- hardware transform / sensors: `config/boards/shields/Orbee/Orbee.dtsi`
- PMW3610 pointing is enabled on the right side
- current PMW3610 CPI baseline is 400
- EC11 encoder definitions exist
- RGB LED widget module is present
- Orbee follows ZMK `main`

## Current Cornix feature inventory

The current Cornix firmware is substantially more than a keymap.

### Keymap / layer behavior

- Windows BASE
- Mac BASE
- LOWER / RAISE / ADJUST
- NAVI / NUM
- COMBO
- MOUSE
- precision mouse
- scroll layer
- Mac LOWER / RAISE / ADJUST
- Mac window/navigation layers
- Mac mouse / natural-scroll / precision layers
- active layer-taps for G/H, C, V, and Question
- unused home-row / shift hold-tap and Backspace/Delete mod-morph definitions, called out separately below
- C-hold Alt+Tab / Cmd+Tab behavior
- Dock-number mod-morphs
- combos
- encoder bindings

### Persistent user-facing state

- Mac/Win mode saved independently for BT0-BT4
- keyboard-layout mode saved per BT profile
- layout cycle: `US -> JIS -> ISO -> US`
- saved mode re-applied on profile selection
- latest Cornix baseline includes retry handling for persistent settings writes

### Dynamic macro system

Five persistent slots:

- tap: play
- hold 500 ms: start/stop record
- stop after 10 s inactivity
- persisted data / integrity handling

### Indicator / battery / power behavior

- manual battery preview
- LED on/off state
- LED brightness state
- charging indication
- full-charge indication
- no automatic low-battery red blink
- six-hour inactivity system-off
- USB/VBUS-powered sleep suppression
- split/BLE/behavior/HID queue headroom and stability tuning

## Phase 0 verified inventory

### Frozen build and active layer set

The source audit used the frozen Cornix config SHA and its pinned board module noted above. `build.yaml` emits the daily `cornix_left` and `cornix_right` `indicator_led_debug` images plus a separate right-side `settings_reset` utility image. The daily wrapper `config/indicator_led_debug.keymap` defines the live LED, macro, Mac/Win, layout, and symbol behaviors before including `config/cornix.keymap`.

The active keymap has 19 layers. IDs are from the constants in `config/cornix.keymap`:

| ID | Layer | User-facing role |
| ---: | --- | --- |
| 0 | BASE | Windows/default layer |
| 1 | MAC | Mac base layer |
| 2 | LOWER | Symbols and punctuation |
| 3 | RAISE | Function and number layer |
| 4 | ADJUST | Adjustment/navigation layer |
| 5 | NAVI | Navigation shortcuts |
| 6 | NUM | C-hold app/window switching layer |
| 7 | COMBO | Bluetooth, profile mode, macro, battery and LED controls |
| 8 | MOUSE | Keyboard-driven pointing and clicks |
| 9 | MPREC | Precision pointing |
| 10 | MSCRL | Keyboard-driven scrolling |
| 11 | MAC_LOWER | Mac symbol layer |
| 12 | MAC_RAISE | Mac function/number and Dock launcher layer |
| 13 | MAC_ADJUST | Mac adjustment layer |
| 14 | MAC_WINR | Mac spaces, tabs, and application navigation |
| 15 | MAC_WINL | Mac Rectangle-style window snapping |
| 16 | MAC_MOUSE | Mac pointing layer |
| 17 | MAC_MSCRL | Mac natural-scroll layer |
| 18 | MAC_MPREC | Mac precision pointing |

### Active and declared behaviors

- Active tap/hold behavior includes G/H layer-taps, V and Question mouse-layer taps, and C-hold Alt+Tab on Windows or Cmd+Tab on Mac. `alt_layer` and `cmd_layer` hold the corresponding modifier while the shared NUM layer is active.
- The MAC_WINL layer has four two-key Hyper shortcuts for top-left, top-right, bottom-left, and bottom-right placement, each with a 50 ms combo timeout.
- `dock0` through `dock9` are mod-morphs used by MAC_RAISE: ordinary digits remain taps; holding Ctrl produces the Mac Hyper+digit launcher chord.
- The COMBO layer selects BT0–BT4, clears a bond/profile, toggles the saved Mac/Win mode and keyboard layout, selects five dynamic macro slots, previews battery, and controls LED power/brightness. In the frozen Cornix keymap it is entered by holding C0, whose physical position is omitted in Orbee; the layer access must be reassigned or explicitly excluded before parity work.
- `config/cornix.keymap` also defines `hm`, `hm_l`, `hm_r`, `hm_shift_l`, `hm_shift_r`, and `bs_del`, but the frozen active layers do not reference them. The BASE/MAC bindings use direct keycodes for those positions. Treat home-row mods, shift hold-taps, and Backspace/Delete mod-morph as dormant definitions, not current parity requirements, unless a later scope decision activates them.
- The keymap defines additional layer-tap and mouse actions at C, V, and Question; the full physical binding list is in [CORNIX_TO_ORBEE_POSITION_MAP.md](CORNIX_TO_ORBEE_POSITION_MAP.md).

### Per-profile mode and keyboard layout

`cornix_layout_mode.c` stores Mac/Win state and US/JIS/ISO state independently for all five BLE profiles. Changing the active profile reapplies its saved mode and shows LED feedback. Layout cycling is US → JIS → ISO → US; US and ISO keep the keymap fallback HID codes, while the custom symbol behavior translates 19 layout-sensitive symbols for JIS. The current implementation retries a failed settings write up to three times at three-second intervals.

### Dynamic macros

The daily build enables five persistent slots. A tap plays the slot; holding for 500 ms starts or stops recording; recording stops after 10 seconds without key activity. Each slot records up to 256 key events and playback spaces events by 30 ms. The implementation validates CRC/version data, migrates v3 slot data, splits storage into settings-sized chunks, retries failed writes every five seconds, cleans up held playback keys, and reports record/save/timeout/empty state through the indicator.

### Mouse, precision, scroll, and encoders

The Cornix mouse layers are keyboard-driven and coexist conceptually with a physical pointing device: holding V enters pointing; I/J/K/L move the cursor, D supports held left-click dragging, and M/comma/period click the left/middle/right buttons. Holding C while pointing enters precision mode; holding X enters scroll mode. Source speeds are normal 800, precision 320, and scroll 45. Mac variants invert the encoder wheel direction for natural scrolling.

`cornix_live_mouse.c` tracks up to four held directions by position and split source. When precision mode changes, it updates the live movement vectors without releasing and repressing the held directions. The source keymap uses two Cornix EC11 sensors: general layers bind volume and wheel actions; window layers bind brightness and wheel actions; Mac layers reverse the wheel direction.

### Battery, charge, indicator, and VBUS

The default daily indicator mode drives two WS2812 pixels per half. The renderer owns the LED strip and EXT_POWER rail; standard ZMK RGB underglow is disabled. The COMBO layer provides LED on/off, brightness down/up, manual battery preview, profile/mode/layout controls, and macro slots. Hue/effect controls in the shared keymap compile to no-op bindings for this daily indicator image. Live feedback covers active/open/connected BLE profiles, split-link state, mode/layout changes, activity, and macro recording/playback.

Charging uses a 2.8-second fade cycle; battery level at or above 97% shows full-charge feedback for 2.5 seconds. VBUS falls back to a five-second poll when no edge-event API is available. The live-status default does not automatically blink red for low battery; a separate battery-low test mode exists in Kconfig but is not the daily default. LED brightness defaults to 10%, uses steps 1/3/6/10/20/40/70/100%, and saves the selected state.

The daily build enters system-off after six hours idle and a key wakes it. It suppresses system-off while USB power is present. The left/central half uses ZMK USB connection state; the right/peripheral half disables the USB device stack and reads nRF VBUS directly through the Cornix helper.

### BLE, split, queues, and patch dependencies

The frozen configs `config/cornix.conf`, `config/cornix_left.conf`, `config/cornix_right.conf`, `config/boards/shields/indicator_led_debug/indicator_led_debug.conf`, and `config/indicator_led_debug_cornix_left.conf` / `config/indicator_led_debug_cornix_right.conf` set 10 ms press/release debounce, KSCAN queue 16, behavior queue 128, hold-tap capture 64, input queue 32, and separate modifier-release reports. BLE settings include TX power +8, peripheral interval 6–12, latency 30, timeout 400, GATT caching, experimental connection support, 1024-byte BLE thread stack, and HID queues 96/48/48. Central split queues are 32 positions, 4 battery entries, and 24 split-run entries with a 1024-byte stack and 600 ms preference timeout; peripheral position queue is 64 with a 1024-byte stack. The left USB path adds 200 ms attach delay and queue size 64; the right USB device stack is disabled. ZMK Studio is disabled.

The source carries six behavior/pipeline patches that must be reviewed against Orbee’s own ZMK revision before any reuse: `patches/0001-cornix-ll-prepare-pipeline-mitigation.patch`; `patches/zephyr/0001-Bluetooth-Controller-Fix-missing-radio-tmr-status-re.patch`; and `patches/zmk/0001-Cornix-keep-battery-reporting-fresh-while-USB-powered.patch`, `0002-Cornix-let-indicator-renderer-own-ext-power-rail.patch`, `0003-Cornix-retain-split-position-state-on-queue-overflow.patch`, and `0004-Cornix-route-HID-reports-to-captured-profile.patch`. The Cornix `config/west.yml` pins Zephyr `10ba6d0cb38bc3d258775d27982f707599320085`, hal_stm32 `4fcc3a3f32abe1c4cb76d9d1cef967728dd03908`, LVGL `f1db87ee98f1810328a8419572fa42a3b5f352ae`, zmk-studio-messages `6cb4c283e76209d59c45fbcb218800cd19e9339d`, ZMK `773dec58eaacaef4703b3e4595e50bd71f6cad3d`, zmk-keyboard-cornix `3318e486328fd2024233b73b1b8ba8887006c112`, and zmk-helpers `bc114546392b4615ac90a99140eaf21dde31209d`. These are source dependencies, not safe-to-copy Orbee patches.

### Custom source, bindings, Kconfig, and inactive source

| Source area | Inventory | Current-build status |
| --- | --- | --- |
| `indicator_led_debug.c` | WS2812 renderer, state persistence, profile/mode/layout/charge/battery status | Compiled in daily indicator image |
| `cornix_layout_mode.c` | Per-profile OS/layout state, symbol translation, settings retries | Compiled in daily indicator image |
| `cornix_live_mouse.c` | Held movement vector and precision-switch behavior | Compiled in daily indicator image |
| `cornix_macro.c` | Five-slot persistent macro recorder/player | Compiled when `CONFIG_CORNIX_MACRO` is enabled; default y |
| `cornix_activity.c`, `cornix_vbus.h` | USB/VBUS query shim for the no-USB peripheral | Activity shim compiled for the right image; header shared |
| `cornix_diag.c` | Watchdog/fatal/hang/power/reset archive and optional LED/HID diagnostics | File is present but not listed by this shield CMakeLists; not part of the audited daily build |

The relevant source files are `config/cornix.keymap`, `config/indicator_led_debug.keymap`, `config/includes/cornix54.h`, `config/boards/shields/indicator_led_debug/CMakeLists.txt`, `config/boards/shields/indicator_led_debug/Kconfig.shield`, and its `src/` files listed above. The behavior YAMLs under `config/dts/bindings/behaviors/` are `cornix,behavior-battery-preview.yaml`, `cornix,behavior-diag-dump.yaml`, `cornix,behavior-diag-show.yaml`, `cornix,behavior-layoutmode-toggle.yaml`, `cornix,behavior-led-setting.yaml`, `cornix,behavior-led-sync.yaml`, `cornix,behavior-live-mouse-move.yaml`, `cornix,behavior-live-mouse-precision.yaml`, `cornix,behavior-macmode-toggle.yaml`, `cornix,behavior-macro-slot.yaml`, `cornix,behavior-mode-sync.yaml`, and `cornix,behavior-symbol.yaml`; the vendor prefix is registered in `config/dts/bindings/vendor-prefixes.txt`. The indicator Kconfig defines live-status and test modes plus the LED and macro timings/capacities; it does not add an active diagnostics build option. `cornix_diag.c` contains watchdog/fatal/hang/power/reset capture and optional LED/HID replay code, but its source and diag binding YAMLs are not part of the audited daily build.

### Orbee hardware and feature preservation inventory

| Orbee baseline item | Frozen main observation to preserve |
| --- | --- |
| Board and split | `seeeduino_xiao_ble`; Orbee_L is split peripheral and Orbee_R is split central |
| Matrix | Shared 4-row, 11-column transform and 43 key positions; shared row GPIOs are XIAO D1/D2/D3/D6; left columns use D10/D9/D8/D7/GPIO0_10/GPIO0_9 and right columns use D10/D9/D8/D7/GPIO0_10 with `col-offset=6` |
| PMW3610 | Right-side SPI0 pointing device; preserve CS GPIO0_9, IRQ GPIO0_2, SPI pinctrl, CPI 400, 90-degree orientation, X and scroll-X inversion, 250 Hz polling, smart algorithm, movement threshold 5, and automouse timeout 0 |
| Sensor behavior | Trackball input listener is present; target defaults include automouse layer 6 and scroll layers 5. Do not replace this path with Cornix sensor assumptions |
| Encoders | Left EC11 A/B are XIAO D5/D0; right A/B are GPIO0_16/GPIO1_10. Both are enabled in their side overlays although common Orbee.dtsi marks the nodes disabled. Orbee.json also reports both sensor entries disabled; reconcile this metadata before later tests |
| Touch inputs | Orbee_L has three GPIO input behaviors on GPIO0_16, GPIO1_0, and GPIO1_10. Preserve them |
| LED/battery module | `zmk-rgbled-widget` is configured; thresholds are 30% high and 10% critical, and the right side enables layer colors. Avoid a second owner for its LED resources |
| Existing keymap | Eight current layers, custom layer-return macro, touch behaviors, PMW3610, and encoder scroll behavior are already present; Cornix bindings must be merged by the Phase 0 map rather than replacing this baseline |

The Orbee transform repeats raw RC(3,6) at O16/O40 and RC(3,7) at O28/O41. The right-side `col-offset=6`, encoder status-vs-JSON mismatch, and current floating `main` module refs in `config/west.yml` are explicit follow-up checks. `zmk`, `zmk-pmw3610-driver-alt`, and `zmk-rgbled-widget` are named on `main` rather than frozen SHAs, so a later build must first record exact module revisions.

## Migration classification

### portable

- Active standard layer/key behavior, mouse-layer key functions, combos, Dock mod-morphs, BT selectors, and the Mac/Win and US/JIS/ISO user-facing interaction models.

### adapt

- Per-profile persistence and symbol code; dynamic macro C/settings storage; keyboard-driven mouse while keeping PMW3610; encoder bindings to Orbee EC11 wiring; indicator, battery, charging, and VBUS behavior to Orbee LED/power ownership; sleep and BLE/split tuning after target-version comparison.

### not-applicable

- Cornix nRF board, Cornix matrix/pins, Cornix left/right electrical roles, and Cornix WS2812/SPI3/EXT_POWER wiring. Orbee hardware stays authoritative.

### needs-decision

- C0's COMBO hold route; C30/C31 and C46–C48 physical destinations; `cornix54.h` alias mismatches; repeated Orbee transform coordinates; dormant home-row/mod-morph definitions and unbuilt diagnostics; and which Cornix patches/settings are compatible with Orbee's exact dependency revisions.

## Important physical-layout finding

The frozen count is Cornix 50 positions versus Orbee 43. Row counts are 12/12/14/12 for Cornix and 10/12/12/9 for Orbee. Only the top row is reduced by one outer position per side (C0 and C11); the bottom row loses the center-adjacent C30/C31 pair, and the thumb/palm row has three fewer right-side positions. Therefore “one fewer outer column per side” does not describe the full position delta. The complete C0–C49 mapping, including the 5 unresolved positions and source metadata discrepancies, is in [CORNIX_TO_ORBEE_POSITION_MAP.md](CORNIX_TO_ORBEE_POSITION_MAP.md).

## Phase plan

### Phase 0 — Inventory and position mapping only

No firmware behavior changes.

Deliverables:

1. Cornix -> Orbee physical position map
2. complete Cornix feature inventory with source files
3. Orbee preserved-feature inventory
4. hardware-dependency classification
5. conflicts / risks / unresolved decisions
6. implementation PR breakdown

For each Cornix physical position record:

- Cornix position ID
- Cornix physical location
- current Cornix base binding
- Orbee destination position
- status: `direct`, `moved`, `omitted`, `needs-decision`
- notes

Acceptance:

- every Cornix layer/behavior/macro/custom module accounted for
- every user-facing function has a migration disposition
- no implementation changes
- no silent feature loss

### Phase 1 — Core keymap parity

Port:

- BASE / Mac BASE
- LOWER / RAISE / ADJUST
- NAVI / NUM / COMBO
- active tap/hold behaviors, mod-morphs, combos, and standard macros
- do not add the unreferenced home-row/shift hold-tap definitions without a separate decision

Constraints:

- follow Phase 0 mapping
- preserve PMW3610 and Orbee hardware definitions

### Phase 2 — Mouse / pointing integration

Port Cornix MOUSE / precision / scroll behavior while keeping Orbee's physical PMW3610.

- preserve current Orbee trackball settings unless explicitly changed
- keep Mac natural-scroll behavior
- check encoder / sensor interaction

### Phase 3 — Per-profile OS and keyboard-layout state

Port:

- BT0-BT4 Mac/Win persistence
- US/JIS/ISO persistence
- restore-on-profile-switch behavior
- current Cornix persistent-write retry behavior

### Phase 4 — Dynamic macros

Port the five persistent macro slots and their recording/playback behavior.

### Phase 5 — Battery / LED / charging behavior

Reimplement the Cornix user-facing semantics for Orbee hardware.

- manual battery preview
- charging indication
- full indication
- LED brightness/on-off persistence where Orbee hardware allows
- retain "no automatic low-battery red blink"
- avoid LED ownership conflicts with Orbee's existing RGB widget/module

This must be a separate PR because it is hardware-dependent.

### Phase 6 — Power / stability parity

Evaluate and port only the settings that are appropriate for Orbee:

- six-hour inactivity sleep
- USB/VBUS sleep suppression
- queue / stack / BLE tuning

Do not copy Cornix values blindly.

### Phase 7 — Integration / regression

Validate:

- both Orbee halves build cleanly
- binding count matches physical layout
- PMW3610 regression
- encoder regression
- BLE pairing/profile behavior
- settings persistence
- mouse layers
- battery / LED behavior
- hardware-test checklist

### PR dependencies and gates

| PR phase | Scope | Depends on | Gate before proceeding |
| --- | --- | --- | --- |
| 1 | Core layers and active key behaviors | Phase 0 position decisions and compiled-transform review | Both halves build; keymap count and mapped positions agree |
| 2 | Keyboard-driven mouse, precision, and scroll | Phase 1 mapping; Orbee PMW3610 remains enabled | Keyboard mouse and physical trackball coexist; no CPI change |
| 3 | Per-profile Mac/Win and US/JIS/ISO | Phase 1 key placements | Profile-switch restore and persistence checks |
| 4 | Five dynamic macro slots | Phase 1 key placement and storage review | Record/play/reboot/persistence and cleanup checks |
| 5 | Battery preview, LED, and charge semantics | Phase 1 controls plus RGB widget/rail ownership review | No LED ownership conflict; both sides report and charge correctly |
| 6 | Sleep, VBUS, BLE/split stability | Exact Orbee modules and power/role comparison | Separate config review; retain current Orbee split and USB roles |
| 7 | Integration | Phases 1–6 | Both halves build; static regression, then separately tracked hardware checklist |

Phases 2–6 can be reviewed as separate PRs after their listed prerequisite. No phase should bundle Cornix electrical definitions into Orbee.

## Explicit exclusions from this migration

- DYA Studio / ZMK Studio enablement
- redesigning Orbee PCB hardware
- changing PMW3610 sensitivity unless separately requested
- unrelated modernization of Orbee ZMK structure

## Codex execution model

Detailed instructions live here (and later in the parent GitHub Issue once Issues are enabled). Phase 0 inventory and candidate mapping are recorded in this PR. Implementation remains outside this completed Phase 0 task; any later Phase 1 task must first resolve the documented `needs-decision` positions and transform checks.

## Phase 0 validation record

- `git diff --check` passes.
- The position map has one entry for every C0–C49 ID, no duplicate or missing IDs, and every listed Cornix/Orbee row, column, and x/y coordinate matches its frozen JSON layout. The 43 direct/moved candidates cover O0–O42 exactly once. Status counts are 38 direct, 5 moved, 2 omitted, and 5 needs-decision.
- The commit changes only `docs/CORNIX_TO_ORBEE_MIGRATION.md` and `docs/CORNIX_TO_ORBEE_POSITION_MAP.md`. Firmware, keymap, DTS, Kconfig, manifest, patch, and Cornix source files are unchanged.
- Firmware builds and physical-device tests were outside this Phase 0 inventory/mapping task.
