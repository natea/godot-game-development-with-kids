# Art Bible: Synaptic

*Created: 2026-04-12*
*Status: Draft*

---

## 1. Visual Identity Statement

**One-line visual rule**: The interface is the instrument — every pixel serves measurement or feedback, nothing exists for decoration.

**Supporting Principles:**

1. **Instrument Aesthetic** — Every UI element must look like it belongs on a scientific instrument, not a game menu. When choosing between a decorative treatment and a functional one, choose functional.
   *Design test*: "Should this element have a gradient, shadow, or glow?" → No. Flat, high-contrast, single-weight strokes only. *(Serves Pillar 2: Every Tap Feels Crisp)*

2. **Color Discipline** — Color is a controlled test variable, not a design tool. The UI chrome is achromatic; color appears only in test stimuli and performance data.
   *Design test*: "Can I use color to make this UI element more visually interesting?" → No. Use luminance contrast, weight, or spacing instead. *(Serves Pillar 1: Measure, Don't Guess)*

3. **Layout Stability** — Persistent readouts and data displays never reflow or shift position between states. The grid is the skeleton; elements occupy fixed addresses.
   *Design test*: "Should this element animate into a new position on state change?" → No. It can change content, opacity, or color — but not position. *(Serves Pillar 3: Reveal Through Repetition)*

---

## 2. Mood & Atmosphere

### Pre-test / Menu
- **Primary emotion**: Composed readiness — the calm before a controlled experiment begins
- **Lighting character**: Cool neutral ~6500K, low ambient luminance, high contrast on interactive elements only
- **Atmospheric descriptors**: Sterile, Deliberate, Unhurried, Calibrated, Weightless
- **Energy level**: 2/10 — Resting baseline

### Active Test: Reaction Speed
- **Primary emotion**: Suspended alertness — controlled vigilance, not anxiety
- **Lighting character**: Near-black canvas (~4% luminance), single high-contrast stimulus target, contrast ratio 20:1 minimum
- **Atmospheric descriptors**: Taut, Silent, Held-breath, Stark, Knife-edge
- **Energy level**: 7/10 — Coiled but still

### Active Test: Color Perception
- **Primary emotion**: Focused scrutiny — clinical, absorptive, almost meditative
- **Lighting character**: Neutral dark field ~7000K, surrounding UI bleached to near-zero saturation to avoid contaminating stimuli
- **Atmospheric descriptors**: Exacting, Crystalline, Concentrated, Neutral, Forensic
- **Energy level**: 5/10 — Deep focus, low arousal

### Per-Trial Feedback
- **Primary emotion**: Objective self-assessment — reading an instrument, not celebrating or mourning
- **Lighting character**: Brief luminance pulse on result value, then immediate decay. Warm-cool split: fast responses shift warm (~4000K accent), slow responses stay cold
- **Atmospheric descriptors**: Immediate, Precise, Unadorned, Transient, Factual
- **Energy level**: 4/10 — Sharp spike, then fade within 600ms

### Results Dashboard
- **Primary emotion**: Reflective clarity — satisfaction of seeing performance as data
- **Lighting character**: Slightly elevated ambient vs. test state. Data in cool white; percentile uses a single accent hue
- **Atmospheric descriptors**: Analytical, Revealing, Still, Structured, Earned
- **Energy level**: 3/10 — Post-exertion quiet

### Historical View
- **Primary emotion**: Longitudinal self-recognition — seeing yourself change over time
- **Lighting character**: Lowest ambient of all states. Older data = dimmer. Trend lines use luminance progression, not hue shift
- **Atmospheric descriptors**: Retrospective, Sparse, Continuous, Patient, Accumulative
- **Energy level**: 2/10 — Contemplative stillness

**Cross-state arc**: Energy escalates from Menu (2) → Reaction Test (7), then decays through Feedback (4) → Dashboard (3) → History (2). Color is structurally absent from test states — it earns meaning only in feedback and results.

---

## 3. Shape Language

### Stimulus Shapes: The Circle as Instrument
Reaction targets are **pure circles**. Rotationally symmetric — reads identically at any fixation angle, removing shape-orientation processing from the reflex loop. The circle carries zero aggression; it is neutral, clinical. Color becomes the only variable the player tracks. *(Serves Pillar 1: Measure, Don't Guess)*

### Data Visualization: Ruled Geometry
Charts use **straight lines, right angles, and sharp ticks** — the grammar of instruments. Trend lines are single-pixel strokes. Axes are hairlines. The grid underneath plots uses the same 8px base unit as the layout grid. Interface and data are the same object. *(Serves Pillar 3: Reveal Through Repetition)*

### UI Shape Grammar: Angular Containers, Micro-Radius CTAs
Panels and containers are **sharp-cornered rectangles**. Primary action buttons carry a **2px corner radius only** — just enough to signal "tappable" without softening the clinical register. This is the entire warmth budget. *(Serves Pillar 2: Every Tap Feels Crisp)*

### Interactive vs. Passive Distinction
Action elements (stimulus circles, primary CTAs) are **filled, high-contrast, center-weighted**. Passive elements (labels, axis ticks, historical lines) are **outlined or hairline weight, low opacity**. Filled shapes advance, stroked shapes recede.

### Grid and Spacing System
**8px base unit** governs all spacing. All component dimensions are multiples of 8. **12-column grid** with 16px gutters. Produces the density of a scientific dashboard without clutter.

---

## 4. Color System

### Primary Palette

| Token | Hex | Role |
|-------|-----|------|
| `--canvas` | `#0A0A0F` | Background — near-black, neutral-cool. The controlled testing field. |
| `--surface` | `#14141A` | Elevated containers — panels, cards. Barely lighter than canvas. |
| `--neutral-mid` | `#3A3A44` | Borders, dividers, axis lines. Structural but receding. |
| `--neutral-text` | `#8A8A96` | Secondary text, labels, captions. Readable but not dominant. |
| `--signal-white` | `#E8E8F0` | Primary text, active icons, metric readouts. The brightest UI element. |
| `--precision-cyan` | `#00D4FF` | The single accent — percentile markers, selected states, tap feedback ring. Clinical, not playful. |
| `--alert-amber` | `#FFB800` | Warning states only — below-average performance, calibration needed. |

### Semantic Color Usage

| Meaning | Color | Backup Cue (colorblind safe) |
|---------|-------|------------------------------|
| Fast / above average | `--precision-cyan` | Upward arrow icon |
| Normal / within range | `--signal-white` | Dash icon (—) |
| Slow / below average | `--alert-amber` | Downward arrow icon |
| Navigation active | `--precision-cyan` | 2px left border rule |
| Navigation inactive | `--neutral-text` | No border |

### Test Stimulus Colors (Isolated Partition)
Test stimuli use a **separate color namespace** prefixed `--stim-*`. These colors NEVER appear in UI chrome:

| Token | Usage |
|-------|-------|
| `--stim-red` | Ishihara plate: protan/deutan confusion pair |
| `--stim-green` | Ishihara plate: protan/deutan confusion pair |
| `--stim-blue` | Ishihara plate: tritan confusion pair |
| `--stim-yellow` | Ishihara plate: tritan confusion pair |
| `--stim-neutral` | Reaction speed stimulus (pure white flash on canvas) |

**Hard rule**: No green in UI chrome. `--stim-green` exists only in the test partition. Any green in the chrome would contaminate colorblind detection.

### UI Chrome Palette
UI uses only the achromatic range: `--canvas` through `--signal-white`, plus `--precision-cyan` and `--alert-amber` as the only two chromatic tokens. Total chromatic budget: 2 hues.

### Results Visualization
- Current session: `--precision-cyan` at full opacity
- Previous sessions: `--precision-cyan` at decreasing opacity (80% → 60% → 40%)
- Opacity-as-recency works identically for all color vision deficiency types
- Reference lines (population norms): `--neutral-mid`, dashed

### Stimulus Compositor Layer
The stimulus rendering area must be isolated from UI chrome at the rendering level. No `--ink` or chrome elements may bleed into the stimulus rectangle during a test frame. This is a technical requirement, not just a design intention.

---

## 5. Character Design Direction

**Not applicable.** Synaptic has no characters — no player avatar, no NPCs, no mascot. The player IS the subject. Any future consideration of character elements (e.g., a tutorial guide) must be evaluated against Pillar 1 (Measure, Don't Guess) — personality elements risk undermining clinical credibility.

---

## 6. Environment Design Language

**Not applicable.** Synaptic has no environments — no worlds, no rooms, no landscapes. The "environment" is the interface itself: a dark, controlled testing field. The canvas color (`--canvas: #0A0A0F`) IS the environment. It is deliberately empty — negative space is the design, not a gap to fill.

---

## 7. UI/HUD Visual Direction

### Typography

**Primary face: JetBrains Mono** — all numerical readouts, timestamps, and performance data. Monospaced is non-negotiable; column-aligned numbers are a readability requirement.

**Secondary face: Inter** — labels, navigation, and body copy. Tabular numerals enabled (`font-variant-numeric: tabular-nums`).

| Role | Face | Weight | Size |
|------|------|--------|------|
| Metric primary (reaction time) | JetBrains Mono | 700 | 48px |
| Metric secondary (percentile) | JetBrains Mono | 400 | 24px |
| Section label | Inter | 600 | 12px, uppercase, 0.08em tracking |
| Body / description | Inter | 400 | 16px |
| Caption / axis label | Inter | 400 | 10px |

No italic anywhere. Italics imply narration; this instrument does not narrate.

### Iconography
Outlined, 1.5px stroke, `--neutral-text` default, `--signal-white` on active. Icons: play (right-pointing triangle outline), back (left chevron), settings (grid of four equal squares — not a gear), module icons as abstract measurement glyphs (waveform, reticle, color swatch grid). No filled icons. No drop shadows. No bounding containers.

### Animation

| Element | Duration | Curve | Notes |
|---------|----------|-------|-------|
| Screen transitions | 180ms | cubic-bezier(0.4, 0, 0.2, 1) | Horizontal slide. No fades — fades imply ambiguity. Deeper = rightward. |
| Number animation | 400ms | ease-out | Counter rollup from previous value. Numbers count to their answer, reinforcing measurement. |
| Tap feedback | 80ms | spring, no overshoot | Stimulus scales 1.0 → 0.88 → 1.0. A 1px `--precision-cyan` ring expands from tap origin at 120ms and fades. |
| Trial progress | continuous | linear | 1px `--neutral-mid` rule at screen top fills left-to-right. |

### Data Visualization
Single-weight lines at 1.5px. No area fills. Square line caps (not rounded — precision). X-axis: labeled tick marks only, no full grid lines. Y-axis: three labeled reference lines (`--neutral-mid`, dashed, 1px). Dot markers: 4px filled circle at each data point on hover only. Series directly labeled inline, right-aligned to line terminus.

### Screen Composition

**Test screen**: Single stimulus circle centered in viewport. No chrome during active test. Reaction timer top-right, 24px JetBrains Mono, `--neutral-text`. Progress rule at top.

**Results screen**: Three-zone vertical stack. Top (24px cap): session identity. Middle (flex): primary metric large, secondaries in horizontal rule below. Bottom: time-series chart, full width.

**History screen**: Fixed 240px left column lists sessions as dated rows. Right panel renders chart for selected session. Selected row: 2px left `--precision-cyan` border.

---

## 8. Asset Standards

### File Naming
All files: `snake_case`. Pattern: `[category]_[name]_[variant].[ext]`
Examples: `ui_btn_primary_hover.tres`, `icon_arrow_right.svg`, `sfx_tap_correct.ogg`

### Directory Structure
```
assets/
  fonts/          # .ttf files only (Godot web export compatibility)
  icons/          # SVG source + exported PNG fallbacks
  audio/sfx/      # OGG Vorbis files
  themes/         # .tres Godot theme resources
  textures/       # Generated Ishihara plates if pre-baked (PNG)
```

### Font Requirements
- Format: `.ttf` only (web export compatibility)
- Two families: one monospace (JetBrains Mono), one sans-serif (Inter)
- Theme sizes: 10, 12, 14, 16, 18, 24, 36, 48px
- Fonts bundled in project, no web font loading

### Icon Standards
- Source: SVG, 24x24px viewBox, `currentColor` fill
- Export: PNG at 1x (24px) and 2x (48px) — no 3x (web + PC only)
- Stroke weight: 1.5px at 24px, pixel-grid aligned
- No gradients, no shadows — flat geometric only

### Audio Standards
- Format: OGG Vorbis exclusively (Godot web export requirement)
- Sample rate: 44.1kHz, mono for SFX
- Normalization: -3 dBFS peak, -18 LUFS integrated
- Max duration: 2s for tap/response SFX, 8s for ambient loops
- Naming: `sfx_[action]_[variant].ogg`

### Texture Standards (Ishihara Plates)
- Prefer runtime generation via GDScript (no stored texture)
- If pre-baked: 512x512px PNG, power-of-two, no mipmaps, import as Lossless
- Colors must map exactly to `--stim-*` tokens

### Godot Theme Standards
- One root theme: `themes/theme_base.tres` — all StyleBoxFlat, no images
- Variants as named theme overrides, not separate .tres files
- All spacing values divisible by 8 (grid rule)
- No embedded fonts — reference by path
- Font sizes defined once in theme, never overridden per-node

---

## 9. Reference Direction

### R1 — Monolith (iOS/Android fitness app)
**Draw from**: The use of a single large numerical readout dominating the viewport — one number, massive, center-weighted. Everything else recedes.
**Avoid**: The warm off-white background and rounded softness. Synaptic uses near-black canvas and hard edges.

### R2 — Saleae Logic Analyzer UI
**Draw from**: The timeline ruler with fine tick marks, the instrument-panel density of labeled channels, and the way inactive areas visually "turn off" rather than disappear.
**Avoid**: Information overload at rest state. Synaptic shows nothing that is not currently active.

### R3 — Farnsworth-Munsell 100 Hue Test (physical test card grid)
**Draw from**: The clinical grid layout of color chips — equal spacing, no decoration, the chips ARE the content. Direct translation to stimulus grid layout.
**Avoid**: The analog/paper texture feel. Surfaces stay flat and backlit.

### R4 — Linear (project management app)
**Draw from**: The sidebar's use of 1px neutral dividers to imply structure without weight, and micro-animation of state transitions (hover underline rules, not color floods).
**Avoid**: The friendly rounded corners and pastel accents.

### R5 — CANTAB (Cambridge Neuropsychological Test Automated Battery)
**Draw from**: The deliberate blank negative space between trials — screen goes empty between stimuli, creating measurable psychological anticipation. Use as a pacing reference for inter-trial intervals.
**Avoid**: The dated early-2000s UI chrome (beveled buttons, gradient fills). Extract the pacing principle only.

---

## Style Prohibitions

These are absolute prohibitions — violations indicate a misunderstanding of the visual identity:

1. **No gradients** — anywhere, ever. Flat fills and opacity only.
2. **No drop shadows** — elevation is communicated by luminance difference between `--canvas` and `--surface`.
3. **No rounded corners > 2px** — the micro-radius on CTAs is the maximum.
4. **No decorative color** — if a color cannot be justified as data or test stimulus, it does not appear.
5. **No particle effects or ambient animation** — nothing moves unless the player caused it.
6. **No serif or display typefaces** — JetBrains Mono and Inter are the complete type palette.
7. **No green in UI chrome** — green exists only in the `--stim-*` test partition.
8. **No texture or noise overlays** — surfaces are mathematically flat.
9. **No skeuomorphism** — no fake materials, no simulated depth, no "realistic" instrument bezels.
10. **No emoji or illustrative elements** — the game's personality comes from precision, not personality.
