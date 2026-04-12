# UX Spec: Main Menu

> **Status**: Complete
> **Author**: ux-designer (autonomous)
> **Last Updated**: 2026-04-12
> **Journey Phase(s)**: Session Start — player arrives intentionally, ready to test
> **Template**: UX Spec

---

## Purpose & Player Need

The main menu is the launchpad — it must get the player into a test in under 5 seconds with zero ambiguity. The player arrives wanting one of two things: (a) run a full standard battery to get a complete cognitive snapshot, or (b) retest a specific module they want to improve or verify. The menu must make both paths equally obvious without clutter. If this screen is slow, confusing, or decorative, it violates Pillar 2 (Every Tap Feels Crisp) at the session's first frame.

---

## Player Context on Arrival

- **First-time players** are redirected through the onboarding flow (welcome card → RT instruction) before ever seeing this screen. By the time they reach the menu proper, they've completed at least one test.
- **Returning players** (session count ≥ 1) land here directly on app launch. They are in a state of calm, deliberate readiness — they opened the app because they wanted to test themselves.
- **Emotional state**: Composed, ready, slightly anticipatory. Not stressed. The design should reinforce calm focus — not excite or overwhelm.
- **Cognitive load**: Low. Player should not need to read anything or make a complex decision. The primary action (Start Battery) should be immediately obvious.

---

## Navigation Position

```
[App Root]
    └── Main Menu  ← this screen
            ├── → Onboarding (first launch only — one-way, does not return here mid-flow)
            ├── → Reaction Time Module
            ├── → Color Perception Module
            └── → History View (conditional — session count ≥ 1)
```

The main menu is the root of the returning-player experience. All test paths originate here and return here after results are dismissed.

---

## Entry & Exit Points

### Entry

| Entry Source | Trigger | Player carries this context |
|---|---|---|
| App launch (returning user) | Automatic — session count ≥ 1 detected | Session history in localStorage |
| Results screen ("Test Again") | Player taps back-to-menu CTA | Previous session data (already saved) |
| Results screen ("Done") | Player dismisses results | Previous session data (already saved) |

### Exit

| Exit Destination | Trigger | Notes |
|---|---|---|
| Standard Battery (RT → Color) | "Start Battery" button, or Space/Enter keypress | Begins full test sequence — no module selection |
| Reaction Time Module only | "Reaction Time" individual module button | Single module run — results shown after completion |
| Color Perception Module only | "Color Perception" individual module button | Single module run — results shown after completion |
| History View | "History" button | Conditional — only shown when session count ≥ 1 |

---

## Layout Specification

### Information Hierarchy

Priority order (what player must see first):
1. **Start Battery CTA** — the single most important action; 80% of players want this
2. **App name / wordmark** — orientation anchor, minimal; confirms they're in the right place
3. **Individual module buttons** — secondary path for targeted retesting
4. **History button** — tertiary; only relevant after session 1
5. **Session count** — ambient context; a quiet signal of investment

### Layout Zones

```
┌─────────────────────────────────────────┐
│  SYNAPTIC          [N sessions]         │  ← HEADER ZONE (10% height)
├─────────────────────────────────────────┤
│                                         │
│                                         │
│          [  START BATTERY  ]            │  ← PRIMARY ACTION ZONE (40% height)
│                                         │
│                                         │
├─────────────────────────────────────────┤
│     [ Reaction Time ]  [ Color ]        │  ← MODULE SELECT ZONE (25% height)
├─────────────────────────────────────────┤
│              [ History ]                │  ← HISTORY ZONE (15% height, conditional)
├─────────────────────────────────────────┤
│         Space or Enter to start         │  ← FOOTER HINT (10% height)
└─────────────────────────────────────────┘
```

**Zone rationale**:
- "Start Battery" is center-stage; vertical centering pulls player eye to it immediately on any screen size
- Module buttons are visually subordinate (smaller, outlined rather than filled) but spatially adjacent — easy to find
- History is below the fold of immediate attention — present but not competing
- Footer keyboard hint is ultra-low-contrast (12px, `--neutral-text`) — a discoverable affordance, not a demand

### Component Inventory

| Zone | Component | Type | Interactive | Content | Pattern |
|---|---|---|---|---|---|
| Header | App wordmark | Text label | No | "SYNAPTIC" — 24px Inter 600, `--signal-white` | Pattern Library: App Wordmark |
| Header | Session counter | Text label | No | "[N] sessions" — 12px Inter 400, `--neutral-text` | Pattern Library: Ambient Counter |
| Primary | Start Battery button | Primary CTA button | Yes | "Start Battery" — 18px Inter 600, `--signal-white` fill | Pattern Library: Primary CTA Button |
| Module | Reaction Time button | Secondary module button | Yes | "Reaction Time" — 14px Inter 500, outlined | Pattern Library: Module Select Button |
| Module | Color Perception button | Secondary module button | Yes | "Color Perception" — 14px Inter 500, outlined | Pattern Library: Module Select Button |
| History | History button | Tertiary button | Yes | "History" — 14px Inter 400, text-only with underline on hover | Pattern Library: Ghost CTA |
| Footer | Keyboard shortcut hint | Text label | No | "Space or Enter to start" — 12px Inter 400, `--neutral-text`, 40% opacity | Pattern Library: Shortcut Hint Label |

### ASCII Wireframe

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  SYNAPTIC                              3 sessions   │
│  ─────────────────────────────────────────────────  │
│                                                     │
│                                                     │
│                                                     │
│               ┌──────────────────┐                  │
│               │   Start Battery  │                  │
│               └──────────────────┘                  │
│                                                     │
│                                                     │
│         ┌─────────────┐   ┌──────────────┐         │
│         │Reaction Time│   │Color Perc... │         │
│         └─────────────┘   └──────────────┘         │
│                                                     │
│                       History                       │
│                                                     │
│            Space or Enter to start                  │
│                                                     │
└─────────────────────────────────────────────────────┘

FIRST-TIME USER (session count = 0):
┌─────────────────────────────────────────────────────┐
│                                                     │
│  SYNAPTIC                                           │
│  ─────────────────────────────────────────────────  │
│                                                     │
│                                                     │
│               ┌──────────────────┐                  │
│               │   Start Battery  │                  │
│               └──────────────────┘                  │
│                                                     │
│         ┌─────────────┐   ┌──────────────┐         │
│         │Reaction Time│   │Color Perc... │         │
│         └─────────────┘   └──────────────┘         │
│                                                     │
│            Space or Enter to start                  │
│                                                     │
└─────────────────────────────────────────────────────┘
(History button absent — no sessions yet)
```

---

## States & Variants

| State / Variant | Trigger | What Changes |
|---|---|---|
| **First Visit** (session count = 0) | First launch after onboarding | History button absent; session counter absent |
| **Returning** (session count ≥ 1) | Any return visit | History button present; session counter visible |
| **Welcome Back** (30+ day gap) | Detected by session persistence | Brief welcome-back banner appears above Start Battery for 3s then auto-dismisses: "Welcome back — [N] sessions on record." |
| **Storage Warning** | localStorage > 80% capacity | Amber notice banner above footer: "Storage filling — export your data soon." |
| **Storage Unavailable** | localStorage not accessible (private browsing, quota exceeded) | Screen renders as First Visit state — session counter absent, History button absent. A subtle inline notice appears below the module buttons: "Session data won't be saved in this browser mode." (`--neutral-text`, 12px, no `--alert-amber` — informational not alarming). All test modules remain fully functional; data is held in memory for the session only. |

---

## Interaction Map

Input methods: Keyboard/Mouse primary, Gamepad none, Touch partial (web responsive).

| Component | Action | Input | Immediate Feedback | Outcome |
|---|---|---|---|---|
| Start Battery button | Click / tap | Mouse click, touch tap | Button scales 0.95, 80ms; 180ms horizontal slide transition right | Starts Standard Battery sequence |
| Start Battery shortcut | Press | Space or Enter key | Same as button press | Starts Standard Battery sequence |
| Reaction Time button | Click / tap | Mouse click, touch tap | Button scales 0.95, 80ms; 180ms slide transition right | Starts RT module only |
| Color Perception button | Click / tap | Mouse click, touch tap | Button scales 0.95, 80ms; 180ms slide transition right | Starts Color module only |
| History button | Click / tap | Mouse click, touch tap | Text link underline; 180ms slide transition right | Opens History View |
| All buttons | Hover | Mouse hover | Border shifts from `--neutral-600` to `--signal-white`; no position change | No state change — affordance only |
| Keyboard navigation | Tab | Tab key | Focus ring appears on element (2px `--signal-white` outline, 2px offset) | Focus moves to next interactive element |

**Tab order**: Start Battery → Reaction Time → Color Perception → History (conditional)

---

## Events Fired

| Player Action | Event Fired | Payload / Data |
|---|---|---|
| Start Battery pressed | `battery_started` | `{ source: "main_menu", session_count: N }` |
| Reaction Time module selected | `module_selected` | `{ module: "reaction_time", source: "main_menu" }` |
| Color Perception module selected | `module_selected` | `{ module: "color_perception", source: "main_menu" }` |
| History opened | `history_viewed` | `{ session_count: N }` |
| Space/Enter shortcut used | `battery_started` | `{ source: "keyboard_shortcut", session_count: N }` |
| App launched (returning) | `session_started` | `{ session_count: N, days_since_last: D }` |

All events modify non-persistent state only (analytics). No persistent game state is modified on main menu — the menu is read-only with respect to save data.

---

## Transitions & Animations

| Transition | Spec |
|---|---|
| **Screen enter** (arriving at main menu) | 180ms horizontal slide from left (coming back from results) or instant on first launch post-onboarding |
| **Screen exit** (going to a test) | 180ms horizontal slide to right — deeper screens slide right per art bible |
| **Welcome-back banner enter** | Fade in over 200ms, auto-dismiss after 3000ms with 200ms fade out |
| **Storage warning banner** | Fade in over 200ms, persists until dismissed or storage drops below threshold |
| **Button press feedback** | Scale 0.95, duration 80ms, ease-out; immediate on `mousedown`/`touchstart` — not on `mouseup` |
| **Focus ring** | Instant appear/disappear on focus change — no animation |

**Motion sensitivity note**: The 180ms slide is the only positional animation. It must respect OS/browser reduced-motion preference — substitute with instant cut if `prefers-reduced-motion: reduce` is active.

---

## Data Requirements

| Data | Source System | Read / Write | Notes |
|---|---|---|---|
| Session count | Session Persistence | Read | Drives History button visibility and session counter display |
| Days since last session | Session Persistence | Read | Drives welcome-back banner |
| Storage usage % | Session Persistence | Read | Drives storage warning banner |
| Last session date | Session Persistence | Read | Used to compute days-since-last |

This screen is **read-only** — it reads session metadata but writes nothing. All writes occur inside test modules and results screens.

---

## Accessibility

**Keyboard navigation**: Full keyboard path through all interactive elements via Tab. Space/Enter activates all buttons. Tab order follows visual top-to-bottom, left-to-right hierarchy.

**Focus indicators**: 2px `--signal-white` outline, 2px offset on all interactive elements. Visible against the dark background at minimum 3:1 contrast ratio.

**Color independence**: No information is conveyed by color alone. The session counter, History button visibility, and warning banners all use text labels that communicate the same meaning as any color coding.

**Text contrast**: All text on the dark background (`#0a0a0a` base) meets WCAG AA minimum (4.5:1 for normal text, 3:1 for large text):
- "SYNAPTIC" wordmark: `--signal-white` on dark — ~20:1
- Button labels: `--signal-white` on dark — ~20:1
- Neutral text elements (session counter, footer hint): `--neutral-text` at ~4.5:1 minimum

**Screen readers**: All buttons have visible labels (no icon-only buttons). The session counter should have `aria-label="N sessions recorded"`. Conditional elements (History button) are removed from the DOM when not applicable — not hidden with `display:none` while remaining in tab order.

**Reduced motion**: Slide transition replaced with instant cut when `prefers-reduced-motion: reduce` is detected. Button scale feedback replaced with opacity change (0.7 opacity, 80ms).

---

## Localization Considerations

| Element | Character limit | Risk |
|---|---|---|
| "Start Battery" | ~18 chars English — allow up to 30 | HIGH: German "Standardbatterie starten" = 26 chars; button must accommodate |
| "Reaction Time" | ~13 chars — allow up to 22 | MEDIUM |
| "Color Perception" | ~16 chars — allow up to 26 | MEDIUM: French "Perception des couleurs" = 24 chars |
| "History" | ~7 chars — allow up to 15 | LOW |
| "SYNAPTIC" | Brand name — never localize | N/A |
| Session counter "[N] sessions" | Variable — allow "sessions" up to 15 chars | MEDIUM: German "Sitzungen" = 9 chars, safe |
| Footer hint "Space or Enter to start" | ~24 chars — allow up to 40 | MEDIUM |

Button widths should be min-content with max-width cap and text wrapping allowed on secondary buttons. Primary CTA must not wrap.

---

## Acceptance Criteria

- [ ] Screen opens within 200ms of app launch for returning user (session count ≥ 1)
- [ ] "Start Battery" button is visible without scrolling on all viewport sizes ≥ 360px wide
- [ ] Pressing Space or Enter on the main menu starts the standard battery immediately (same outcome as clicking "Start Battery")
- [ ] History button is absent when session count = 0; appears when session count ≥ 1
- [ ] Session counter displays correctly formatted "[N] sessions" in header when session count ≥ 1
- [ ] All interactive elements are reachable via Tab key in correct order: Start Battery → Reaction Time → Color Perception → History (if shown)
- [ ] All interactive elements have visible focus rings (2px outline) when keyboard-focused
- [ ] Welcome-back banner appears when last session was ≥ 30 days ago and auto-dismisses after 3 seconds
- [ ] Storage warning banner appears when localStorage usage exceeds 80% of allocated capacity
- [ ] Screen slide-in transition is replaced with instant cut when `prefers-reduced-motion: reduce` is active
- [ ] "Start Battery" button text is not truncated or wrapped at any supported viewport width
- [ ] Screen is read-only — no save data is written on this screen

---

## Open Questions

- None — all design decisions resolved for MVP scope.
