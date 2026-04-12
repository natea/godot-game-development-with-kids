# Color Perception System

> **Status**: Designed
> **Author**: game-designer + systems-designer
> **Last Updated**: 2026-04-12
> **Implements Pillar**: Pillar 1 (Measure, Don't Guess), Pillar 3 (Reveal Through Repetition)

## Summary

The color perception system presents Ishihara-style plates to screen for color
vision deficiency (CVD). It uses 24 curated plates with clinically validated color
pairings, randomized dot placement, and a three-step emotional scaffolding model
to communicate results. Positioned as a screening indicator, not a clinical diagnosis.

> **Quick reference** — Layer: `Core` · Priority: `MVP` · Key deps: `Stimulus-Response Engine`

## Overview

The color perception system extends the Stimulus-Response Engine with a different
stimulus type: instead of a single white circle, the player sees a circular field
of colored dots where one cluster forms a number or shape in a contrasting hue. The
player identifies what they see (or reports they see nothing). Over 12 plates per
session (MVP), the system accumulates evidence about the player's ability to
distinguish specific color pairings — particularly red-green (protan/deutan) and
blue-yellow (tritan) confusion pairs. The system never says "you are colorblind."
It reports screening confidence and suggests professional evaluation when indicators
are present. Each additional session increases diagnostic confidence, serving
Pillar 3 (Reveal Through Repetition).

## Player Fantasy

You are discovering something real about your own biology. The test is calm and
unhurried — no time pressure, no wrong answers in the traditional sense. Each plate
is a quiet question: "What do you see?" The revelation, if it comes, is delivered
with respect and context: "You may have difficulty distinguishing red-green hues —
this is common, affecting ~8% of men." The fantasy is **self-knowledge without
judgment** — the same satisfaction as learning your blood type, not the anxiety of
a medical diagnosis.

## Detailed Design

### Core Rules

1. **Plate structure**: Each plate is a circular field (280px diameter) filled with
   dots of varying sizes (8px–24px diameter, randomized per presentation). Dots
   overlap slightly to form a dense mosaic. A subset of dots forms a **target
   pattern** (a number 1-9 or simple shape) in a **confusion color**, while
   background dots use the **paired color**.
2. **Color pairings are curated, not procedural**: The 24 plates use specific
   color pairs derived from Ishihara methodology. Colors are defined as exact
   hex values in the `--stim-*` namespace. Color pairings are NEVER procedurally
   generated — the clinical validity depends on using established confusion pairs.
3. **Dot placement IS randomized**: While color pairings are fixed, the spatial
   arrangement of dots is randomized each time a plate is presented. This prevents
   players from memorizing spatial patterns across sessions.
4. **Plate types by CVD category**:
   - 8 plates test protan/deutan (red-green) confusion
   - 2 plates test tritan (blue-yellow) confusion
   - 2 plates are **control plates** — visible to all viewers regardless of CVD
     (used to detect random answering or display calibration issues)
5. **Player response**: For each plate, the player either:
   - Taps/clicks on the number/shape they see (correct identification)
   - Selects "I don't see a pattern" (cannot identify)
   - The response is recorded as `identified_correctly`, `identified_incorrectly`,
     or `not_identified`
6. **No time pressure**: Plates remain visible until the player responds. There is
   no timeout. Color perception is not a speed test — accuracy is the only metric.
7. **Response evaluation**: Each plate has a known correct answer. The system
   records whether the player's response matches, and which CVD category the plate
   tests.
8. **Per-plate data**: `PlateResult { plate_id, cvd_category, correct_answer,
   player_answer, is_correct, response_time_ms, session_id }`
9. **Module output**: After all 12 plates, the module emits `plate_results:
   Array[PlateResult]` to the Statistical Analysis Engine.

### Plate Set Definition (MVP — 12 plates)

| Plate ID | CVD Category | Target | Confusion Pair | Control? |
|----------|-------------|--------|----------------|----------|
| P01 | Control | "12" | Gray on slightly different gray | Yes |
| P02 | Control | "8" | Orange on slightly different orange | Yes |
| P03 | Protan/Deutan | "6" | `--stim-red` on `--stim-green` | No |
| P04 | Protan/Deutan | "2" | `--stim-green` on `--stim-red` | No |
| P05 | Protan/Deutan | "5" | `--stim-red` on `--stim-green` | No |
| P06 | Protan/Deutan | "3" | `--stim-green` on `--stim-red` | No |
| P07 | Protan/Deutan | "9" | `--stim-red` on `--stim-green` | No |
| P08 | Protan/Deutan | "7" | `--stim-green` on `--stim-red` | No |
| P09 | Protan/Deutan | "4" | `--stim-red` on `--stim-green` | No |
| P10 | Protan/Deutan | "1" | `--stim-green` on `--stim-red` | No |
| P11 | Tritan | "5" | `--stim-blue` on `--stim-yellow` | No |
| P12 | Tritan | "7" | `--stim-yellow` on `--stim-blue` | No |

### States and Transitions

This system extends the Stimulus-Response Engine's state machine with these
modifications:

| State | Entry Condition | Exit Condition | Behavior |
|-------|----------------|----------------|----------|
| `PLATE_DISPLAYED` | Replaces `STIMULUS_PRESENTED` | Player responds (tap pattern or "no pattern") | Plate visible. No timer. Player studies and responds at their own pace. |
| `RESPONSE_EVALUATED` | Replaces `RESPONSE_CAPTURED` | Evaluation complete | Correctness computed. PlateResult packaged. Brief feedback (correct/incorrect — optional, see Edge Cases). |
| `TIMEOUT` | — | — | **Not inherited.** Disabled for color perception. Plates have no time limit — color discrimination is not a speed test. |

All other states (IDLE, WAITING, TRIAL_COMPLETE, MODULE_COMPLETE) are inherited
from the Stimulus-Response Engine. The wait phase between plates uses a fixed
`inter_plate_delay_ms` (1200ms) rather than randomized delay (no anticipation
prevention needed — this is not a speed test).

### Interactions with Other Systems

| System | Direction | Interface |
|--------|-----------|-----------|
| **Stimulus-Response Engine** | Upstream (extends) | Inherits state machine, input capture, and module lifecycle. Replaces stimulus type and response evaluation. |
| **Statistical Analysis Engine** | Downstream | Provides `plate_results: Array[PlateResult]` for CVD screening analysis. Stats engine computes CVD indicators separately from RT metrics. |
| **Results & Interpretation** | Downstream (via Stats) | CVD screening results flow through Stats to the interpretive layer for emotional scaffolding. |
| **Feedback System** | Downstream | Emits `plate_responded(plate_result)` for minimal feedback. Feedback is deliberately muted — no celebration/punishment for color perception responses. |

### CVD Screening Logic (in Statistical Analysis Engine)

The color perception system produces raw plate results. The Statistical Analysis
Engine computes:

- **Protan/deutan miss rate**: incorrect or unidentified responses on red-green
  plates / total red-green plates
- **Tritan miss rate**: same for blue-yellow plates
- **CVD screening indicator**: if protan/deutan miss rate ≥ `cvd_threshold` (50%),
  flag as "possible red-green CVD." If tritan miss rate ≥ 50%, flag as "possible
  blue-yellow CVD."
- **Screening confidence**: increases with more sessions (more plates = more
  evidence). Computed per CVD category.

### Three-Step Emotional Scaffolding

1. **Before the color module** (pre-framing card): "This test measures color
   discrimination. There is no pass or fail. Some people see the world in
   different colors — that's biology, not a deficiency."
2. **During results** (if CVD indicators present): "You may have difficulty
   distinguishing red-green hues. This is common — it affects approximately
   8% of men and 0.5% of women worldwide."
3. **Resource link** (if CVD indicators present): "For a professional assessment,
   consult an optometrist or ophthalmologist. Learn more: [NHS Color Vision
   Deficiency] | [American Academy of Ophthalmology]"

If no CVD indicators: "Your color discrimination appears typical for the tested
color pairs. Confidence: [X]% — additional sessions will increase accuracy."

## Formulas

### CVD Miss Rate

The cvd_miss_rate formula is defined as:

`cvd_miss_rate = (incorrect_count + unidentified_count) / total_plates_in_category`

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| incorrect_count | I | int | 0–8 | Plates where player identified wrong pattern |
| unidentified_count | U | int | 0–8 | Plates where player said "no pattern" |
| total_plates_in_category | N | int | 2–8 | Total plates for this CVD category |
| cvd_miss_rate | MR | float | 0.0–1.0 | Proportion of missed plates |

**Output Range:** 0.0 (perfect) to 1.0 (all missed).

**Example:** Player misses 5 of 8 red-green plates. Miss rate = 5/8 = 0.625 (62.5%).

### CVD Screening Confidence

`cvd_confidence = min(100, (cumulative_plates_tested / cvd_confidence_ceiling) * 100)`

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| cumulative_plates_tested | P | int | 0–∞ | Total plates of this CVD category across all sessions |
| cvd_confidence_ceiling | C | int | 24–72 | Plates needed for maximum confidence |
| cvd_confidence | conf | float | 0–100 | Screening confidence percentage |

**Output Range:** 0% to 100%. At default ceiling of 48, reaches 50% after 24 plates
(3 sessions of 8 red-green plates each).

**Example:** 3 sessions × 8 protan/deutan plates = 24 total. Confidence = 24/48 × 100 = 50%.

### Dot Placement Randomization

Dots are placed using Poisson disk sampling within the plate circle:
- Plate radius: 140px
- Minimum dot spacing: 4px (allows slight overlap)
- Dot radius: random uniform between 4px and 12px
- Target pattern dots: same placement algorithm but constrained to the pattern mask
- Expected dot count: 180–220 per plate (varies with randomized sizes)

No formula needed for the player — this is implementation detail for visual fidelity.

## Edge Cases

| Scenario | Expected Behavior | Rationale |
|----------|------------------|-----------|
| Player fails both control plates | Show calibration warning: "Your display may not be rendering colors accurately. Results may be unreliable." Continue testing but flag session as `calibration_suspect=true`. | Control plates are visible to everyone. Failure suggests display issue, not CVD. |
| Player passes all plates perfectly (0 misses) | Report "No indicators of color vision deficiency detected" with current confidence level. Do not say "you are not colorblind." | Screening can have false negatives, especially for mild CVD. Precise language matters. Pillar 1. |
| Player misses exactly 1 red-green plate out of 8 | Miss rate = 12.5%, below threshold. Report "No strong indicators" but note "1 uncertain response on red-green plates — additional sessions will clarify." | Single-plate misses can be attention errors, not CVD. Don't over-interpret sparse data. |
| Player's CVD indicators are inconsistent across sessions | Report both the latest session results and the cumulative trend. "Your latest session showed 3/8 red-green misses, but your cumulative rate across 5 sessions is 15%." | Pillar 3: trends matter more than single sessions. Inconsistency itself is data. |
| Player navigates away mid-module | Completed plates are preserved. Incomplete session does not contribute to CVD confidence until resumed or a new full session is completed. | Partial data is unreliable for screening. Don't update CVD indicators from incomplete modules. |
| Display cannot render stim colors accurately (TN panel, wrong color profile) | No automatic detection possible. Pre-test disclaimer: "Color accuracy depends on your display. For best results, use a display with accurate color reproduction." | Browser cannot detect display calibration. Honest disclaimer is the only mitigation. |
| Player intentionally answers randomly | Control plates will likely be failed (they're easy for everyone). `calibration_suspect` flag triggers. Results noted as unreliable. | Control plates serve as a validity check against random or malicious answering. |

## Dependencies

| System | Direction | Nature of Dependency |
|--------|-----------|---------------------|
| Stimulus-Response Engine | This depends on SRE | Inherits timing, state machine, input capture, and module lifecycle. Hard dependency — cannot function without it. |
| Statistical Analysis Engine | Depended on by Stats | Provides `plate_results` for CVD screening computation. Stats engine cannot compute CVD indicators without this data. |
| Results & Interpretation | Depended on by R&I (via Stats) | CVD screening results flow through Stats to the interpretive layer. |
| Feedback System | Depended on by Feedback | Emits `plate_responded` for minimal feedback on each plate response. |

**Upstream dependencies:** Stimulus-Response Engine (hard).

## Tuning Knobs

| Parameter | Current Value | Safe Range | Effect of Increase | Effect of Decrease |
|-----------|--------------|------------|-------------------|-------------------|
| `plates_per_session` | 12 | 8–24 | More data per session; higher confidence faster, longer sessions | Less data; faster sessions, slower confidence accumulation |
| `cvd_threshold` | 0.50 | 0.30–0.75 | Fewer CVD flags; fewer false positives, more false negatives | More CVD flags; more false positives, fewer false negatives |
| `cvd_confidence_ceiling` | 48 | 24–72 | Harder to reach max confidence; more conservative | Easier to reach max confidence; may report high confidence prematurely |
| `inter_plate_delay_ms` | 1200 | 600–2000 | Longer pause between plates; more recovery time | Shorter pause; faster pacing |
| `plate_diameter_px` | 280 | 200–400 | Larger plates; easier to see fine detail, more screen space used | Smaller plates; harder to see patterns, especially on small displays |
| `dot_min_radius_px` | 4 | 3–8 | Larger minimum dots; easier to see, less detail | Smaller dots; more detail, harder on low-res displays |
| `dot_max_radius_px` | 12 | 8–16 | Larger maximum dots; more visual variety | Smaller max; more uniform appearance |

**Interaction warnings:**
- `cvd_threshold` directly controls sensitivity/specificity tradeoff. Lower = more sensitive (catches more CVD) but more false positives. Must be set carefully.
- `plates_per_session` × sessions needed for `cvd_confidence_ceiling` determines how many sessions before the game can make a confident screening statement.

## Visual/Audio Requirements

| Event | Visual Feedback | Audio Feedback | Priority |
|-------|----------------|---------------|----------|
| Plate presentation | Plate appears center-screen on dark canvas (`--canvas`). No animation — instant display. Surrounding UI chrome bleached to near-zero saturation. | None — clinical silence during color test. | Critical |
| Pre-framing card | Neutral text card before first plate. `--signal-white` text on `--canvas`. "This test measures color discrimination. There is no pass or fail." | None. | High |
| Player responds (correct) | Plate fades slightly (to 80% opacity) over 200ms. No color-coded feedback (green = correct would contaminate the test). Brief `--signal-white` checkmark icon, 16px, top-right. | Soft neutral click (same as module start click). | Medium |
| Player responds (incorrect/unidentified) | Same fade behavior. No negative visual indicator. No red, no X. | Same neutral click. No differentiation from correct. | Medium |
| Module complete | Transition to results with emotional scaffolding. | Completion chime (same as RT module). | Medium |
| CVD indicator present in results | Results text uses `--signal-white` only. No red or green in the CVD result display (they are the colors being tested). Arrow icons for direction, not color coding. | None — results are read in silence. | Critical |

**Critical visual rule**: During the color perception module, the stimulus area
MUST be isolated from any UI chrome that uses color. No `--precision-cyan` or
`--alert-amber` elements visible during plate presentation. The dark canvas is
the only backdrop.

## UI Requirements

| Information | Display Location | Update Frequency | Condition |
|-------------|-----------------|-----------------|-----------|
| Plate counter ("3/12") | Top-right, `--neutral-text`, 16px | Per plate completion | During active module |
| Pre-framing card text | Center screen, `--signal-white`, 16px Inter | Once before module | Before first plate |
| Response buttons | Below plate: "I see: [number pad 1-9]" and "I don't see a pattern" | Per plate | During PLATE_DISPLAYED state |
| Progress bar | Top edge, 1px, `--neutral-mid` | Per plate | During active module |

## Cross-References

| This Document References | Target GDD | Specific Element Referenced | Nature |
|--------------------------|-----------|----------------------------|--------|
| Inherits state machine and input capture | `design/gdd/stimulus-response-engine.md` | State machine, `_input()` handling, module lifecycle | Rule dependency |
| Plate results feed CVD analysis | `design/gdd/statistical-analysis-engine.md` | CVD miss rate computation, confidence calculation | Data dependency |
| CVD results displayed with emotional scaffolding | `design/gdd/results-interpretation.md` | Three-step scaffolding presentation | State trigger |
| Minimal feedback on plate response | `design/gdd/feedback-system.md` | `plate_responded` signal, muted feedback | State trigger |

## Acceptance Criteria

- [ ] GIVEN a plate is displayed, WHEN the player identifies the correct number, THEN `is_correct=true` is recorded in the PlateResult
- [ ] GIVEN a plate is displayed, WHEN the player selects "I don't see a pattern," THEN `not_identified` is recorded
- [ ] GIVEN the same plate presented twice across sessions, WHEN dot placement is compared, THEN spatial arrangement differs (randomized) while color pairings remain identical
- [ ] GIVEN 12 plates in a session, WHEN the module completes, THEN `plate_results` array contains exactly 12 PlateResult objects
- [ ] GIVEN a player misses 5/8 red-green plates, WHEN CVD screening runs, THEN "possible red-green CVD" is flagged
- [ ] GIVEN a player fails both control plates, WHEN results are shown, THEN calibration warning is displayed and session flagged as suspect
- [ ] GIVEN CVD indicators are present, WHEN results are displayed, THEN the three-step emotional scaffolding is applied (pre-framing → contextual result → resource link)
- [ ] GIVEN no CVD indicators, WHEN results are displayed, THEN language says "no indicators detected" (not "you are not colorblind")
- [ ] GIVEN a plate is displayed, WHEN no time limit is enforced, THEN the plate remains visible indefinitely until the player responds
- [ ] GIVEN the color test module is active, WHEN UI is rendered, THEN no chromatic UI elements (cyan, amber) are visible in the viewport
- [ ] No hardcoded color values — all plate colors reference `--stim-*` tokens from configuration

## Open Questions

| Question | Owner | Deadline | Resolution |
|----------|-------|----------|-----------|
| Should response feedback differentiate correct from incorrect during the test? Current design: no differentiation (both get neutral click). | Game Designer | Sprint 1 | Revealing correctness mid-test might bias subsequent responses |
| What specific hex values should `--stim-red`, `--stim-green`, `--stim-blue`, `--stim-yellow` be? Requires color science expertise. | Technical Artist | Before implementation | Must match published Ishihara confusion pairs |
| Should plates be presented in fixed order or randomized order within a session? | Game Designer | Sprint 1 | Fixed order enables progressive difficulty; random prevents order effects |
| What is the false negative rate for mild CVD on typical consumer displays? | Systems Designer | Before Alpha | May need to adjust `cvd_threshold` or add display-quality disclaimer |
