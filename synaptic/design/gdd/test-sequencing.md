# Test Sequencing

> **Status**: Designed
> **Author**: game-designer
> **Last Updated**: 2026-04-12
> **Implements Pillar**: Pillar 2 (Every Tap Feels Crisp), Pillar 3 (Reveal Through Repetition)

## Summary

The test sequencing system orchestrates test modules into batteries, manages module
selection and ordering, handles the onboarding flow for first-time users, and
provides the overall session structure. It is the conductor — individual test
modules are the instruments.

> **Quick reference** — Layer: `Feature` · Priority: `MVP` · Key deps: `Stimulus-Response Engine, Color Perception System`

## Overview

The test sequencing system controls what happens between test modules: which module
runs next, how modules are grouped into batteries, what the player sees before and
after each module, and how the first-time experience differs from returning sessions.
At MVP, sequencing is simple — a fixed two-module battery (reaction time → color
perception) with brief inter-module transitions. The player can also select
individual modules from a menu for targeted retesting. This system exists because
even a two-module game needs structured flow — the onboarding experience, the
transition between tests, and the session start/end are all moments that shape the
player's perception of the product's quality. Pillar 2 demands that every transition
feels deliberate, not accidental.

## Player Fantasy

The experience flows. You launch the app, and within 5 seconds you understand what
to do. The first test starts with a single instruction — "Click when the circle
appears" — and you're measuring your reflexes. After the reaction time test, a
brief transition card prepares you for the color perception test. After both tests,
your complete results dashboard appears. Nothing was confusing, nothing required
reading a manual, and the entire battery took 4 minutes. The fantasy is **effortless
flow** — the feeling that the game is guiding you through a well-designed protocol
without friction.

## Detailed Design

### Core Rules

1. **Battery**: A battery is an ordered sequence of test modules. At MVP, there is
   one battery type: "Standard Battery" = [Reaction Time, Color Perception].
2. **Module selection**: From the main menu, the player can either:
   - Start a full battery (runs all modules in order)
   - Select an individual module for targeted retesting
3. **Module ordering within a battery**: Fixed at MVP. Reaction Time always runs
   first (it's simpler, serving as warm-up). Color Perception runs second (requires
   more cognitive focus, benefits from the player being settled in).
4. **Inter-module transition**: Between modules in a battery, show a transition
   card for `transition_duration_ms` (3000ms):
   - Module name and brief description (1 sentence)
   - "Starting in 3... 2... 1..." countdown
   - Player can skip the countdown by pressing any key
5. **First-time onboarding** (detected by Session Persistence — session count = 0):
   - Welcome screen: "Synaptic measures your reaction speed and color perception.
     Each test takes about 2 minutes."
   - Before first RT module: instruction card: "A white circle will appear after
     a random delay. Click or press any key as fast as you can when you see it."
   - Before first color module: instruction card: "You'll see a circle of colored
     dots. Some contain a hidden number. Identify the number, or tap 'No pattern'
     if you can't see one."
   - After first complete battery: brief explanation of results: "Your results
     show your reaction speed percentile and color perception screening. Come back
     for more sessions to increase accuracy."
6. **Returning user flow** (session count ≥ 1):
   - Main menu shows: "Start Battery" and individual module buttons
   - No instruction cards (player already knows the protocol)
   - Quick-start: pressing Space or Enter from main menu starts the standard battery
7. **Session boundaries**: A session starts when the player begins any module and
   ends when they navigate away from the results screen (or close the app). One
   session can contain multiple batteries or individual modules.
8. **Module completion triggers save**: When any module completes, the Test
   Sequencing system signals Session Persistence to save. It does not wait for the
   full battery to finish.

### States and Transitions

| State | Entry Condition | Exit Condition | Behavior |
|-------|----------------|----------------|----------|
| `MAIN_MENU` | App launch or return from results | User selects battery or module | Show menu options. Check session count for onboarding vs. returning UI. |
| `ONBOARDING` | First launch (session count = 0), user selects "Start" | Onboarding cards complete | Show welcome card → instruction cards → transition to first module. |
| `PRE_MODULE` | Battery advancing to next module, or individual module selected | Transition countdown completes or user skips | Show transition card with module name and countdown. |
| `MODULE_ACTIVE` | Countdown complete | Module signals `module_completed` | Delegate control to the active test module (SRE or CPS). Sequencing is passive. |
| `POST_MODULE` | Module completes | Transition to next module or results | Save data. If more modules in battery → PRE_MODULE. If battery complete → RESULTS. |
| `RESULTS` | Battery complete or individual module complete | User navigates (new battery, menu, or exit) | Hand off to Results & Interpretation system. Wait for user navigation. |

### Screen Flow

```
App Launch
  ├─ First time? → ONBOARDING → Welcome → RT Instruction → PRE_MODULE(RT)
  └─ Returning?  → MAIN_MENU
                     ├─ "Start Battery" → PRE_MODULE(RT) → MODULE(RT) → POST_MODULE
                     │                    → PRE_MODULE(Color) → MODULE(Color) → POST_MODULE
                     │                    → RESULTS
                     ├─ "Reaction Time"  → PRE_MODULE(RT) → MODULE(RT) → RESULTS
                     ├─ "Color Perception" → PRE_MODULE(Color) → MODULE(Color) → RESULTS
                     └─ "History"        → Historical view (Session Persistence + R&I)
```

### Interactions with Other Systems

| System | Direction | Interface |
|--------|-----------|-----------|
| **Stimulus-Response Engine** | Downstream (orchestrates) | Calls `start_module(config)` with RT-specific configuration. Listens for `module_completed`. |
| **Color Perception System** | Downstream (orchestrates) | Calls `start_module(config)` with color-specific configuration. Listens for `module_completed`. |
| **Session Persistence** | Downstream (triggers save) | Signals `save_session_data()` after each module completion. Reads session count for onboarding detection. |
| **Results & Interpretation** | Downstream (triggers display) | Signals `show_results(module_type)` after module or battery completion. |

## Formulas

No complex formulas in this system. Key timing values:

| Parameter | Value | Description |
|-----------|-------|-------------|
| `transition_duration_ms` | 3000 | Inter-module transition card display time |
| `countdown_steps` | 3 | "3... 2... 1..." countdown ticks |
| `onboarding_card_duration_ms` | ∞ (until input) | Instruction cards wait for user to proceed |
| `welcome_auto_advance_ms` | 8000 | Welcome screen auto-advances after 8s (or on input) |

## Edge Cases

| Scenario | Expected Behavior | Rationale |
|----------|------------------|-----------|
| Player starts a battery, completes RT, then closes app before color test | RT results are saved (save fires after each module). Color module not started — no partial color data. Next launch shows main menu, not mid-battery state. | Module-level saves prevent data loss. No mid-battery resume at MVP — complexity not justified. |
| Player selects individual module repeatedly | Each individual module run is a separate session record entry. Results show per-run data. Trend tracks across all sessions. | Targeted retesting is a core use case. Each run produces valid data. |
| Player skips onboarding (impossible at MVP — but future concern) | Onboarding is mandatory on first launch. No skip button. Cards advance on input, so the minimum path is: read card → press → read card → press → start test. | First impressions matter. 10 seconds of instruction prevents confusion. |
| Player presses key during transition countdown | Countdown skips to completion. Module starts immediately. | Respect impatient users. The countdown is courtesy, not requirement. |
| App launched offline (desktop export) | No difference — no network features. Everything works locally. | By design, no online dependency. |
| Session Persistence reports storage error | Sequencing continues normally. Warning banner shown but flow is not interrupted. | Storage errors should not break the testing experience. |
| Player returns after a long absence (> 30 days) | Welcome-back message: "Welcome back! Your [N] previous sessions are still here." Main menu as usual. | Acknowledge the return without re-onboarding. |
| Player's first session has all timeouts | Results show "Insufficient data." Offer: "Try again?" button leading back to the same module. | Don't send the player to the menu after a failed attempt — let them retry immediately. |

## Dependencies

| System | Direction | Nature of Dependency |
|--------|-----------|---------------------|
| Stimulus-Response Engine | This depends on SRE | Orchestrates RT module. Cannot run reaction time tests without it. Hard dependency. |
| Color Perception System | This depends on CPS | Orchestrates color module. Cannot run color tests without it. Hard dependency. |
| Session Persistence | This depends on SP (soft) | Reads session count for onboarding detection. Can function without it (always shows onboarding). Soft dependency. |
| Results & Interpretation | Depended on by R&I (soft) | R&I can trigger "Start new module" through this system. Soft dependency. |

## Tuning Knobs

| Parameter | Current Value | Safe Range | Effect of Increase | Effect of Decrease |
|-----------|--------------|------------|-------------------|-------------------|
| `transition_duration_ms` | 3000 | 1500–5000 | Longer transition; more preparation time, can feel slow | Shorter transition; faster pacing, less mental preparation |
| `countdown_steps` | 3 | 2–5 | More countdown ticks; longer buildup | Fewer ticks; faster start |
| `welcome_auto_advance_ms` | 8000 | 5000–15000 | Longer welcome display; more reading time | Shorter; faster onboarding for quick readers |
| `battery_modules` | ["reaction_time", "color_perception"] | MVP: fixed | N/A at MVP | N/A at MVP |
| `return_absence_threshold_days` | 30 | 7–90 | Longer absence before welcome-back message | Shorter; more frequent welcome-back messages |

## Visual/Audio Requirements

| Event | Visual Feedback | Audio Feedback | Priority |
|-------|----------------|---------------|----------|
| Main menu display | Module buttons in vertical stack. "Start Battery" primary CTA (2px radius, `--signal-white`). Individual modules secondary. | None. | High |
| Transition card | Module name in 24px Inter 600, center. Description in 14px Inter, below. Countdown in 48px JetBrains Mono, center-bottom. | Countdown ticks: soft click per number (same as module start click). | Medium |
| Onboarding instruction card | Instruction text in 16px Inter, center. "Press any key to continue" in 12px, `--neutral-text`, bottom. | None. | High |
| Welcome screen | "Synaptic" in 36px Inter 600, center. Tagline in 14px Inter, below. | None. | Medium |
| Screen transitions | 180ms horizontal slide per art bible. Deeper screens slide right. | None. | Medium |

## UI Requirements

| Information | Display Location | Update Frequency | Condition |
|-------------|-----------------|-----------------|-----------|
| "Start Battery" button | Center screen, main menu | Persistent | On main menu |
| Individual module buttons | Below battery button | Persistent | On main menu |
| "History" button | Below module buttons | Persistent | When session count ≥ 1 |
| Session count | Top-right of main menu: "[N] sessions" | On menu display | When session count ≥ 1 |
| Transition countdown | Center-bottom of transition card | Per second | During PRE_MODULE |
| Instruction text | Center of onboarding cards | Per card | During ONBOARDING |

## Cross-References

| This Document References | Target GDD | Specific Element Referenced | Nature |
|--------------------------|-----------|----------------------------|--------|
| Orchestrates RT module lifecycle | `design/gdd/stimulus-response-engine.md` | `start_module(config)` and `module_completed` interface | State trigger |
| Orchestrates color module lifecycle | `design/gdd/color-perception-system.md` | `start_module(config)` and `module_completed` interface | State trigger |
| Reads session count for onboarding | `design/gdd/session-persistence.md` | Session count, first-launch detection | Data dependency |
| Triggers results display | `design/gdd/results-interpretation.md` | `show_results(module_type)` interface | State trigger |

## Acceptance Criteria

- [ ] GIVEN first app launch (session count = 0), WHEN the app loads, THEN onboarding flow is shown: welcome card → RT instruction → first module
- [ ] GIVEN a returning user (session count ≥ 1), WHEN the app loads, THEN main menu is shown with battery start and individual module options
- [ ] GIVEN a full battery is started, WHEN RT module completes, THEN a 3-second transition card appears before the color perception module begins
- [ ] GIVEN a transition countdown is active, WHEN the player presses any key, THEN the countdown skips and the next module starts immediately
- [ ] GIVEN an individual module is selected, WHEN it completes, THEN results are shown for that module only (no transition to another module)
- [ ] GIVEN a module completes during a battery, WHEN save is triggered, THEN data is persisted before the next module begins
- [ ] GIVEN a player returns after 30+ days, WHEN the app loads, THEN a welcome-back message is shown
- [ ] GIVEN a session where all trials timeout, WHEN results are shown, THEN a "Try again?" option is available on the results screen
- [ ] GIVEN pressing Space or Enter on the main menu, WHEN handled, THEN the standard battery starts (quick-start shortcut)
- [ ] No hardcoded module order or transition timings — all values from configuration

## Open Questions

| Question | Owner | Deadline | Resolution |
|----------|-------|----------|-----------|
| Should the battery order be randomizable to prevent order effects on results? | Game Designer | Before Vertical Slice | Fixed order at MVP is simpler. Randomization may affect RT via warm-up effects. |
| Should there be a "practice mode" that doesn't save results? | Game Designer | Sprint 2 | Would let players try without polluting their data. But Pillar 1 says all data is real data. |
| Should returning users see instruction cards if they haven't played in > 30 days? | UX Designer | Sprint 2 | Current: no re-onboarding. May need a "remind me" option. |
| How many modules trigger a "battery" vs. individual retests in session records? | Systems Designer | Sprint 1 | Need clear session record tagging for batteries vs. singles. |
