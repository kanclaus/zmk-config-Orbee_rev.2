# Phase 0: Cornix → Orbee position map

Status: source audit complete; candidate mapping only. No firmware or keymap changes were made.

## Reference snapshots and indexing

- Cornix config: kanclaus/Cornix_ZMK main at 9fd24c56c8d00daed65efe78748bb0f47e218e44.
- Cornix board module: zmk-keyboard-cornix at 3318e486328fd2024233b73b1b8ba8887006c112, pinned by that Cornix config manifest.
- Orbee target: kanclaus/zmk-config-Orbee_rev.2 main at f627f76a11943c17ff6d11dcf801d66889593934.
- C0–C49 are the zero-based positions in Cornix layout_50 and in the active default_layer/mac_layer binding order. O0–O42 are the zero-based positions in Orbee.json and the Orbee keymap order.
- Cornix location is written as source JSON row/column plus physical x/y. Orbee location is target JSON row/column plus x/y. Cornix JSON rows start at 0; Orbee JSON rows start at 1.
- The short Cornix aliases are shown only where the actual config/includes/cornix54.h defines a usable, non-conflicting position name. The discrepancies are detailed below.

Status meanings:

- direct: same hand and corresponding row-local slot exists in Orbee; this is a physical candidate, not a claim that the Orbee binding already has Cornix behavior.
- moved: the source slot is absent and the retained key shifts to another numbered/physical slot in the same row.
- omitted: the source physical switch has no target position. This does not by itself mean that its key function is absent elsewhere.
- needs-decision: there is no unique target slot, candidate mappings collide, or the source position metadata is inconsistent.

## Position-count proof

Cornix has 50 positions and Orbee has 43. The active layout row counts are:

| Row, from top | Cornix | Orbee | Difference |
| --- | ---: | ---: | ---: |
| Top | 12 | 10 | -2 |
| Home | 12 | 12 | 0 |
| Bottom | 14 | 12 | -2 |
| Thumb / palm | 12 | 9 | -3 |
| Total | 50 | 43 | -7 |

The phrase “one fewer outer column per side” describes the top row only: source C0 and C11 are the two outermost top positions without target slots. It does not explain the complete matrix delta. The bottom row loses the two center-adjacent positions C30/C31, while the thumb/palm row loses three right-side positions. The home row keeps all 12 positions.

## Transform and keymap cross-check

Cornix layout_50 in its pinned board module contains 50 position-map entries. Its transform order is C0–5 = RC(0,0..5), C6–11 = RC(0,12..7), C12–17 = RC(1,0..5), C18–23 = RC(1,12..7), C24–30 = RC(2,0..6), C31 = RC(1,13), C32–37 = RC(2,12..7), C38–43 = RC(3,0..5), and C44–49 = RC(3,12..7).

Orbee.json, Orbee.keymap, and the 43 entries in Orbee.dtsi agree on the target position count and binding order. The Orbee transform contains repeated raw coordinates: O16 and O40 both use RC(3,6), and O28 and O41 both use RC(3,7). Orbee_R.overlay also applies col-offset=6 to the shared transform. Phase 0 did not generate or inspect compiled devicetrees, so this is an explicit transform-resolution gate before keymap edits; this document does not conclude whether the repetitions are valid for the final left/right builds.

## Complete Cornix → Orbee candidate map

| Cornix | Cornix physical location | BASE | MAC | Orbee destination | Status | Reason / note |
| ---: | --- | --- | --- | --- | --- | --- |
| C0 | r0/c0 (0,0.5), LT5 | `&lt COMBO ESC` | `&lt COMBO ESC` | — | omitted | Left outermost top switch has no target position. ESC also exists at C46 and Orbee O28; this is a position omission, not proof that ESC is lost. C0 is the frozen entry to COMBO, so layer access must be reassigned or explicitly excluded. |
| C1 | r0/c1 (1,0.5), LT4 | Q | Q | O0 (r1/c0, 0,1) | moved | After removing C0, the retained left-top row shifts one slot toward the edge in the target index/layout. |
| C2 | r0/c2 (2,0.25), LT3 | W | W | O1 (r1/c1, 1,1) | moved | Same left-top row; one-slot shift after C0 is removed. |
| C3 | r0/c3 (3,0), LT2 | E | E | O2 (r1/c2, 2,1) | moved | Same left-top row; one-slot shift after C0 is removed. |
| C4 | r0/c4 (4,0.25), LT1 | R | R | O3 (r1/c3, 3,1) | moved | Same left-top row; one-slot shift after C0 is removed. |
| C5 | r0/c5 (5,0.5), LT0 | T | T | O4 (r1/c4, 4,1) | moved | Same left-top row; one-slot shift after C0 is removed. |
| C6 | r0/c8 (8.5,0.5), RT0 | Y | Y | O5 (r1/c8, 8,1) | direct | Right-top row retains five corresponding positions. |
| C7 | r0/c9 (9.5,0.3), RT1 | U | U | O6 (r1/c9, 9,1) | direct | Right-top row retains five corresponding positions. |
| C8 | r0/c10 (10.5,0), RT2 | I | I | O7 (r1/c10, 10,1) | direct | Right-top row retains five corresponding positions. |
| C9 | r0/c11 (11.5,0.33), RT3 | O | O | O8 (r1/c11, 11,1) | direct | Right-top row retains five corresponding positions. |
| C10 | r0/c12 (12.5,0.5), RT4 | P | P | O9 (r1/c12, 12,1) | direct | Right-top row retains five corresponding positions. |
| C11 | r0/c13 (13.5,0.5), RT5 | DELETE | DELETE | — | omitted | Right outermost top switch has no target position. DELETE remains on Orbee O42 as the tap side of its current RCTRL/DELETE mod-tap. |
| C12 | r1/c0 (0,1.5), LM5 | TAB | TAB | O10 (r2/c0, 0,2) | direct | Left home row retains six corresponding positions. |
| C13 | r1/c1 (1,1.5), LM4 | A | A | O11 (r2/c1, 1,2) | direct | Left home row retains six corresponding positions. |
| C14 | r1/c2 (2,1.25), LM3 | S | S | O12 (r2/c2, 2,2) | direct | Left home row retains six corresponding positions. |
| C15 | r1/c3 (3,1), LM2 | D | D | O13 (r2/c3, 3,2) | direct | Left home row retains six corresponding positions. |
| C16 | r1/c4 (4,1.25), LM1 | F | F | O14 (r2/c4, 4,2) | direct | Left home row retains six corresponding positions. |
| C17 | r1/c5 (5,1.5), LM0 | `&lt NAVI G` | `&lt MAC_WINR G` | O15 (r2/c5, 5,2) | direct | Left home row retains six corresponding positions; tap/hold behavior differs by OS layer. |
| C18 | r1/c8 (8.5,1.5), RM0 | `&lt NAVI H` | `&lt MAC_WINL H` | O16 (r2/c7, 7,2) | direct | Right home row retains six positions by hand-local order. The target right side starts one JSON column inward; see transform gate above. |
| C19 | r1/c9 (9.5,1.3), RM1 | J | J | O17 (r2/c8, 8,2) | direct | Right home row retains six positions by hand-local order. |
| C20 | r1/c10 (10.5,1), RM2 | K | K | O18 (r2/c9, 9,2) | direct | Right home row retains six positions by hand-local order. |
| C21 | r1/c11 (11.5,1.33), RM3 | L | L | O19 (r2/c10, 10,2) | direct | Right home row retains six positions by hand-local order. |
| C22 | r1/c12 (12.5,1.5), RM4 | `CORNIX_SYMBOL(CORNIX_SYM_MINUS, MINUS)` | `CORNIX_SYMBOL(CORNIX_SYM_MINUS, MINUS)` | O20 (r2/c11, 11,2) | direct | Right home row retains six positions by hand-local order. |
| C23 | r1/c13 (13.5,1.5), RM5 | EXCLAMATION | EXCLAMATION | O21 (r2/c12, 12,2) | direct | Right home row retains six positions by hand-local order. |
| C24 | r2/c0 (0,2.55), LB5 | LSHFT | LSHFT | O22 (r3/c0, 0,3) | direct | Left bottom finger row retains six corresponding positions. |
| C25 | r2/c1 (1,2.55), LB4 | Z | Z | O23 (r3/c1, 1,3) | direct | Left bottom finger row retains six corresponding positions. |
| C26 | r2/c2 (2,2.25), LB3 | X | X | O24 (r3/c2, 2,3) | direct | Left bottom finger row retains six corresponding positions. |
| C27 | r2/c3 (3,2), LB2 | `&lt_c_alt NUM C` | `&lt_c_cmd NUM C` | O25 (r3/c3, 3,3) | direct | Left bottom finger row retains six corresponding positions; hold action changes Alt/Command by OS. |
| C28 | r2/c4 (4,2.25), LB1 | `&lt_g MOUSE V` | `&lt_g MAC_MOUSE V` | O26 (r3/c4, 4,3) | direct | Left bottom finger row retains six corresponding positions; hold enters OS-specific mouse layer. |
| C29 | r2/c5 (5,2.55), LB0 | B | B | O27 (r3/c5, 5,3) | direct | Left bottom finger row retains six corresponding positions. |
| C30 | r2/c6 (6,2), LH3 | C_MUTE | C_MUTE | — (candidate inner slots conflict with C17/C18 row map) | needs-decision | Extra center-adjacent left position has no dedicated Orbee bottom-row slot. Its consumer mute function is not present in the current Orbee default binding set. |
| C31 | r2/c7 (7.5,2), RH3 | LG(LA(K)) | LG(LA(K)) | — (geometry near O16; row mapping also suggests O28) | needs-decision | Extra center-adjacent right position has no spare target slot. O16 is already the C18 right-home candidate; O28 is already the C32 right-bottom candidate. |
| C32 | r2/c8 (8.5,2.55), RB0 | N | N | O28 (r3/c7, 7,3) | direct | Right bottom finger row retains six corresponding positions; the target right side starts one JSON column inward. |
| C33 | r2/c9 (9.5,2.3), RB1 | M | M | O29 (r3/c8, 8,3) | direct | Right bottom finger row retains six corresponding positions. |
| C34 | r2/c10 (10.5,2), RB2 | COMMA | COMMA | O30 (r3/c9, 9,3) | direct | Right bottom finger row retains six corresponding positions. |
| C35 | r2/c11 (11.5,2.33), RB3 | PERIOD | PERIOD | O31 (r3/c10, 10,3) | direct | Right bottom finger row retains six corresponding positions. |
| C36 | r2/c12 (12.5,2.55), RB4 | `&lt_g MOUSE QUESTION` | `&lt_g MAC_MOUSE QUESTION` | O32 (r3/c11, 11,3) | direct | Right bottom finger row retains six corresponding positions; hold enters OS-specific mouse layer. |
| C37 | r2/c13 (13.5,2.55), RB5 | RSHFT | RSHFT | O33 (r3/c12, 12,3) | direct | Right bottom finger row retains six corresponding positions. Orbee O33 currently has RIGHT_SHIFT on hold and QUESTION on tap. |
| C38 | r3/c0 (0,3.55), no usable alias | LCTRL | LGUI | O34 (r4/c0, 0,4) | direct | Left thumb/palm row has six target slots; map by JSON row order. The position-name header does not define C38. |
| C39 | r3/c1 (1,3.55), no usable alias | LGUI | LALT | O35 (r4/c1, 1,4) | direct | Left thumb/palm row has six target slots; map by JSON row order. The position-name header does not define C39. |
| C40 | r3/c2 (2,3.3), LP0 | LALT | LCTRL | O36 (r4/c2, 2,4) | direct | Left thumb/palm row has six target slots; map by JSON row order. |
| C41 | r3/c3 (3.5,3.63), LH2/LP1 conflict | `&lt LOWER LANGUAGE_1` | `&lt MAC_LOWER LANGUAGE_1` | O37 (r4/c3, 3,4) | direct | Position index is clear in JSON/order, but cornix54.h assigns both LH2 and LP1 to 41. |
| C42 | r3/c4 (4.5,3.7), LH1/LP2 conflict | `&lt RAISE SPACE` | `&lt MAC_RAISE SPACE` | O38 (r4/c4, 4,4) | direct | Position index is clear in JSON/order, but cornix54.h assigns both LH1 and LP2 to 42. |
| C43 | r3/c5 (5.5,3.65), LH0 | `&lt ADJUST LANGUAGE_2` | `&lt MAC_ADJUST LANGUAGE_2` | O39 (r4/c5, 5,4) | direct | Left thumb/palm row has six target slots; map by JSON row order. |
| C44 | r3/c8 (7.64,5.38), RH0 | BSPC | BSPC | O40 (r4/c7, 7,4) | direct | First surviving right thumb slot; Orbee O40 is BACKSPACE. |
| C45 | r3/c9 (9,3.75), RH1 | ENTER | ENTER | O41 (r4/c8, 8,4) | direct | Second surviving right thumb slot; Orbee O41 is ENTER. |
| C46 | r3/c10 (10,3.63), RH2 | ESC | ESC | — (O28 currently has ESC but conflicts with C32 row map) | needs-decision | Three fewer right-thumb/palm positions exist in Orbee. Moving ESC to O28 would conflict with the physical row candidate for C32. |
| C47 | r3/c11 (11.5,3.38), no usable alias | LC(Z) | LG(Z) | — (O42 is already the C49/RCTRL candidate) | needs-decision | Undo mapping needs a selected target or an explicit omission; no separate right-thumb destination remains. |
| C48 | r3/c12 (12.5,3.55), no usable alias | LC(Y) | LG(LS(Z)) | — (O42 is already the C49/RCTRL candidate) | needs-decision | Redo mapping needs a selected target or an explicit omission; no separate right-thumb destination remains. |
| C49 | r3/c13 (13.5,3.55), RP0 | RCTRL | LGUI | O42 (r4/c12, 12,4) | direct | Outermost surviving right thumb/palm slot. Orbee O42 holds RCTRL and taps DELETE; DELETE also corresponds to the omitted C11 physical position. |

Candidate totals: 38 direct, 5 moved, 2 omitted, and 5 needs-decision. Every C0–C49 source position is classified exactly once.

## Position-macro discrepancies and unresolved gates

- config/includes/cornix54.h labels its diagram “52 KEY MATRIX / LAYOUT MAPPING”, while the selected physical layout and active keymap have 50 positions. RP1=50 and RP2=51 are outside the active layout.
- The same header assigns LP1=41 while LH2=41, and LP2=42 while LH1=42. C38, C39, C47, and C48 have no usable macro definitions even though the diagram labels palm-row slots. The physical position IDs in this table therefore come from the frozen JSON, position_map, and binding order, not from guessed aliases.
- Resolve the C0 COMBO hold route, C30/C31, the right-thumb C46–C48, and the named-position header before Phase 1 chooses final key placements. For omitted positions, record whether the associated function is moved, combined with another key, or intentionally dropped.
- Inspect the compiled Orbee left and right devicetrees before any keymap edit to settle the repeated RC tuples and right-half col-offset behavior. Preserve the 43-position transform and all Orbee matrix/pin/split definitions during that review.
