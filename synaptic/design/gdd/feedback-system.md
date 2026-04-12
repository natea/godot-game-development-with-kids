# Feedback System

> **Status**: Designed
> **Author**: game-designer + sound-designer
> **Last Updated**: 2026-04-12
> **Implements Pillar**: Pillar 2 (Every Tap Feels Crisp)

## Summary

The feedback system provides immediate audio and visual responses to every player
input during tests. It fires on the same frame as input capture, using spatial
animations (scale, ring expansion) rather than temporal ones (rollups, fades) to
preserve time-perception accuracy. Feedback intensity scales subtly with
performance without distorting measurement.

> **Quick reference** — Layer: `Core` · Priority: `MVP` · Key deps: `Stimulus-Response Engine`

## Overview

Every time the player taps, clicks, or presses a key during a test, the feedback
system fires visual and audio confirmation within the same render frame. The system
has one job: make the player feel that the instrument registered their input with
precision. Feedback is spatial (a ring expanding outward, the stimulus scaling
briefly) not temporal (no counting animations, no delayed reveals). The RT number
appears instantly — not because animation is bad, but because any delay between
input and displayed result would contaminate the player's time perception. The
system also provides contextual feedback for edge cases (early responses, timeouts)
with a deliberately different feel that communicates "that didn't count" without
punishment.

## Player Fantasy

The tap connects. The moment your finger hits the key, the screen responds —
a crisp pop, a ring of cyan light expanding from the impact point, and the number
snaps into place: 231ms. The instrument heard you. The feedback is the game's
handshake: proof that your input was captured at the exact moment it occurred. The
fantasy is **instantaneous acknowledgment** — the feeling that zero time passed
between your action and the system's response.

## Detailed Design

### Core Rules

1. **Same-frame guarantee**: All primary feedback (visual acknowledgment + RT
   number display) MUST appear in the same render frame as the input event. This
   is a hard constraint, not a target.
2. **Feedback is spatial, not temporal**: Animations expand in space (ring
   expansion, scale pulse), not in time (no number rollup, no bar fill, no
   delayed reveal). The RT number appears at its final value instantly.
3. **Three feedback tiers**:
   - **Valid response**: Full feedback (pop sound + scale pulse + ring + RT number)
   - **Early response**: Warning feedback (low tone + amber border flash + "Too early" text)
   - **Timeout**: Absence feedback (stimulus fades + "No response" text + no sound)
4. **Feedback does not affect measurement**: All feedback rendering occurs AFTER
   the RT calculation is complete. Feedback is a consequence of measurement, never
   a component of it.
5. **Performance-scaled intensity** (subtle): The cyan ring expansion speed varies
   slightly with RT — faster responses produce a slightly faster ring. This is
   subliminal reinforcement, not a conscious game mechanic. The scaling is purely
   visual and has no measurement impact.
6. **Color perception module override**: During the color test, feedback is
   deliberately muted. No color-coded feedback. Sound is a neutral click only.
   No ring animation (would introduce chromatic elements into the test field).

### Feedback Events

| Event | Visual | Audio | Duration | Trigger |
|-------|--------|-------|----------|---------|
| **Valid response (RT)** | Stimulus scales 1.0→0.88→1.0 (80ms spring). 1px `--precision-cyan` ring expands from input origin (120ms, fades). RT number appears in 48px JetBrains Mono at final value. | Crisp percussive pop, 60ms, -6dBFS. | 120ms total visual, 60ms audio | `response_recorded` signal |
| **Early response** | 2px `--alert-amber` border on viewport edges, 200ms. "Too early" in 24px Inter, `--signal-white`, center-screen, 500ms. | Low sine tone, 200ms, -12dBFS. | 500ms total | `early_response` signal |
| **Timeout** | Stimulus fades to 20% opacity over 300ms. "No response" in 20px Inter, `--neutral-text`, center-screen, 800ms. | Silence. | 800ms total | `timeout` signal |
| **Module start** | "Ready" text center-screen (1s), then blank screen. | Single soft click, 40ms, -12dBFS. | 1000ms | `module_started` signal |
| **Module complete** | "Complete" text fades in, center-screen. | Two ascending tones, 80ms each, 150ms total, -6dBFS. | 500ms fade | `module_completed` signal |
| **Plate response (color test)** | Plate fades to 80% opacity over 200ms. Small `--signal-white` checkmark, 16px, top-right. No color in feedback. | Neutral click, 40ms, -12dBFS. Same for correct and incorrect. | 200ms | `plate_responded` signal |

### States and Transitions

The feedback system is stateless — it is event-driven. Each feedback event is
self-contained: it starts, plays to completion, and cleans up. Multiple feedback
events do not overlap in normal operation (the trial state machine prevents this).

If a feedback animation is interrupted (rare edge case — player navigates away
mid-animation), all active tweens and audio are killed immediately. No lingering
visual artifacts.

### Interactions with Other Systems

| System | Direction | Interface |
|--------|-----------|-----------|
| **Stimulus-Response Engine** | Upstream (listens) | Consumes signals: `response_recorded(rt_ms, trial_index, is_outlier)`, `early_response()`, `timeout()`, `module_started()`, `module_completed()`. |
| **Color Perception System** | Upstream (listens) | Consumes signal: `plate_responded(plate_result)`. Triggers muted feedback variant. |
| **Results & Interpretation** | None | No direct interaction. Feedback handles per-trial moments; Results handles post-module summaries. |

## Formulas

### Ring Expansion Speed (Performance-Scaled)

`ring_speed_px_per_ms = base_ring_speed + (speed_bonus * (1 - rt_normalized))`

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| base_ring_speed | S_b | float | 1.0–2.0 | Minimum ring expansion speed (px/ms) |
| speed_bonus | S_x | float | 0.0–1.0 | Additional speed for fast responses |
| rt_normalized | RT_n | float | 0.0–1.0 | `(rt_ms - rt_floor) / (rt_ceiling - rt_floor)`, clamped 0–1 |
| rt_floor | — | int | 100 | Fastest expected RT (ms) |
| rt_ceiling | — | int | 400 | Slowest "normal" RT (ms) |
| ring_speed_px_per_ms | S | float | 1.0–3.0 | Final ring expansion speed |

**Output Range:** 1.0 to 3.0 px/ms. Fast responses (150ms) get ~2.5 px/ms.
Slow responses (350ms) get ~1.2 px/ms. The difference is subtle — approximately
40% faster ring for a very fast response vs. a slow one.

**Example:** RT = 200ms. rt_normalized = (200-100)/(400-100) = 0.333.
ring_speed = 1.5 + (0.8 × (1 - 0.333)) = 1.5 + 0.533 = 2.03 px/ms.

### Scale Pulse Curve

The stimulus scale during the valid-response pulse follows a spring curve:

`scale(t) = 1.0 - amplitude * sin(π * t / duration) * exp(-decay * t)`

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| amplitude | A | float | 0.10–0.15 | Maximum scale change (1.0 - 0.88 = 0.12) |
| duration | D | int | 60–100 | Pulse duration in ms |
| decay | k | float | 2.0–4.0 | How quickly the pulse settles |

No overshoot: the curve touches 0.88 once and returns to 1.0 monotonically.

## Edge Cases

| Scenario | Expected Behavior | Rationale |
|----------|------------------|-----------|
| Two feedback events triggered in rapid succession | Second event replaces the first. Previous animation killed, new one starts. No overlapping animations. | State machine prevents this in normal operation, but defensive cleanup avoids visual glitches. |
| Audio system unavailable (browser restriction, muted tab) | Visual feedback plays normally. Audio failure is silent — no error messages, no degraded visual feedback. | Visual feedback is the primary channel. Audio is enhancement, not requirement. |
| Very fast RT (< 150ms) | Ring speed at maximum. No additional emphasis — don't celebrate speed, just acknowledge it. | Pillar 1: the game measures, it doesn't judge. Fast isn't "better" — it's data. |
| Very slow RT (> 400ms, but before timeout) | Ring speed at minimum. RT number still appears instantly. No negative feedback. | Same principle — slow isn't "worse." Neutral presentation for all valid responses. |
| Player's display runs below 60fps | Feedback timing is frame-based — pulse may appear slightly different at 30fps. The same-frame guarantee still holds (feedback appears on the frame input is processed). | Feedback degradation at low framerates is acceptable. Measurement accuracy is not affected. |
| Feedback during color perception module | All chromatic feedback suppressed. No cyan ring. No amber border. Sound = neutral click only. | Color test requires an uncontaminated visual field. Any hue in feedback could interfere with plate perception. |

## Dependencies

| System | Direction | Nature of Dependency |
|--------|-----------|---------------------|
| Stimulus-Response Engine | This depends on SRE | Listens to all trial event signals. Cannot trigger feedback without stimulus-response events. Hard dependency. |
| Color Perception System | This depends on CPS (soft) | Listens to plate response events for muted feedback. Can function without it (RT-only mode). Soft dependency. |

**Upstream dependencies:** Stimulus-Response Engine (hard), Color Perception System (soft).
**Downstream dependencies:** None — this is a leaf system. Nothing depends on the feedback system.

## Tuning Knobs

| Parameter | Current Value | Safe Range | Effect of Increase | Effect of Decrease |
|-----------|--------------|------------|-------------------|-------------------|
| `ring_max_radius_px` | 60 | 30–120 | Larger ring; more dramatic feedback | Smaller ring; subtler feedback |
| `ring_duration_ms` | 120 | 60–200 | Slower ring expansion; more lingering | Faster ring; snappier feel |
| `scale_amplitude` | 0.12 | 0.05–0.20 | More dramatic scale pulse; more noticeable | Subtler pulse; more clinical |
| `scale_duration_ms` | 80 | 40–120 | Slower pulse; more visible | Faster pulse; snappier |
| `pop_volume_db` | -6 | -12–0 | Louder pop; more satisfying but potentially fatiguing over 20 trials | Quieter pop; less presence |
| `early_warning_duration_ms` | 500 | 300–1000 | Longer "Too early" display | Shorter display; faster restart |
| `ring_speed_bonus` | 0.8 | 0.0–1.5 | More visible difference between fast/slow responses | Less difference; more uniform feedback |

**Interaction warnings:**
- `scale_duration_ms` must be shorter than `inter_trial_delay_ms` (800ms) from the Stimulus-Response Engine. If scale animation outlasts the inter-trial delay, visual artifacts occur.
- `ring_speed_bonus` at 0.0 disables performance scaling entirely (ring speed constant for all RTs). At 1.5, fast responses are dramatically faster — may distract.

## Visual/Audio Requirements

See the Feedback Events table in Detailed Design — it serves as the complete
visual/audio specification for this system. All events are itemized with exact
durations, colors, and audio levels.

**Audio asset list:**
| Asset | Format | Duration | Level | Description |
|-------|--------|----------|-------|-------------|
| `sfx_tap_response.ogg` | OGG Vorbis, 44.1kHz, mono | 60ms | -6 dBFS peak | Crisp percussive pop for valid response |
| `sfx_early_warning.ogg` | OGG Vorbis, 44.1kHz, mono | 200ms | -12 dBFS peak | Low sine tone for early response |
| `sfx_module_start.ogg` | OGG Vorbis, 44.1kHz, mono | 40ms | -12 dBFS peak | Soft click for module start and plate responses |
| `sfx_module_complete.ogg` | OGG Vorbis, 44.1kHz, mono | 150ms | -6 dBFS peak | Two ascending tones for module completion |

## Game Feel

### Feel Reference

Should feel like **a premium mechanical keyboard keystroke** — the feedback is
instant, crisp, and deeply satisfying in its precision. The pop sound should have
the click quality of a Cherry MX Blue switch, not the thud of a membrane key. Anti-
reference: NOT like mobile notification sounds (delayed, soft, ambient) — feedback
must feel like a direct physical consequence of the tap.

### Input Responsiveness

| Action | Max Input-to-Response Latency (ms) | Frame Budget (at 60fps) | Notes |
|--------|-----------------------------------|------------------------|-------|
| Valid response (visual + audio) | 0ms | 0 frames additional | MUST fire in same render frame as input. Non-negotiable. |
| Early response warning | 16ms | 1 frame | Acceptable — player already knows they tapped early. |
| Timeout feedback | 0ms | 0 frames | Triggers at exact timeout threshold. |

### Weight and Responsiveness Profile

- **Weight**: Weightless but present. The tap has no physical weight — it's a
  signal, not a force. But the audio pop gives it presence and substance.
- **Snap quality**: Maximum. Binary, instant, no interpolation on the RT display.
  The number doesn't animate to its value — it appears.
- **Failure texture**: Early response is a gentle "not yet" — warm amber border,
  low tone, not aggressive. Timeout is absence — the system simply stops waiting.
  Neither feels like punishment.

### Feel Acceptance Criteria

- [ ] Response pop sound is described as "crisp" or "satisfying" by playtesters
- [ ] No playtester perceives a delay between tap and feedback
- [ ] The ring animation feels like a "ripple" from the tap point, not a UI effect
- [ ] Early response feedback feels like a gentle nudge, not an error buzzer
- [ ] After 20 trials, the repeated feedback is not described as "annoying" or "fatiguing"

## UI Requirements

The feedback system does not own persistent UI elements. It produces transient
visual effects that overlay the test screen:

| Element | Position | Lifetime | Notes |
|---------|----------|----------|-------|
| Cyan ring | Centered on input position | 120ms, then removed | Created and destroyed per-event |
| RT number | Center screen, below stimulus | 1.5s, then fades over 200ms | Managed by feedback, not persistent UI |
| "Too early" text | Center screen | 500ms | Managed by feedback |
| "No response" text | Center screen | 800ms | Managed by feedback |

## Cross-References

| This Document References | Target GDD | Specific Element Referenced | Nature |
|--------------------------|-----------|----------------------------|--------|
| Listens to trial event signals | `design/gdd/stimulus-response-engine.md` | `response_recorded`, `early_response`, `timeout` signals | State trigger |
| Listens to plate response events | `design/gdd/color-perception-system.md` | `plate_responded` signal | State trigger |
| Uses art bible color tokens | Art Bible (`design/art/art-bible.md`) | `--precision-cyan`, `--alert-amber`, `--signal-white` color definitions | Rule dependency |

## Acceptance Criteria

- [ ] GIVEN a valid response is recorded, WHEN feedback triggers, THEN visual feedback (scale pulse + ring) and audio (pop) appear in the same render frame as the input event
- [ ] GIVEN a valid response, WHEN the RT number displays, THEN it appears at its final value instantly — no counting animation or rollup
- [ ] GIVEN an early response, WHEN feedback triggers, THEN amber border flash and "Too early" text appear with low tone audio, visually distinct from valid response feedback
- [ ] GIVEN a timeout, WHEN feedback triggers, THEN stimulus fades and "No response" text appears with no audio
- [ ] GIVEN the color perception module is active, WHEN a plate response occurs, THEN only neutral feedback (click + opacity fade) is used — no cyan ring, no chromatic elements
- [ ] GIVEN a response with RT = 150ms and another with RT = 350ms, WHEN ring animations are compared, THEN the fast-response ring expands noticeably faster
- [ ] GIVEN the browser tab is muted, WHEN feedback triggers, THEN visual feedback plays normally without errors
- [ ] GIVEN 20 consecutive trials, WHEN feedback plays for each, THEN no audio clipping, visual artifact accumulation, or performance degradation occurs
- [ ] Performance: All feedback rendering completes within 1ms of processing time per event
- [ ] No hardcoded feedback parameters — all values from Tuning Knobs loaded from configuration

## Open Questions

| Question | Owner | Deadline | Resolution |
|----------|-------|----------|-----------|
| Should the pop sound vary slightly in pitch based on RT (higher pitch = faster)? | Sound Designer | Sprint 1 | Current design: no pitch variation. May add subliminal reinforcement. |
| Should there be a subtle background ambient tone during the wait phase? | Audio Director | Sprint 1 | Game concept says "no music during tests" — but ambient is not music. |
| Is the 0.12 scale amplitude noticeable enough on small screens? | UX Designer | Before Vertical Slice | May need responsive scaling based on viewport size. |
