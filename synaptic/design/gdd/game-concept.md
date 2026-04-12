# Game Concept: Synaptic

*Created: 2026-04-12*
*Status: Draft — Revised after design review (2026-04-12)*

---

## Elevator Pitch

> It's a minimalist reflex and perception testing game where you react to
> precisely-timed stimuli across multiple test batteries to discover whether
> your reaction speed is normal, fast, or slow — and whether you're colorblind.

---

## Core Identity

| Aspect | Detail |
| ---- | ---- |
| **Genre** | Cognitive Testing / Reflex Game |
| **Platform** | Web (Browser) + PC Desktop |
| **Target Audience** | Curiosity-driven adults who want to measure their cognitive performance (see Player Profile) |
| **Player Count** | Single-player |
| **Session Length** | 3-10 minutes |
| **Monetization** | Free (premium: none planned for MVP) |
| **Estimated Scope** | Small (12-16 weeks full vision, solo) |
| **Comparable Titles** | Human Benchmark, Cambridge Brain Sciences, Stroop Test apps |

---

## Core Fantasy

You are the subject and the scientist. Every tap generates real data about your
brain. The game doesn't guess — it measures. Over multiple sessions, a picture
of your cognitive abilities emerges: your reaction time percentile against
population norms, your color perception accuracy, and whether you show signs
of color vision deficiency.

The fantasy is **self-knowledge through precision** — the satisfaction of seeing
a number that means something real about you, and watching it change over time.

---

## Unique Hook

Like Human Benchmark, AND ALSO it integrates Ishihara-style colorblind
screening into the test battery, interprets your results with anchored
language (not just raw numbers), tracks trends across sessions with
browser-calibrated statistics, and wraps the entire experience in
satisfying game juice that makes clinical screening feel like play.

---

## Visual Identity Anchor

**Direction: "Clinical Precision"**

- **Visual Rule**: The interface is the instrument — every pixel serves measurement or feedback.
- **Mood & Atmosphere**: Clean, focused, premium. Like a well-designed medical device or a high-end data dashboard. Dark background creates a controlled testing environment.
- **Shape Language**: Geometric, grid-aligned. Circles for stimuli (clean hit targets), rectangles for data display. No organic shapes — everything is deliberate.
- **Color Philosophy**: Neutral dark canvas (near-black) so test colors appear uncontaminated by UI chrome. Color is a *test variable*, not decoration — the UI itself uses only white, gray, and subtle accent colors. Test stimuli use calibrated, scientifically-meaningful colors (especially for colorblind detection plates).

**Design Tests**:
- "Should we add a colorful background theme?" → No. Color is a controlled variable in this game. Background stays neutral.
- "Should buttons have gradient fills?" → No. Flat, high-contrast. The instrument aesthetic demands clarity over decoration.

---

## Player Experience Analysis (MDA Framework)

### Target Aesthetics (What the player FEELS)

| Aesthetic | Priority | How We Deliver It |
| ---- | ---- | ---- |
| **Sensation** (sensory pleasure) | 1 | Crisp audio pops, screen flash on reaction, precise number display — the tap-measure-see cycle is intrinsically satisfying |
| **Fantasy** (make-believe, role-playing) | N/A | Not applicable — the fantasy IS reality (self-measurement) |
| **Narrative** (drama, story arc) | N/A | No narrative elements |
| **Challenge** (obstacle course, mastery) | 3 | Consistency improvement over sessions, but no failure state — challenge is self-referential, not gated |
| **Fellowship** (social connection) | N/A | Single-player self-assessment |
| **Discovery** (exploration, secrets) | 2 | Discovering your own cognitive profile — circadian patterns, improvement curves, color perception insights |
| **Expression** (self-expression, creativity) | N/A | Not applicable |
| **Submission** (relaxation, comfort zone) | 4 | Daily testing ritual, meditative focus state during tests |

### Key Dynamics (Emergent player behaviors)

- Players will develop pre-test rituals (deep breath, find focus) to optimize scores
- Players will run specific modules repeatedly to isolate weak areas
- Players will compare morning vs. evening scores to discover their own circadian patterns
- Players will share colorblind detection results with friends/family who suspect CVD

### Core Mechanics (Systems we build)

1. **Stimulus-Response Engine** — precisely-timed visual stimuli with ~5ms input measurement precision (browser floor). Measurement displayed with ±5ms uncertainty band. Input captured on `keydown`/`mousedown` within the same animation frame.
2. **Ishihara Color Plate System** — curated set of 24 standardized plates (not procedurally generated) with controlled color relationships. Dot placement randomized per presentation, but color pairings are fixed and clinically validated. Positioned as a "screening indicator," not a clinical diagnosis.
3. **Statistical Analysis Engine** — percentile calculation against browser-calibrated norms (lab norms adjusted with a +15ms systematic offset correction for browser measurement overhead). Confidence intervals and trend tracking.
4. **Interpretive Results Layer** — translates raw data into anchored language ("About average — faster than most casual users, slower than trained gamers"). Uses labeled spectrum (Slow / Average / Fast / Elite) as primary display; raw ms as secondary detail disclosure.
5. **Session History & Trends** — persistent local storage with one-click JSON export at MVP to prevent data loss. Rolling trend averages as primary view (not single-session verdicts).
6. **Feedback Juice System** — audio/visual feedback that fires on the input frame (same-frame acknowledgment). Number display is instantaneous (no rollup animation that would distort time perception). Juice is spatial (ring expansion, scale pulse) not temporal.

---

## Player Motivation Profile

### Primary Psychological Needs Served

| Need | How This Game Satisfies It | Strength |
| ---- | ---- | ---- |
| **Autonomy** (freedom, meaningful choice) | Choose which tests to run, when to retest, which modules to focus on | Supporting |
| **Competence** (mastery, skill growth) | Concrete percentile improvement over time, measurable progress | Core |
| **Relatedness** (connection, belonging) | Comparison to population norms ("where do I fit?") | Minimal |

### Player Type Appeal (Bartle Taxonomy)

- [x] **Achievers** (goal completion, collection, progression) — How: Beat personal bests, improve percentiles, complete all test modules, achieve consistency streaks
- [x] **Explorers** (discovery, understanding systems, finding secrets) — How: Discover cognitive patterns (time-of-day effects, practice curves), unlock advanced modules, understand what the data means
- [ ] **Socializers** — Not targeted
- [ ] **Killers/Competitors** — Not targeted

### Flow State Design

- **Onboarding curve**: First test is simple reaction time (tap when green) — universally understood in 2 seconds. Results screen explains percentile immediately. Colorblind test introduced in second battery.
- **Difficulty scaling**: Tests don't get "harder" — but advanced modules (choice reaction, peripheral detection) demand more cognitive load. The challenge is consistency, not difficulty.
- **Feedback clarity**: Exact millisecond display, percentile rank, trend arrows (improving/declining/stable), color-coded performance bands (green/yellow/red zones).
- **Recovery from failure**: No failure state. Every reaction is data. A "slow" reaction still contributes to your profile. Results framed as journey data ("Your score today: 312ms. Reaction time varies with fatigue and improves with practice"), not verdicts.
- **Colorblind reveal design**: Three-step emotional scaffolding: (1) Before color module, neutral framing card: "This test measures color discrimination. There is no pass or fail." (2) If results suggest CVD, contextual language: "You may have difficulty distinguishing red-green hues — this affects ~8% of men and ~0.5% of women." (3) Link to reputable resources (NHS, AAO). Never pathologize; always normalize.

---

## Core Loop

### Moment-to-Moment (30 seconds)
Watch the screen → stimulus appears → react (tap/click/keypress) → see your
reaction time in milliseconds with immediate visual+audio feedback → brief
pause → next trial. Each reaction feels crisp: the number pops onto screen
with a satisfying animation, colored by performance band. A single test module
runs 15-20 trials in ~60-90 seconds.

### Short-Term (5-15 minutes)
Complete a full test battery (reaction speed → color perception → choice
reaction). After each module, see per-module results: median RT, percentile,
best/worst trial, consistency score. After the full battery: composite
dashboard showing all modules. "One more battery" pull comes from wanting to
improve a weak module or verify an anomalous result.

### Session-Level (3-10 minutes)
Run 1-2 full batteries. Review the results dashboard: today's scores vs.
historical median, trend direction, colorblind assessment confidence level.
Natural stopping point is the results screen. Hook for return: "my colorblind
confidence is at 72% — two more sessions to reach diagnostic threshold" or
"my RT has been trending down — is that real or noise?"

### Long-Term Progression
- Historical trend lines build over days/weeks
- Colorblind detection confidence increases with more data points
- Advanced test modules unlock after completing initial batteries
- Cognitive profile becomes richer and more statistically significant
- Player can export/screenshot their complete profile

### Retention Hooks
- **Curiosity**: Is my morning reaction time really different from evening? Is the colorblind indicator converging on a diagnosis?
- **Investment**: Weeks of historical data that would be lost. Statistical significance requires ongoing sessions.
- **Social**: N/A (by design)
- **Mastery**: Reaction time percentile improvement. Consistency improvement. Beating personal median.

---

## Game Pillars

### Pillar 1: Measure, Don't Guess
Every result is backed by real statistical methodology — reaction time
percentiles from published norms, Ishihara plate logic for color vision,
proper trial counts for statistical significance. No fake numbers, no
inflated scores, no gamified distortions.

*Design test*: If debating between "show a fun but approximate result" vs.
"show a precise result with confidence interval," this pillar says show the
precise result.

### Pillar 2: Every Tap Feels Crisp
Inputs have zero perceived latency. Visual and audio feedback is immediate,
precise, and satisfying. The game feel is premium even though the graphics
are minimal. No animation delay between stimulus and response measurement.

*Design test*: If debating between "add an entrance animation before showing
the stimulus" vs. "show stimulus frame-instantly," this pillar says instant —
never sacrifice input fidelity for aesthetics.

### Pillar 3: Reveal Through Repetition
Each iteration adds signal. The game becomes more valuable the more you play —
not through unlocks or content gates, but through better data about yourself.
A single session gives a snapshot; a month of sessions gives a portrait.

*Design test*: If debating between "show definitive colorblind diagnosis after
one test" vs. "build diagnostic confidence over multiple sessions," this pillar
says build confidence — accuracy requires data, and accuracy is non-negotiable.

### Anti-Pillars (What This Game Is NOT)

- **NOT a social/competitive game**: No leaderboards, no PvP, no friend comparisons. Comparing yourself to population norms is scientific context; comparing to friends turns self-assessment into social pressure, compromising Pillar 1.
- **NOT a narrative experience**: No story, no characters, no world-building, no flavor text between tests. Every screen serves testing or results. Narrative would add friction that compromises Pillar 2 (crisp feel).
- **NOT a casual idle game**: Every interaction requires focused attention and deliberate input. Background or distracted play produces unreliable data, violating Pillar 1 (measure, don't guess).

---

## Inspiration and References

| Reference | What We Take From It | What We Do Differently | Why It Matters |
| ---- | ---- | ---- | ---- |
| Human Benchmark | Core reaction time testing, clean UI, instant feedback | Add colorblind detection, historical trends, statistical rigor, game juice | Validates massive audience for self-testing (10M+ monthly users) |
| Ishihara Test Plates | Gold-standard colorblind screening methodology | Curated plate set with randomized dot placement (not procedural color pairings), positioned as screening indicator not diagnosis | Validates the screening approach with 100+ years of clinical use |
| Wordle | Daily ritual, short sessions, shareable results | Testing instead of word puzzles, quantitative instead of binary outcome | Validates daily-ritual retention model for short-session games |

**Non-game inspirations**: Medical diagnostic interfaces (precision, clarity), Fitbit/Apple Health dashboards (personal data tracking as motivation), scientific instruments (the beauty of functional design).

---

## Target Player Profile

| Attribute | Detail |
| ---- | ---- |
| **Age range** | 16-45 |
| **Gaming experience** | Casual to mid-core — comfortable with web apps and browser games |
| **Time availability** | 3-10 minutes per session, potentially daily |
| **Platform preference** | Browser (desktop or mobile) |
| **Current games they play** | Human Benchmark, Wordle, brain training apps, typing speed tests |
| **What they're looking for** | Measurable self-knowledge — "am I normal?" answered with real data |
| **What would turn them away** | Inaccurate results, gamified distortions ("your brain age is 25!"), excessive UI friction, ads interrupting tests |

---

## Technical Considerations

| Consideration | Assessment |
| ---- | ---- |
| **Recommended Engine** | Godot 4 — lightweight, excellent HTML5 export, perfect for 2D UI-driven games, free/open-source |
| **Key Technical Challenges** | ~5ms input timing precision in browser (not sub-ms — browser event loop floor); browser-to-lab norm offset correction (+15ms); calibrated color rendering across uncalibrated displays; statistical computation (percentiles, confidence intervals) |
| **Art Style** | Minimalist 2D — geometric shapes, data visualization, no sprite art |
| **Art Pipeline Complexity** | Low — procedural/vector graphics, UI-driven aesthetic, no external art assets needed |
| **Audio Needs** | Moderate — crisp UI sounds for every interaction, ambient test-mode tone, result reveal sounds. No music during tests (would affect focus). |
| **Networking** | None — all data stored locally. No accounts, no server. |
| **Content Volume** | 3 test modules (MVP) → 8 modules (full vision), ~50 Ishihara plate variations, 1 results dashboard |
| **Procedural Systems** | Ishihara plate dot placement randomization (color pairings are fixed/curated, not procedural), stimulus timing randomization |

---

## Risks and Open Questions

### Design Risks
- Core loop has no evolution across sessions — session 10 is structurally identical to session 1. Mitigation: module variety at Vertical Slice tier; adaptive difficulty at Full Vision. Acknowledged: this is a ~10-session product for most users, not an infinite retention game.
- Both retention hooks (RT improvement, CVD confidence) expire within ~10 sessions. This is acceptable if positioned honestly — not every game needs infinite engagement.
- Results presentation must translate raw data into self-knowledge, not just display numbers. Interpretive results layer is a core mechanic, not a nice-to-have.

### Technical Risks
- Browser input timing has ~5ms precision floor (not sub-ms). Measurement displayed with ±5ms uncertainty band. Godot HTML5 export adds additional processing — must benchmark actual precision per browser.
- Lab RT norms (Luce 1986, Jain 2015) collected under hardware-timer CRT conditions. Browser measurements have ~15ms systematic offset. Must apply correction factor before percentile comparison or build browser-specific norm tables.
- Color rendering varies across uncalibrated displays — Ishihara screening on consumer TN panels may have 30-40% false negative rate for mild CVD. Mitigation: position as "screening indicator" not "diagnosis"; recommend professional testing if screening suggests CVD.
- Local storage cleared by private browsing, Safari 7-day policy, user action. One-click JSON export included at MVP to protect trend data.

### Market Risks
- Human Benchmark is free and established — must clearly differentiate on colorblind screening + interpretive results + historical tracking + game feel
- "Brain training" market has credibility issues after Lumosity FTC settlement — must position as measurement and screening, never as training or improvement tool

### Scope Risks
- Statistical analysis engine complexity (percentile computation, browser-offset correction, confidence intervals) may exceed initial estimates
- Curated Ishihara plate set requires color science expertise to validate color pairings
- Cross-browser timing benchmarking needed for Chrome/Firefox/Safari to establish per-browser offset corrections

### Open Questions
- What browser-specific RT offset correction should be applied? (Prototype needed: measure actual browser overhead vs. hardware timer baseline)
- How many curated Ishihara plates are needed for adequate screening power? (Research needed: minimum 12 plates for binary CVD present/absent screening with acceptable false negative rate)
- Should we support mobile/touch input at MVP, or desktop-only? (Prototype needed: measure touch input latency vs. keyboard/mouse)
- What false positive/negative rates are acceptable for a "screening indicator" positioning? (Define before implementation)

---

## MVP Definition

**Core hypothesis**: Players find the reaction-time and color-perception testing
loop engaging enough to complete 3+ sessions, and the results presentation
clearly communicates whether they're within normal ranges and whether they show
signs of color vision deficiency.

**Required for MVP**:
1. Simple reaction time test (tap when stimulus appears, 20 trials)
2. Color perception test (identify odd-colored dot, 12 curated Ishihara-style plates)
3. Results screen with interpretive language (anchored spectrum: Slow/Average/Fast/Elite) + raw ms as secondary detail + browser-corrected percentile
4. Colorblind screening result with 3-step emotional scaffolding (pre-framing → contextual result → resource link)
5. Immediate per-trial feedback (ms display with ±5ms band, same-frame spatial juice — no temporal animation)
6. Session history stored locally with one-click JSON export to protect trend data
7. Rolling trend display as primary view (not single-session verdicts)

**Explicitly NOT in MVP** (defer to later):
- Choice reaction test module (respond to specific stimuli only)
- Advanced colorblind type classification (protanopia vs. deuteranopia vs. tritanopia)
- Daily challenge mode
- Mobile-optimized layout
- Peripheral vision or pattern recognition modules

### Scope Tiers (if budget/time shrinks)

| Tier | Content | Features | Timeline |
| ---- | ---- | ---- | ---- |
| **MVP** | 2 test modules (reaction + color) | Core testing + results + normal-range comparison | 3-4 weeks, solo |
| **Vertical Slice** | 3 modules + colorblind report | + Historical tracking + trend visualization | 5-6 weeks, solo |
| **Alpha** | 5 modules + full Ishihara suite | + Advanced stats + session comparison + CVD type classification | 8-10 weeks, solo |
| **Full Vision** | 8 modules + adaptive difficulty | + Daily challenge mode + export/share + mobile layout | 12-16 weeks, solo |

---

## Next Steps

1. Run `/setup-engine` to configure Godot 4 and populate version-aware reference docs
2. Run `/art-bible` to create the visual identity specification — do this BEFORE writing GDDs. The art bible gates asset production and shapes technical architecture decisions (rendering, VFX, UI systems).
3. Use `/design-review design/gdd/game-concept.md` to validate concept completeness before going downstream
4. Discuss vision with the `creative-director` agent for pillar refinement
5. Decompose the concept into individual systems with `/map-systems` — maps dependencies, assigns priorities, and creates the systems index
6. Author per-system GDDs with `/design-system` — guided, section-by-section GDD writing for each system identified in step 5
7. Plan the technical architecture with `/create-architecture` — produces the master architecture blueprint and Required ADR list
8. Record key architectural decisions with `/architecture-decision (xN)` — write one ADR per decision in the Required ADR list from `/create-architecture`
9. Validate readiness to advance with `/gate-check` — phase gate before committing to production
10. Prototype the riskiest system with `/prototype [core-mechanic]` — validate the core loop before full implementation
11. Run `/playtest-report` after the prototype to validate the core hypothesis
12. If validated, plan the first sprint with `/sprint-plan new`
