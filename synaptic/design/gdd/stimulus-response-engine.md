# Stimulus-Response Engine

> **Status**: Designed
> **Author**: game-designer + systems-designer
> **Last Updated**: 2026-04-12
> **Implements Pillar**: Pillar 2 (Every Tap Feels Crisp), Pillar 1 (Measure, Don't Guess)

## Summary

The stimulus-response engine is the foundational measurement system that presents
timed visual stimuli, captures player input with maximum browser-achievable precision,
and records reaction time in milliseconds. It is the atomic unit of every test module
in Synaptic — all other systems build on its timing and data output.

> **Quick reference** — Layer: `Foundation` · Priority: `MVP` · Key deps: `None`

## Overview

The stimulus-response engine handles one job: present a visual target after a
randomized delay, capture the exact moment the player reacts, and compute the
elapsed time between stimulus onset and response. A single trial takes 2–6 seconds.
A module runs 20 trials in sequence, producing a dataset of reaction times. The
player's only action is a single input (click or keypress) — the system handles
everything else: timing randomization, early-response detection, timeout handling,
and trial-level data packaging. This system exists because measurement precision
is the foundation of every claim the game makes about the player's cognitive
performance. If the timing is wrong, every downstream number is meaningless.

## Player Fantasy

You are wired into a precision instrument. The screen goes dark, your focus
narrows, and the moment the stimulus appears your body reacts before conscious
thought catches up. The number that appears — 247ms — is not a game score. It is
a measurement of your nervous system, captured with clinical precision. The
fantasy is **being measured accurately** — the satisfaction that the instrument
respects your performance, that a 12ms improvement between sessions is real
signal, not noise. This system serves Pillar 2 (Every Tap Feels Crisp) by
ensuring zero perceived latency between input and acknowledgment, and Pillar 1
(Measure, Don't Guess) by capturing timing with the highest precision the
browser platform allows.

## Detailed Design

### Core Rules

1. A **trial** is one stimulus-response cycle: wait → present → capture → record.
2. The **wait phase** begins with a blank screen (canvas color `#0A0A0F`). Duration
   is drawn from a uniform random distribution between `min_wait_ms` (1500ms) and
   `max_wait_ms` (4000ms). The random delay prevents rhythmic anticipation.
3. The **stimulus** appears instantaneously — no fade-in, no entrance animation.
   For the basic reaction time module, the stimulus is a filled white circle
   (`--stim-neutral`) centered in the viewport, diameter `stimulus_size_px` (80px).
4. **Input capture** registers on the first `InputEventMouseButton` (pressed=true)
   or `InputEventKey` (pressed=true) received after stimulus onset. The timestamp
   is captured using `Time.get_ticks_msec()` at the moment the `_input()` callback
   fires.
5. **Reaction time** = `response_timestamp - stimulus_timestamp`, in milliseconds.
   This value has a precision floor of approximately ±5ms due to browser event loop
   granularity in HTML5 export.
6. An **early response** (input during the wait phase, before stimulus appears)
   triggers a "Too early" warning. The trial is voided, not counted, and a new
   wait phase begins with a fresh random delay.
7. A **timeout** occurs if no input is received within `timeout_threshold_ms`
   (2000ms) after stimulus onset. The trial is recorded with the timeout flag set
   and excluded from median/percentile calculations but included in consistency
   metrics.
8. A **module** consists of `trials_per_module` (20) sequential trials. After all
   trials complete, the module emits its complete dataset.
9. Input is **single-event only**: after a response is registered, further inputs
   are ignored until the next trial's stimulus appears. There is no double-tap or
   multi-press mechanic.

### States and Transitions

| State | Entry Condition | Exit Condition | Behavior |
|-------|----------------|----------------|----------|
| `IDLE` | Module not started or completed | `start_module()` called | No input processing. Screen shows module intro or results. |
| `WAITING` | Trial begins (after IDLE or previous TRIAL_COMPLETE) | Timer expires (random delay elapsed) OR early input detected | Blank screen. Timer counting down. Any input = early response → reset delay. |
| `STIMULUS_PRESENTED` | Wait timer expires | Input received OR timeout timer expires | Stimulus visible. `stimulus_timestamp` recorded. Awaiting input. |
| `RESPONSE_CAPTURED` | Valid input received during STIMULUS_PRESENTED | Processing complete (< 1 frame) | `response_timestamp` recorded. RT calculated. Signals emitted. Transitions immediately to TRIAL_COMPLETE. |
| `TIMEOUT` | Timeout timer expires during STIMULUS_PRESENTED | Processing complete (< 1 frame) | Trial recorded as timeout. Signals emitted. Transitions to TRIAL_COMPLETE. |
| `TRIAL_COMPLETE` | Response captured or timeout processed | Next trial begins (after `inter_trial_delay_ms`) OR module complete | Per-trial data packaged. If `trial_index < trials_per_module`, advance to WAITING. Otherwise → MODULE_COMPLETE. |
| `MODULE_COMPLETE` | All trials finished | External reset or new module start | Complete dataset emitted via `module_completed` signal. System returns to IDLE. |
| `EARLY_RESPONSE` | Input received during WAITING state | Feedback shown (500ms) | "Too early" feedback displayed. Trial voided. Returns to WAITING with new random delay. Same trial index (not incremented). |

### Interactions with Other Systems

| System | Direction | Interface |
|--------|-----------|-----------|
| **Feedback System** | Downstream (consumes) | Listens to `response_recorded` signal. Receives: `reaction_time_ms: int`, `trial_index: int`, `is_outlier: bool`. Triggers visual/audio juice. |
| **Statistical Analysis Engine** | Downstream (consumes) | Receives `module_completed` signal with `module_data: Array[TrialResult]`. Each TrialResult contains: `rt_ms: int`, `is_timeout: bool`, `is_early: bool`, `stimulus_timestamp: int`, `response_timestamp: int`. |
| **Color Perception System** | Downstream (extends) | Extends this engine by replacing the stimulus type (circle → Ishihara plate) and adding response correctness evaluation. Inherits the timing, state machine, and input capture logic. |
| **Test Sequencing** | Downstream (orchestrates) | Calls `start_module(config: ModuleConfig)` to begin a test. Listens to `module_completed` to advance to next module in battery. |

## Formulas

### Reaction Time Measurement

The reaction_time formula is defined as:

`reaction_time_ms = response_timestamp - stimulus_timestamp`

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| response_timestamp | t_r | int | 0–∞ | `Time.get_ticks_msec()` at input callback |
| stimulus_timestamp | t_s | int | 0–∞ | `Time.get_ticks_msec()` at stimulus display |
| reaction_time_ms | RT | int | 0–2000 | Elapsed time; clamped by timeout threshold |

**Output Range:** 0ms to 2000ms under normal conditions. Values of 0ms are
flagged as measurement artifacts. Typical human range: 150ms–400ms.

**Precision:** ±5ms due to browser event loop quantization in HTML5 export.
All displayed values include this uncertainty band.

**Example:** Stimulus appears at t_s = 142350ms, player clicks at t_r = 142597ms.
RT = 142597 - 142350 = 247ms (displayed as "247ms ±5ms").

### Inter-Stimulus Interval

The wait_duration formula is defined as:

`wait_duration_ms = uniform_random(min_wait_ms, max_wait_ms)`

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| min_wait_ms | w_min | int | 1000–3000 | Minimum delay before stimulus |
| max_wait_ms | w_max | int | 2000–6000 | Maximum delay before stimulus |
| wait_duration_ms | W | int | w_min–w_max | Actual delay for this trial |

**Output Range:** 1500ms to 4000ms at default tuning.

**Example:** With defaults, a trial might draw W = 2731ms. The player sees a blank
screen for 2.7 seconds before the stimulus appears.

### Module Statistics (computed locally, passed to Statistical Analysis)

`module_median_rt = median(valid_trials.map(t => t.rt_ms))`
`module_mean_rt = mean(valid_trials.map(t => t.rt_ms))`
`valid_trials = all_trials.filter(t => !t.is_timeout && !t.is_early)`

## Edge Cases

| Scenario | Expected Behavior | Rationale |
|----------|------------------|-----------|
| Player responds during wait phase (before stimulus) | "Too early" feedback shown for 500ms. Trial voided and restarted with new random delay. Trial counter does not increment. | Prevents gaming the system by clicking rhythmically. Pillar 1: real measurement only. |
| Player holds key/button continuously | Only the first `pressed=true` event registers. Subsequent events ignored until a `pressed=false` is received followed by a new `pressed=true`. | Prevents trivial input exploits. |
| No response within 2000ms of stimulus | Trial recorded as timeout (`is_timeout=true`). Excluded from median RT calculation. Included in consistency/reliability metrics. "No response" feedback shown. | 2000ms exceeds any plausible human simple RT. Recording it preserves data for the Statistical Analysis Engine's consistency analysis. |
| Browser tab loses focus during active trial | Trial is voided. On regain focus, a 2-second "Refocusing..." pause occurs, then trial restarts with new random delay. Trial counter does not increment. | Focus loss invalidates timing — the player's attention was elsewhere. Pillar 1: never record bad data. |
| Two input events arrive in the same frame | First event processed; second discarded. First is determined by `_input()` callback order (Godot processes in tree order). | Deterministic resolution prevents measurement ambiguity. |
| RT = 0ms (stimulus and response same timestamp) | Recorded with `is_artifact=true` flag. Excluded from all statistical calculations. "Measurement artifact" noted in raw data. | Physiologically impossible for simple RT. Likely a timing system artifact. |
| Player triggers early response 3+ times consecutively | After 3 consecutive early responses, show a coaching message: "Wait for the circle to appear before clicking." Continue testing — do not lock out. | Prevents frustration from unclear instructions without gatekeeping access. |
| Module is interrupted mid-trial (player navigates away) | Current trial voided. Completed trials preserved in memory. Module can be resumed or restarted via Test Sequencing. | Respects player agency. Completed data is valuable even if module is incomplete. |

## Dependencies

| System | Direction | Nature of Dependency |
|--------|-----------|---------------------|
| Feedback System | This is depended on by Feedback | Provides `response_recorded` signal with RT and outlier status. Feedback System renders visual/audio juice. |
| Statistical Analysis Engine | This is depended on by Stats | Provides `module_completed` signal with full trial dataset. Stats engine computes percentiles and norms. |
| Color Perception System | This is depended on by Color | Color system extends this engine's stimulus type and adds correctness evaluation. Inherits timing, states, and input capture. |
| Test Sequencing | This is depended on by Sequencing | Sequencing calls `start_module()` and listens for `module_completed`. This system is orchestrated by Sequencing but operates independently per module. |

**Upstream dependencies:** None. This is a foundation system.

## Tuning Knobs

| Parameter | Current Value | Safe Range | Effect of Increase | Effect of Decrease |
|-----------|--------------|------------|-------------------|-------------------|
| `min_wait_ms` | 1500 | 1000–3000 | Longer minimum wait; more suspense, slower pacing | Shorter wait; faster pacing but easier to anticipate |
| `max_wait_ms` | 4000 | 2000–6000 | Longer maximum wait; more unpredictable, can feel tedious past 5000 | Shorter maximum; faster pacing but narrower randomization window |
| `timeout_threshold_ms` | 2000 | 1500–5000 | More lenient timeout; captures slow but genuine responses | Stricter timeout; more timeouts for slower responders |
| `trials_per_module` | 20 | 10–50 | More data points; better statistical power, longer sessions | Fewer data points; faster sessions, less reliable statistics. Below 15 trials, median RT has high variance. |
| `stimulus_size_px` | 80 | 40–200 | Larger target; easier to see, less visual precision required | Smaller target; harder to spot, adds visual search component (undesirable for simple RT) |
| `inter_trial_delay_ms` | 800 | 400–1500 | Longer pause between trials; more recovery time | Shorter pause; faster rhythm, can feel rushed |
| `early_response_cooldown_ms` | 500 | 300–1000 | Longer "Too early" feedback display | Shorter feedback; faster restart |

**Interaction warnings:**
- `min_wait_ms` and `max_wait_ms` must maintain at least 500ms gap. If gap is too narrow, delays become predictable.
- `trials_per_module` below 15 degrades statistical reliability of the module's median RT. The Statistical Analysis Engine should flag low-trial-count modules.

## Visual/Audio Requirements

| Event | Visual Feedback | Audio Feedback | Priority |
|-------|----------------|---------------|----------|
| Stimulus onset | White circle (`--stim-neutral`) appears center-screen, instantaneously. No fade, no animation. | None — silence is part of the test environment. | Critical |
| Valid response | Stimulus circle scales 1.0 → 0.88 → 1.0 over 80ms (spring, no overshoot). 1px `--precision-cyan` ring expands from tap origin at 120ms and fades. RT number appears in 48px JetBrains Mono. | Short crisp pop (< 100ms, percussive). Pitch unchanged regardless of speed. | Critical |
| Early response | Screen flashes `--alert-amber` border (2px) for 200ms. "Too early" text appears in `--signal-white`, 24px Inter, center-screen. | Soft low-pitched tone (200ms, sine wave, -12dB relative to pop). | High |
| Timeout | Stimulus fades to 20% opacity over 300ms. "No response" text appears. | None — absence of sound reinforces absence of response. | Medium |
| Module start | Trial counter appears top-right: "1/20" in `--neutral-text`, 16px Inter. Brief 1-second countdown: "Ready" → blank screen → first wait phase. | Single soft click to mark module start. | Medium |
| Module complete | All stimuli clear. "Complete" text fades in. Transition to results. | Completion chime — two ascending tones, 150ms total. | Medium |

## Game Feel

### Feel Reference

Should feel like **Human Benchmark's simple reaction time test** — the gold standard
for browser-based reflex testing. The specific quality to match: the instant visual
acknowledgment of input, where the response number appears on the same frame the
player acts. NOT like mobile brain-training apps (Lumosity, Peak) which add entrance
animations and transition delays between the player's action and the result display.

### Input Responsiveness

| Action | Max Input-to-Response Latency (ms) | Frame Budget (at 60fps) | Notes |
|--------|-----------------------------------|------------------------|-------|
| Tap/click response | 0ms (same frame) | 0 frames additional | Visual acknowledgment MUST appear in the same render frame as input capture. This is the single most important feel target. |
| RT number display | 0ms (same frame) | 0 frames additional | The number appears with the tap feedback, not after an animation delay. |
| "Too early" feedback | 16ms (next frame acceptable) | 1 frame | Slightly less critical — player knows they made an error. |

### Weight and Responsiveness Profile

- **Weight**: Weightless. Actions have no inertia, no momentum, no follow-through.
  The interaction is binary: stimulus → response. This is an instrument, not a
  physical simulation.
- **Player control**: Maximum. The player's only action is a single tap. There is
  nothing to course-correct. Control is total within the action (when to tap) and
  zero outside it (cannot affect stimulus timing).
- **Snap quality**: Maximum crispness. Input → result is binary and instantaneous.
  No interpolation, no smoothing, no easing between states.
- **Acceleration model**: N/A — there is no movement. State transitions are instant.
- **Failure texture**: Fair and informative. "Too early" is gentle coaching, not
  punishment. Timeout is neutral observation, not judgment. The system never makes
  the player feel bad — it reports what happened.

### Feel Acceptance Criteria

- [ ] Response feedback appears on the same frame as input — no perceivable delay
- [ ] RT number display is instantaneous — no counting animation or rollup
- [ ] No playtester uses "laggy," "delayed," or "sluggish" to describe the input
- [ ] The difference between a 200ms and a 300ms reaction is felt as meaningfully different in the feedback
- [ ] Early response feedback feels like a gentle nudge, not a punishment

## UI Requirements

| Information | Display Location | Update Frequency | Condition |
|-------------|-----------------|-----------------|-----------|
| Trial counter ("5/20") | Top-right corner | Per trial completion | During active module |
| Reaction time (ms) | Center screen, below stimulus position | On response | After each valid response, visible for 1.5s |
| ±5ms precision band | Below RT number, smaller text | On response | Always shown with RT |
| "Too early" warning | Center screen | On early response | Visible for 500ms |
| "No response" timeout | Center screen | On timeout | Visible for 800ms |
| Progress bar | Top edge, full width, 1px | Continuous fill | During active module |

## Cross-References

| This Document References | Target GDD | Specific Element Referenced | Nature |
|--------------------------|-----------|----------------------------|--------|
| Module dataset feeds statistical computation | `design/gdd/statistical-analysis-engine.md` | `module_data: Array[TrialResult]` input format | Data dependency |
| Response events trigger visual/audio feedback | `design/gdd/feedback-system.md` | `response_recorded` signal consumption | State trigger |
| Stimulus type is extended by color perception | `design/gdd/color-perception-system.md` | Stimulus replacement (circle → Ishihara plate) | Ownership handoff |
| Module lifecycle orchestrated by sequencing | `design/gdd/test-sequencing.md` | `start_module()` / `module_completed` interface | State trigger |

## Acceptance Criteria

- [ ] GIVEN the system is in WAITING state, WHEN player presses any key or clicks, THEN "Too early" feedback is displayed and trial restarts with a new random delay
- [ ] GIVEN a stimulus is displayed, WHEN the player responds, THEN reaction time is recorded as the difference between stimulus onset and input timestamps, within ±5ms precision
- [ ] GIVEN a stimulus is displayed, WHEN 2000ms passes with no input, THEN the trial is recorded as a timeout and excluded from median calculations
- [ ] GIVEN a module of 20 trials, WHEN all trials complete, THEN `module_completed` signal fires with an array containing all 20 TrialResult objects
- [ ] GIVEN the browser tab loses focus during an active trial, WHEN the tab regains focus, THEN the current trial is voided and restarted after a 2-second pause
- [ ] GIVEN the wait phase, WHEN the random delay is measured across 100+ trials, THEN the distribution is approximately uniform between min_wait_ms and max_wait_ms
- [ ] GIVEN a valid response, WHEN the RT number displays, THEN it appears in the same render frame as the input event (0 additional frames of latency)
- [ ] GIVEN any trial state, WHEN input is registered, THEN only the first input event per trial is processed; subsequent events are ignored
- [ ] GIVEN 3 consecutive early responses, WHEN the player triggers a 4th, THEN a coaching message appears in addition to the standard "Too early" feedback
- [ ] Performance: Full trial cycle (stimulus display → RT calculation → signal emission) completes within 2ms of processing time
- [ ] No hardcoded tuning values — all parameters from Tuning Knobs section are loaded from external configuration

## Open Questions

| Question | Owner | Deadline | Resolution |
|----------|-------|----------|-----------|
| What is the actual input timing precision in Godot 4.6 HTML5 export? ±5ms is estimated — needs benchmarking. | Technical Director | Before Alpha | Prototype needed |
| Should `Time.get_ticks_msec()` or `Time.get_ticks_usec()` be used for timestamps? | Engine Programmer | Before implementation | Benchmark both in HTML5 |
| Should early response penalty increase (longer cooldown) after repeated early taps? | Game Designer | Sprint 1 | Current design: no escalation, just coaching after 3rd |
