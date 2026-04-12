# HUD Design

> **Status**: Complete
> **Author**: ux-designer (autonomous)
> **Last Updated**: 2026-04-12
> **Template**: HUD Design

---

## HUD Philosophy

**Precision Minimal — The screen is the instrument, not the game.**

During tests, the entire viewport IS the test environment. The HUD does not overlay the game — it IS the interface. Every persistent element on screen serves exactly one purpose: deliver measurement feedback or orient the player within the test sequence. Nothing decorates. When a test is running, visual chrome is reduced to its absolute minimum so the stimulus has total attention.

This maps to: "Minimal but present — only critical information visible, everything else contextual." The philosophy is closer to Dark Souls than Diablo, but the reason is clinical rather than aesthetic: every extraneous UI element during a reaction test is a potential distractor that corrupts the measurement.

**Design test**: "Should we add a timer countdown until the stimulus appears?" → No. The randomized wait IS the test condition. Showing a countdown would allow the player to pre-load their response, violating Pillar 1 (Measure, Don't Guess).

---

## Information Architecture

### Full Information Inventory

From all GDD UI Requirements sections:

| Element | Source GDD | Phase |
|---|---|---|
| Trial counter ("5/20") | Stimulus-Response Engine | During RT module |
| Progress bar (top edge, 1px) | Stimulus-Response Engine | During RT module |
| RT number (ms display) | Stimulus-Response Engine | Post-response, 1.5s |
| ±5ms precision band | Stimulus-Response Engine | Post-response with RT |
| "Too early" warning | Stimulus-Response Engine | On early response, 500ms |
| "No response" text | Stimulus-Response Engine | On timeout, 800ms |
| Plate counter ("3/12") | Color Perception System | During color module |
| Progress bar (top edge, 1px) | Color Perception System | During color module |
| Pre-framing card text | Color Perception System | Before first plate |
| Response buttons (number pad + "no pattern") | Color Perception System | During PLATE_DISPLAYED |
| Transition card (module name + countdown) | Test Sequencing | Between modules in battery |

### Categorization

| Element | Category | Rationale |
|---|---|---|
| Trial / Plate counter | **Must Show** | Player needs to know progress through the module; affects pacing decision (take a breath before trial 18/20) |
| Progress bar (1px) | **Must Show** | Lightweight ambient progress — no cognitive load, pure orientation signal |
| RT number + precision band | **Contextual** | Appears for exactly 1.5s after each valid response, then disappears — transient, not persistent |
| "Too early" warning | **Contextual** | Appears only on early response trigger; dismissed automatically |
| "No response" text | **Contextual** | Appears only on timeout trigger; dismissed automatically |
| Response buttons (color module) | **Must Show** | Player cannot complete a plate without these — they ARE the interaction mechanism |
| Pre-framing card | **Contextual** | Shown once before the first plate of the color module |
| Transition card | **Contextual** | Shown only between modules in a full battery run |

**Conflict check**: The "Must Show" list contains 3 items (counter, progress bar, response buttons). The progress bar is 1px — essentially invisible. The counter is top-right at 12px neutral text. This is compatible with the Precision Minimal philosophy: both items have near-zero visual weight and zero cognitive demand during the active trial. No conflict.

---

## Layout Zones

The HUD is context-sensitive across three test states:

### Zone Map: Reaction Time Module

```
┌─────────────────────────────────────────────────────┐
│ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡░░░ 5/20  │  ← TOP BAR (progress px + counter)
├─────────────────────────────────────────────────────┤
│                                                     │
│                                                     │
│                    ●  ←stimulus                     │  ← STIMULUS ZONE (center, ~30% area)
│                                                     │
│                 [312 ms]                            │
│                 [±5 ms]                             │  ← FEEDBACK ZONE (center, below stimulus)
│                                                     │
│                                                     │
│                                                     │
│                                                     │
└─────────────────────────────────────────────────────┘
(Transient feedback elements appear and dissolve in FEEDBACK ZONE)
```

### Zone Map: Color Perception Module

```
┌─────────────────────────────────────────────────────┐
│ ≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡≡░░░░ 3/12  │  ← TOP BAR (progress px + counter)
├─────────────────────────────────────────────────────┤
│                                                     │
│                                                     │
│              [  Ishihara Plate  ]                   │  ← PLATE ZONE (center, ~50% area)
│                                                     │
│                                                     │
│  [1] [2] [3] [4] [5] [6] [7] [8] [9]              │
│           [I don't see a pattern]                   │  ← RESPONSE ZONE (below plate)
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Zone Map: Transition Card (between modules)

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│                  Color Perception                   │  ← MODULE NAME (center, 24px)
│          This test measures color vision.           │  ← DESCRIPTION (center, 14px)
│                                                     │
│                                                     │
│                       3                             │  ← COUNTDOWN (center-bottom, 48px mono)
│                                                     │
│           [press any key to skip]                   │  ← SKIP HINT (footer, 12px)
└─────────────────────────────────────────────────────┘
```

---

## HUD Elements

### 1. Progress Bar

| Property | Value |
|---|---|
| **Category** | Must Show |
| **Content** | Trial/plate completion progress — fills left to right |
| **Visual form** | 1px horizontal hairline at top edge of viewport, full width |
| **Color** | `--neutral-mid` filling from left; unfilled portion = `--neutral-dark` |
| **Update behavior** | Jumps to new value immediately on trial/plate completion — no lerp |
| **Contextual trigger** | Visible whenever a test module is active |
| **Animation** | Instant jump only — no smooth fill (smooth fill would distort time perception during active trials) |

### 2. Trial / Plate Counter

| Property | Value |
|---|---|
| **Category** | Must Show |
| **Content** | "[N]/[total]" — e.g., "5/20" or "3/12" |
| **Visual form** | Numeric text, 12px JetBrains Mono 400, `--neutral-text`, top-right corner |
| **Update behavior** | Updates immediately after each trial/plate completes |
| **Contextual trigger** | Visible whenever a test module is active |
| **Animation** | None — instant text update |
| **Accessibility** | `aria-label="Trial 5 of 20"` dynamically updated |

### 3. RT Number Display (transient)

| Property | Value |
|---|---|
| **Category** | Contextual |
| **Content** | Reaction time in milliseconds — e.g., "312" (no "ms" label; the number IS the measurement) |
| **Visual form** | 48px JetBrains Mono 700, `--signal-white`, center screen below stimulus position |
| **Color coding** | Fast (< 200ms): subtle warm tint `--perf-fast`; Average (200-400ms): `--signal-white`; Slow (> 400ms): no tint (cold white is default) |
| **Update behavior** | Appears instantaneously on response frame — NO counting animation, NO rollup |
| **Lifetime** | 1500ms visible, then 200ms fade to 0 opacity, then removed from DOM |
| **Contextual trigger** | After each valid response in RT module only |
| **Animation** | Scale pulse 1.0→1.12→1.0 over 80ms on appear (same-frame juice), then holds; fade-out only |

### 4. ±5ms Precision Band (transient)

| Property | Value |
|---|---|
| **Category** | Contextual |
| **Content** | "±5ms" — precision uncertainty disclosure |
| **Visual form** | 11px Inter 400, `--neutral-text` (low contrast — secondary disclosure), below RT number |
| **Lifetime** | Same as RT number — appears and disappears with it |
| **Contextual trigger** | Always shown with RT number, never shown without it |
| **Rationale** | Pillar 1 (Measure, Don't Guess) — always show measurement uncertainty |

### 5. "Too Early" Warning (transient)

| Property | Value |
|---|---|
| **Category** | Contextual |
| **Content** | "Too early" |
| **Visual form** | 16px Inter 500, `--alert-amber`, center screen; amber border flash on viewport edge (2px) |
| **Lifetime** | 500ms, no fade — cuts out instantly |
| **Contextual trigger** | Input received during WAITING state (before stimulus appears) |
| **Audio** | Low tone pop (see Feedback System GDD) — distinct from valid response pop |
| **3rd-strike variant** | After 3 consecutive early responses: add coaching message below in 12px: "Wait for the circle to appear" |

### 6. "No Response" Text (transient)

| Property | Value |
|---|---|
| **Category** | Contextual |
| **Content** | "No response" |
| **Visual form** | 14px Inter 400, `--neutral-text` (deliberately low contrast — informational, not alarming) |
| **Lifetime** | 800ms, then removed |
| **Contextual trigger** | Trial timeout (2000ms with no input) |
| **Audio** | None — silence is appropriate for a non-event |
| **Rationale** | A missed trial is data, not failure. Low contrast communicates this — it's an observation, not a warning. |

### 7. Response Buttons — Color Module (persistent during PLATE_DISPLAYED)

| Property | Value |
|---|---|
| **Category** | Must Show |
| **Content** | Number keys 1–9 (digit identifiers visible in Ishihara plates) + "I don't see a pattern" |
| **Visual form** | 9 square digit buttons in a row, 40×40px each, 4px gap, 14px JetBrains Mono; "I don't see a pattern" is a full-width text button below, 13px Inter 400 |
| **Color** | Buttons: `--neutral-dark` background, `--neutral-text` label, `--signal-white` on hover/focus |
| **Layout** | Horizontally centered below plate, fixed position — does not shift between plates |
| **Update behavior** | Present for entire duration of plate display; disappears when plate transitions |
| **Keyboard mapping** | Keys 1–9 map directly to digit buttons; Tab + Space/Enter for "no pattern" button |
| **Color contamination rule** | Buttons use NO chromatic colors — achromatic only. No `--precision-cyan` or `--alert-amber` in the color module viewport |

### 8. Pre-Framing Card (Color Module)

| Property | Value |
|---|---|
| **Category** | Contextual |
| **Content** | "This test measures color discrimination. There is no pass or fail." |
| **Visual form** | 16px Inter 400, `--signal-white`, center screen; "Press any key to continue" in 12px `--neutral-text` at bottom |
| **Lifetime** | Player-dismissed (keypress or click) |
| **Contextual trigger** | Once, before the first plate in any color module session |
| **Emotional register** | Neutral, clinical. Not reassuring ("Don't worry!") — matter-of-fact. |

### 9. Transition Card (between battery modules)

| Property | Value |
|---|---|
| **Category** | Contextual |
| **Content** | Module name (24px Inter 600), 1-sentence description (14px Inter 400), countdown number (48px JetBrains Mono 700), skip hint (12px Inter 400, `--neutral-text`) |
| **Lifetime** | 3000ms countdown; player can skip by pressing any key |
| **Contextual trigger** | Between modules in a full battery run only; not shown for individual module runs |
| **Countdown audio** | Soft click per number change (same SFX as module start) |

---

## Visual Budget

**Maximum simultaneous HUD elements: 4**

This maximum occurs in the post-response state of the RT module: progress bar + trial counter + RT number + precision band. All other states have fewer elements. The 4-element maximum leaves >95% of viewport unoccupied by chrome, consistent with the Precision Minimal philosophy.

| State | Element Count | % Viewport Chrome |
|---|---|---|
| Waiting for stimulus | 2 (bar + counter) | <1% |
| Stimulus visible | 2 (bar + counter) | <1% |
| Post-response | 4 (bar + counter + RT + band) | ~2% |
| Early response | 3 (bar + counter + warning) | ~1% |
| Timeout | 3 (bar + counter + notice) | ~1% |
| Plate displayed | 3 (bar + counter + buttons) | ~8% (buttons occupy bottom zone) |

**Tuning Knobs** (values owned by respective GDDs — listed here for HUD implementer reference):

| Parameter | Value | Source GDD |
|---|---|---|
| RT display lifetime | 1500ms | Stimulus-Response Engine |
| RT display fade duration | 200ms | Feedback System |
| "Too early" lifetime | 500ms | Feedback System |
| "No response" lifetime | 800ms | Feedback System |
| Transition countdown duration | 3000ms | Test Sequencing |
| Cyan ring lifetime | 120ms | Feedback System |

---

## Dynamic Behaviors

### HUD Density by State

| State | Visible Elements | HUD Density |
|---|---|---|
| **Waiting for stimulus** (RT) | Progress bar, trial counter only | Minimal (2 elements) |
| **Stimulus visible** (RT) | Progress bar, trial counter only | Minimal (2 elements) — no additions during active trial |
| **Post-response** (RT) | Progress bar, trial counter, RT number, ±5ms band | Light (4 elements, 2 transient) |
| **Early response** (RT) | Progress bar, trial counter, "Too early" warning | Light (3 elements, 1 transient) |
| **Timeout** (RT) | Progress bar, trial counter, "No response" text | Light (3 elements, 1 transient) |
| **Plate displayed** (Color) | Progress bar, plate counter, response buttons | Moderate (3 elements) |
| **Pre-framing card** (Color) | Pre-framing card only | Overlay (full screen card) |
| **Transition card** | Transition card only | Overlay (full screen card) |

### Color Contamination Lockout

During the Color Perception module (from plate display to plate response), the following elements are **hard-locked off**:
- Cyan ring (`--precision-cyan`) — not used
- Amber border flash (`--alert-amber`) — not used
- Any chromatic color except `--stim-*` tokens on the plate itself

This is enforced at the module level — the color module suppresses the Feedback System's chromatic effects and substitutes neutral-only feedback (click sound + opacity fade only).

---

## Platform & Input Variants

| Platform | Variant |
|---|---|
| **Web (desktop, 1080p+)** | Default layout as specified above |
| **Web (laptop, 1280×800)** | No changes — all elements are viewport-relative, no fixed-pixel zones that would clip |
| **Web (mobile, ≤ 768px)** | **Out of scope for MVP** — mobile layout is explicitly deferred per GDD. If accessed on mobile, show advisory: "Best experienced on desktop." No layout adaptation attempted. |
| **Tab hidden** | Active trial is voided when browser tab loses focus (per Stimulus-Response Engine). HUD shows no state during hidden tab — timer is frozen until focus returns. On refocus: 2-second pause, then trial restarts. |

---

## Accessibility

**No gamepad support** — web browser game, keyboard/mouse primary.

**Keyboard access during RT module**: Any key triggers a response — no specific key mapping required. This is fully accessible for keyboard users.

**Keyboard access during Color module**: Digit keys 1–9 directly activate number buttons. Tab navigates to "I don't see a pattern" button; Space/Enter activates. Full keyboard path exists.

**Focus management**: During RT module, focus is held on the main viewport — not on any specific button. During color module, focus is on the response button row. Between modules (transition card), focus is on the skip area.

**Screen readers**:
- Trial counter has `aria-live="polite"` and `aria-label` — updates announced after each trial
- Transient feedback elements ("Too early", RT number) have `aria-live="assertive"` for immediate announcement
- Response buttons in color module have `aria-label="[digit] — press to indicate you see this number"` and "I don't see a pattern — press if no number is visible"
- Plate itself: `aria-label="Color perception plate [N] of 12 — identify the number or pattern hidden in the dots"` with `role="img"`

**Color-independent feedback**: "Too early" and "No response" are communicated via text, not only color. The amber border flash on "Too early" is supplementary — the text label carries the meaning.

**Reduced motion**: 
- RT number scale pulse → replaced with instant appear (no scale animation)
- Ring expansion animation → replaced with instant appear/disappear
- Transition card countdown → no animation affected

**Contrast**:
- RT number (`--signal-white` on `#0a0a0a`): ~20:1 — AAA
- Trial counter (`--neutral-text` on `#0a0a0a`): minimum 4.5:1 — AA
- Response buttons (`--neutral-text` on `--neutral-dark`): minimum 4.5:1 — AA
- "No response" text (low contrast by design): may fall below AA — this is an intentional design choice reflecting the emotional register (non-alarming). Flag for accessibility review before release.

---

## Open Questions

- **"No response" contrast**: Low contrast is intentional (de-emphasize missed trials) but may not meet WCAG AA (4.5:1). Decision needed: accept as-is with rationale documented, or find an achromatic value that reads as "soft" but still clears AA.
- **Mobile advisory copy**: What exact text should appear when a mobile user accesses the game at MVP? Needs copywriting.
- **RT number warm tint (`--perf-fast`)**: Exact color token value not yet defined in art bible. UX requires a warm tint for sub-200ms responses that does not introduce color as a measurement contaminant. Needs art director sign-off — must remain desaturated enough to not read as a chromatic test element.
