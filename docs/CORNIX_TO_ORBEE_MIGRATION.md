# Cornix -> Orbee feature parity migration

Status: planning / Phase 0 inventory

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
- positional home-row mods
- shift hold-taps
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

## Migration classification

### A. Hardware-independent: port first

- keymap layers
- hold-tap / mod-morph / combos / standard macros
- Windows / Mac interaction model
- per-profile Mac/Win persistence
- per-profile US/JIS/ISO persistence
- dynamic macro slots
- sleep policy where supported

### B. Adapt to Orbee hardware

- battery preview
- charging/full indication
- LED brightness/on-off persistence
- VBUS-dependent behavior
- encoder behavior

### C. Preserve as Orbee-specific

- PMW3610
- Orbee matrix / transforms / pins
- Orbee split definition
- Orbee physical sensor wiring

## Important physical-layout finding

A preliminary comparison shows that this is **not safe to treat as a simple row-for-row truncation**.

Cornix and Orbee differ in more than just total binding count. The final mapping must be generated from all three sources:

- Cornix physical layout metadata / position macros
- Orbee `Orbee.json`
- Orbee matrix transform in `Orbee.dtsi`

The user's design intent remains: Orbee is effectively the Cornix concept with one fewer outer column per side. Phase 0 must prove exactly which logical positions that corresponds to and document every non-direct mapping.

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
- home-row mods
- hold-taps / mod-morphs / standard macros

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

## Explicit exclusions from this migration

- DYA Studio / ZMK Studio enablement
- redesigning Orbee PCB hardware
- changing PMW3610 sensitivity unless separately requested
- unrelated modernization of Orbee ZMK structure

## Codex execution model

Detailed instructions live here (and later in the parent GitHub Issue once Issues are enabled).

Codex tasks should be launched one Phase at a time.

The first Codex task is **Phase 0 only**. It must not modify firmware behavior. It should investigate both repositories, produce the mapping/inventory, and report findings before implementation begins.
