# Interaction Pattern Library

> **Status**: Complete (MVP set)
> **Author**: ux-designer (autonomous)
> **Last Updated**: 2026-04-12
> **Template**: Interaction Pattern Library

---

## Overview

Synaptic uses a small, tightly controlled set of interaction patterns that reflect the game's clinical instrument aesthetic. All patterns derive from three constraints:

1. **Flat, achromatic chrome** — no gradients, no decorative shadows, no color in UI structure
2. **Same-frame feedback** — all input acknowledgment fires in the same render frame as the input event
3. **Position stability** — no element changes its screen position on state change; only content and opacity change

This library is the single source of truth for how interactive elements look and behave. New patterns require an explicit addition here before they may be used in any UX spec.

---

## Pattern Catalog

| Pattern | Category | Used In |
|---|---|---|
| [Primary CTA Button](#primary-cta-button) | Input | Main Menu |
| [Module Select Button](#module-select-button) | Input | Main Menu |
| [Ghost CTA](#ghost-cta) | Input | Main Menu |
| [Digit Response Grid](#digit-response-grid) | Input | HUD (Color Module) |
| [Any-Key Trigger](#any-key-trigger) | Input | HUD (RT Module), Transition Card |
| [Focus Ring](#focus-ring) | Input / Accessibility | All interactive elements |
| [App Wordmark](#app-wordmark) | Data Display | Main Menu |
| [Ambient Counter](#ambient-counter) | Data Display | Main Menu, HUD |
| [Shortcut Hint Label](#shortcut-hint-label) | Data Display | Main Menu |
| [Hairline Progress Bar](#hairline-progress-bar) | Data Display | HUD |
| [Transient Stat Display](#transient-stat-display) | Feedback | HUD (RT Module) |
| [Transient Warning Text](#transient-warning-text) | Feedback | HUD |
| [Screen Slide Transition](#screen-slide-transition) | Navigation | All screen transitions |
| [Button Press Confirmation](#button-press-confirmation) | Feedback | All buttons |
| [Informational Overlay Card](#informational-overlay-card) | Overlay | HUD (Pre-framing, Transition) |
| [Countdown Display](#countdown-display) | Data Display | HUD (Transition Card) |

**Cross-cutting standards** (apply to all patterns):
- [Animation Standards](#animation-standards) — consolidated timing and easing table
- [Sound Standards](#sound-standards) — pattern-to-audio-event mapping

---

## Patterns

---

### Primary CTA Button

**Category**: Input
**Used In**: Main Menu ("Start Battery")

**Description**: The highest-priority action on any screen. Filled background, maximum contrast, 2px corner radius (the entire warmth budget — signals "tappable" without softening the clinical register). Used for exactly one action per screen — never two Primary CTAs on the same screen.

**Specification**:
- Size: min-width 180px, height 48px, padding 0 24px
- Background: `--signal-white`
- Label: `--neutral-900` (dark text on white fill), 18px Inter 600
- Corner radius: 2px
- Border: none
- Hover state: background shifts to `--neutral-100` (subtle dimming — instrument dimmer, not color change)
- Focus state: Focus Ring pattern (see below) — 2px `--signal-white` outline, 2px offset; inverted colors since background is white
- Press feedback: Button Press Confirmation pattern (see below)
- Disabled state: opacity 40%, pointer-events none

**Keyboard**: Space or Enter activates. The button's keyboard shortcut (if any) is documented in the Shortcut Hint Label associated with it.

**When to Use**: The single most important action the player should take on a screen. One per screen maximum.

**When NOT to Use**: Secondary actions, destructive actions, or screens with multiple equal-priority actions. Use Module Select Button or Ghost CTA instead.

**Reference**:
```
┌──────────────────┐
│   Start Battery  │   ← white fill, dark label, 2px radius
└──────────────────┘
```

---

### Module Select Button

**Category**: Input
**Used In**: Main Menu ("Reaction Time", "Color Perception")

**Description**: Secondary action button for selecting a specific module or option. Outlined (not filled) to visually subordinate it to the Primary CTA. Used for actions that are valid but less common than the primary path.

**Specification**:
- Size: min-width 140px, height 40px, padding 0 16px
- Background: transparent
- Border: 1px solid `--neutral-600`
- Label: `--neutral-200`, 14px Inter 500
- Corner radius: 2px
- Hover state: border color → `--signal-white`; label → `--signal-white`
- Focus state: Focus Ring pattern
- Press feedback: Button Press Confirmation pattern
- Disabled state: opacity 40%, pointer-events none

**When to Use**: Secondary choices that are valid paths but not the default recommendation.

**When NOT to Use**: The single primary action (use Primary CTA). Purely destructive or navigating-away-without-saving actions. Tertiary or infrequent links (use Ghost CTA).

**Reference**:
```
┌─────────────────┐   ┌──────────────────┐
│  Reaction Time  │   │ Color Perception  │   ← outlined, no fill
└─────────────────┘   └──────────────────┘
```

---

### Ghost CTA

**Category**: Input
**Used In**: Main Menu ("History")

**Description**: Lowest-weight interactive element — text only with an underline on hover. Used for infrequent or conditional actions that should be available but not visually competing with primary and secondary buttons.

**Specification**:
- Size: auto (text intrinsic width), height auto
- Background: none
- Border: none
- Label: `--neutral-400`, 14px Inter 400
- Hover state: text-decoration underline; color → `--neutral-200`
- Focus state: Focus Ring pattern (applied to a tight bounding box around the text)
- Press feedback: Button Press Confirmation pattern (opacity only — no scale; text elements should not scale on press)
- Disabled / hidden: removed from DOM entirely when the action is not available (e.g., History hidden when session count = 0)

**When to Use**: Optional, infrequent, or conditional destinations. Actions that would be discoverable rather than immediately obvious.

**When NOT to Use**: Actions that more than 20% of users are expected to take in a session. Never use for destructive actions.

---

### Digit Response Grid

**Category**: Input
**Used In**: HUD — Color Perception Module

**Description**: A row of 9 square buttons representing digits 1–9 (numbers visible in Ishihara plates), plus a full-width "I don't see a pattern" button below. This is the only multi-button input grid in the game — specific to the color perception module.

**Specification**:
- Digit buttons: 40×40px squares, 4px gap between buttons, 9 buttons in a single row
- Digit label: 14px JetBrains Mono 700, `--neutral-300`
- Digit button background: `--neutral-900`; border: 1px solid `--neutral-700`
- Digit hover: border → `--neutral-400`; label → `--signal-white`
- "No pattern" button: full width of the digit row, height 36px, text-only border (1px `--neutral-700`), 13px Inter 400, `--neutral-400`
- Keyboard mapping: digit key 1–9 directly activates corresponding button (no tab needed); Tab + Enter/Space activates "no pattern"
- Press feedback: Button Press Confirmation pattern
- Focus state: Focus Ring on individual button
- **Color contamination rule**: No chromatic colors anywhere in this component — achromatic only. Background, border, label all use neutral tokens.

**When to Use**: Only for this exact input pattern — a fixed set of numbered choices from a visual test plate.

**When NOT to Use**: Any other multiple-choice input. For general option selection, use a different pattern.

**Reference**:
```
[1] [2] [3] [4] [5] [6] [7] [8] [9]
[      I don't see a pattern       ]
```

---

### Any-Key Trigger

**Category**: Input
**Used In**: HUD — RT Module (trial response), HUD — Transition Card (skip), HUD — Pre-framing Card (continue)

**Description**: The entire viewport acts as an input surface — any key press or mouse click advances or triggers the current action. No specific button is needed. Used where the player should be completely focused on watching the screen rather than finding a specific key.

**Specification**:
- Input: any `keydown` event OR any `mousedown` / `touchstart` event
- The first event per trial/state is captured; subsequent events within the same trial are ignored
- No visible button or affordance during the active wait period (Waiting for stimulus state)
- For cards (Pre-framing, Transition): "Press any key to continue" or "Press any key to skip" hint shown in 12px `--neutral-text`, bottom-center — this is the Shortcut Hint Label pattern
- Focus: viewport-level listener, no focus element; screen reader users have a dedicated "Continue" button accessible via Tab

**When to Use**: When the input action is universal and the player should be focused on content rather than finding a control. Exclusively for test-state triggers and card dismissals.

**When NOT to Use**: Any menu or structured navigation. Never in contexts where the player might be typing or using keyboard shortcuts for other purposes.

---

### Focus Ring

**Category**: Input / Accessibility
**Used In**: All interactive elements

**Description**: The keyboard focus indicator applied to all interactive elements. The same ring is used everywhere — consistency makes it immediately recognizable.

**Specification**:
- Style: 2px solid outline
- Color: `--signal-white` (on dark backgrounds); `--neutral-900` (on light backgrounds, e.g., inside Primary CTA)
- Offset: 2px (between element edge and ring)
- Corner radius: matches the element's corner radius (2px for buttons, 0px for text links)
- Appear/disappear: instant — no animation
- Visibility: visible only on keyboard navigation (`:focus-visible` CSS selector — does not appear on mouse click)

**Accessibility requirement**: Every interactive element in the game must display this ring when keyboard-focused. No exceptions.

**When to Use**: Applied automatically to all buttons, links, and interactive controls via global CSS rule.

**When NOT to Use**: Never suppressed or overridden. If a custom element needs a different visual focus treatment, it must meet or exceed the contrast of this ring.

---

### App Wordmark

**Category**: Data Display
**Used In**: Main Menu

**Description**: The game's name displayed as a text-rendered logotype. Not an image — a typographic rendering that inherits accessibility benefits of text.

**Specification**:
- Content: "SYNAPTIC" — uppercase, tracked
- Font: 24px Inter 700, letter-spacing 0.15em
- Color: `--signal-white`
- Position: top-left of header zone
- Never localized — brand name is invariant across languages
- `aria-label="Synaptic"` (screen readers read uppercase letter-by-letter; label provides natural pronunciation)

**When to Use**: Top-level header of main menu only. Not repeated on other screens.

**When NOT to Use**: Within test modules or results screens — those screens don't need branding.

---

### Ambient Counter

**Category**: Data Display
**Used In**: Main Menu (session count), HUD (trial counter, plate counter)

**Description**: A quiet numeric readout that provides orientation context without demanding attention. Low contrast, small type, top corner. The player can glance at it; they don't need to read it constantly.

**Specification**:
- Content: "[N] sessions" (main menu) or "[N]/[total]" (HUD)
- Font: 12px JetBrains Mono 400
- Color: `--neutral-text` (minimum 4.5:1 contrast — AA, no higher)
- Position: top-right corner, 12px inset from edges
- Update behavior: instant text swap on value change — no animation
- `aria-live="polite"` with `aria-label` matching spoken form (e.g., "Trial 5 of 20")
- Conditional visibility: session counter hidden when session count = 0; shown from session 1 onward

**When to Use**: For orientation signals that the player benefits from knowing but doesn't need prominently. Progress through a sequence, historical context.

**When NOT to Use**: For data the player needs to act on (use Transient Stat Display or a dedicated readout with higher visual weight).

---

### Shortcut Hint Label

**Category**: Data Display
**Used In**: Main Menu (footer), Transition Card, Pre-framing Card

**Description**: Ultra-low-weight text that communicates an available keyboard shortcut. Present but barely visible — a discoverable affordance for power users, not a tutorial prompt.

**Specification**:
- Content: "Space or Enter to start" / "Press any key to skip" / "Press any key to continue"
- Font: 12px Inter 400
- Color: `--neutral-text` at 40% opacity (below standard contrast — intentional; this is supplementary info)
- Position: bottom-center of screen or card
- Never interactive — pure label
- Not localized for MVP (keyboard key names are language-invariant for Latin keyboard layouts)

**When to Use**: When a keyboard shortcut exists for a primary action and is worth surfacing without making it prominent.

**When NOT to Use**: For the only way to perform an action (that would be inaccessible). Always paired with a visible button alternative.

---

### Hairline Progress Bar

**Category**: Data Display
**Used In**: HUD (both modules)

**Description**: A 1-pixel horizontal bar at the very top edge of the viewport. Shows progress through the current module. Minimal enough to be subconscious — the player can register progress without consciously reading a meter.

**Specification**:
- Height: 1px exactly
- Width: 100% viewport width
- Position: top edge of viewport, z-index above all other elements
- Filled portion: `--neutral-mid`, left to right
- Unfilled portion: `--neutral-dark` (near-invisible)
- Update: instant jump on trial/plate completion — no lerp, no smooth fill
- Rationale for no lerp: smooth fill takes time to complete and introduces temporal elements into an environment where time perception must not be distorted

**When to Use**: As a lightweight progress signal during any module. One bar per screen maximum.

**When NOT to Use**: For data that needs to be read precisely (use Ambient Counter). On menu or results screens where progress concept doesn't apply.

---

### Transient Stat Display

**Category**: Feedback
**Used In**: HUD — RT Module (RT number + precision band)

**Description**: A large numeric readout that appears immediately on the input frame after a valid response, displays for a fixed duration, then fades. Designed to deliver measurement data with maximum impact and minimum permanence.

**Specification**:
- Primary value: 48px JetBrains Mono 700, `--signal-white`
- Secondary label (precision band): 11px Inter 400, `--neutral-text`, below primary value
- Position: center screen, below stimulus position — fixed coordinates that never shift between trials
- Appear: same render frame as input event — zero additional frames of delay
- Appear animation: scale 1.0→1.12→1.0 over 80ms (spatial pulse — not temporal rollup)
- Hold: 1500ms at full opacity
- Disappear: 200ms linear fade to 0 opacity, then removed from DOM
- **No counting animation** — value appears at its final measured value instantly
- Color modifier: fast responses (< 200ms) may use `--perf-fast` warm tint (token TBD — pending art director sign-off)
- `aria-live="assertive"` — screen reader announces value immediately

**When to Use**: For single measured values that should feel like instrument readouts — precise, immediate, and transient.

**When NOT to Use**: For values the player needs to refer back to (persist those in the results dashboard). For counts or progress (use Ambient Counter). Never with rollup/counting animations.

---

### Transient Warning Text

**Category**: Feedback
**Used In**: HUD — RT Module ("Too early", "No response")

**Description**: Short instructional or informational text that appears briefly in response to a trial event, then disappears. Visually distinct from normal data display — warnings use `--alert-amber`, informational notices use `--neutral-text`.

**Specification**:

**Warning variant** ("Too early"):
- Font: 16px Inter 500
- Color: `--alert-amber`
- Supplementary: 2px amber border flash on viewport edges, 200ms
- Lifetime: 500ms — cuts out instantly (no fade)
- `aria-live="assertive"`

**Informational variant** ("No response"):
- Font: 14px Inter 400
- Color: `--neutral-text` (deliberately low contrast — it's an observation, not an alert)
- No border flash
- Lifetime: 800ms — cuts out instantly
- `aria-live="polite"`

**Coach variant** (after 3 consecutive early responses):
- Appended below warning: "Wait for the circle to appear" — 12px Inter 400, `--neutral-text`
- Same lifetime as warning variant

**When to Use**: For trial-level events that require player acknowledgment or course correction. Always transient — never persistent.

**When NOT to Use**: For errors that require player action to resolve (those need a persistent notice). For positive feedback (use Transient Stat Display).

---

### Screen Slide Transition

**Category**: Navigation
**Used In**: All screen transitions in the game

**Description**: A horizontal slide animation that communicates navigation depth. Deeper screens slide in from the right; returning to shallower screens slides back from the left.

**Specification**:
- Duration: 180ms
- Easing: ease-out (decelerates into the resting position)
- Direction: navigating deeper → new screen slides in from right; navigating back → new screen slides in from left
- Scope: full-viewport transition — both incoming and outgoing screens participate
- Reduced motion: replaced with instant cut (0ms) when `prefers-reduced-motion: reduce` is active

**When to Use**: All navigation between top-level screens (Main Menu → Test Module → Results, etc.).

**When NOT to Use**: Between states within the same screen (use opacity change or content swap). For overlay cards that appear within a screen (use fade).

---

### Button Press Confirmation

**Category**: Feedback
**Used In**: All buttons (Primary CTA, Module Select, Ghost CTA, Digit Response Grid)

**Description**: The immediate visual feedback that confirms a button was pressed. Fires on `mousedown`/`touchstart` — not `mouseup` — for zero-delay feel.

**Specification**:
- Trigger: `mousedown` or `touchstart` (not `click` or `mouseup`)
- Effect: scale transform 1.0 → 0.95
- Duration: 80ms, ease-out
- Return: scale returns to 1.0 on `mouseup`/`touchend`, 60ms ease-out
- Reduced motion: replaced with opacity 1.0 → 0.7 → 1.0 at same timing
- **Exception**: Ghost CTA uses opacity change (0.7) instead of scale — text elements should not scale on press

**When to Use**: All interactive elements that the player activates by pressing. Universal across the UI.

**When NOT to Use**: Elements that are not interactive. Read-only counters or labels.

---

### Informational Overlay Card

**Category**: Overlay
**Used In**: HUD — Pre-framing Card (before color module), HUD — Transition Card (between modules)

**Description**: A full-viewport card that temporarily replaces the test environment with instructional or transitional content. Not a modal — there is no overlay behind it; it IS the entire screen content for its duration.

**Specification**:
- Background: same dark canvas as the test environment — no panel border, no lightbox effect
- Content layout: centered vertically and horizontally
- Title: 24px Inter 600, `--signal-white`
- Body text: 14px Inter 400, `--signal-white`, max-width 480px, line-height 1.6
- Dismissal hint: Shortcut Hint Label pattern ("Press any key to continue / skip") at bottom-center
- Enter animation: 180ms fade in from 0 to full opacity (no slide — the card is not a "deeper" screen)
- Exit animation: 180ms fade out OR immediate cut on keypress dismissal
- Screen reader: entire card content readable sequentially; `aria-live="polite"` on the card region

**Variants**:
- **Pre-framing**: Single block of text, dismissed by player. No countdown.
- **Transition**: Title + description + Countdown Display. Auto-advances when countdown reaches 0, or dismissed early by player.

**When to Use**: Between significant mode changes that benefit from a moment of cognitive preparation (test module transitions, mode explanations).

**When NOT to Use**: Error messages (use banner or inline notice). Confirmations (use a different modal). Tutorial or onboarding steps beyond the first-launch flow.

---

### Countdown Display

**Category**: Data Display
**Used In**: HUD — Transition Card

**Description**: A large monospaced number that counts down from 3 to 1 to signal an imminent state change. Paired exclusively with the Informational Overlay Card — Transition variant.

**Specification**:
- Content: integer countdown value (3, 2, 1)
- Font: 48px JetBrains Mono 700, `--signal-white`
- Position: center-bottom of the card (below description text, above shortcut hint)
- Update: number changes on 1-second interval — instant swap, no animation on the number itself
- Audio: soft click on each number change (same SFX as module start sound)
- `aria-live="polite"` — each number change announced

**When to Use**: Only in the Transition Card context. Announces an automatic state change the player cannot prevent (but can skip).

**When NOT to Use**: In menus or results screens. For any countdown that players must act on urgently (the test itself uses stimulus appearance, not a visible countdown).

---

## Animation Standards

Consolidated reference for all timing and easing values used across patterns. Implementers must use these values — do not introduce custom durations.

| Animation | Duration | Easing | Trigger | Notes |
|---|---|---|---|---|
| Screen slide (enter/exit) | 180ms | ease-out | Screen navigation | Replaced by instant cut on `prefers-reduced-motion` |
| Overlay card fade in | 180ms | linear | Card appear | |
| Overlay card fade out | 180ms | linear | Card dismiss or keypress | |
| Button press scale down | 80ms | ease-out | `mousedown` / `touchstart` | 1.0 → 0.95 |
| Button press scale up | 60ms | ease-out | `mouseup` / `touchend` | 0.95 → 1.0 |
| Button press opacity (reduced motion) | 80ms | ease-out | `mousedown` | 1.0 → 0.7 → 1.0 |
| Transient stat appear (scale pulse) | 80ms | ease-out | Input frame | 1.0 → 1.12 → 1.0 |
| Transient stat disappear (fade) | 200ms | linear | After hold duration | |
| Transient stat hold duration | 1500ms | — | After appear | No animation; just timer |
| "Too early" warning lifetime | 500ms | — | Display duration, instant cut | No fade |
| "No response" text lifetime | 800ms | — | Display duration, instant cut | No fade |
| Welcome-back banner fade in | 200ms | linear | App launch (30+ day gap) | |
| Welcome-back banner fade out | 200ms | linear | After 3000ms auto-dismiss | |
| Cyan ring expansion (feedback) | 120ms | ease-out | Valid response (RT module) | Ring created then removed |
| Amber border flash lifetime | 200ms | ease-out | Early response | 2px viewport edge flash |

**Global rule**: No animation may introduce perceptual delay between a player action and its confirmation. All feedback animations fire in the same render frame as the input event.

**Reduced motion**: Any positional animation (slide, scale) is replaced with an opacity-only alternative or an instant cut. Duration-based holds (warning text, stat display) are unchanged under reduced motion.

---

## Sound Standards

Audio events associated with interaction patterns. Full SFX specifications live in the Feedback System GDD (`design/gdd/feedback-system.md`). This table maps patterns to their audio events.

| Pattern | Audio Event | Trigger | Notes |
|---|---|---|---|
| Button Press Confirmation | None | Button press | Buttons are silent — only test-level events have audio |
| Transient Stat Display | `snd_response_pop` | Valid RT response — fires with the visual | Crisp, dry pop; pitch fixed (no variation for MVP) |
| Transient Warning Text ("Too early") | `snd_early_response` | Early response | Low tone; clearly distinct from `snd_response_pop` |
| Transient Warning Text ("No response") | None | Timeout | Silence — a missed trial is a non-event |
| Countdown Display | `snd_countdown_tick` | Each number change in countdown | Soft click; same as module-start sound |
| Any-Key Trigger (module start) | `snd_module_start` | When countdown reaches 0 or player skips | Soft chime-click |
| Informational Overlay Card (appear) | None | Card fade in | No audio on card entry |

**Global rules**:
- No audio during the Waiting state (before stimulus appears) — silence is a controlled test condition
- No audio during Color Perception plate display — only neutral feedback (click + opacity)
- No music during any test module — per game design Pillar 2

---

## Gaps & Patterns Needed

The following screens and interactions are planned but not yet specced. When their UX specs are authored, these patterns may need to be added:

| Future Screen | Anticipated New Patterns |
|---|---|
| Results Dashboard | Stat card (module score + label), Trend sparkline, Performance band indicator, Export button |
| History View | Session list item, Trend line chart, Date group header |
| Results — CVD disclosure | Three-step emotional scaffolding card (specialized Informational Overlay variant) |
| Onboarding welcome screen | Full-screen welcome card (simpler variant of Informational Overlay Card) |

**Consistency watch**: The Transient Stat Display pattern (48px JetBrains Mono) is used for in-test RT display. If the Results Dashboard also uses large numeric displays for final scores, those should use the same type size and family for consistency — but they are persistent (not transient) and require a different pattern entry.

---

## Open Questions

- **`--perf-fast` token**: The Transient Stat Display specifies a warm tint for sub-200ms reactions. Exact hex value not yet in the art bible. Needs art director sign-off. Constraint: must read as achromatic enough to not be perceived as a chromatic test element when the player transitions from the RT module.
- **"No response" contrast**: Intentionally below WCAG AA. Requires accessibility review and documented rationale before release.
- **Digit Response Grid touch targets on small viewports**: 40×40px buttons at 4px gap require minimum ~400px width to display without wrapping. Mobile is out of scope for MVP, but this should be revisited if mobile support is added.
