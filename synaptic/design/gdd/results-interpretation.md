# Results & Interpretation

> **Status**: Designed
> **Author**: game-designer + ux-designer
> **Last Updated**: 2026-04-12
> **Implements Pillar**: Pillar 1 (Measure, Don't Guess), Pillar 3 (Reveal Through Repetition)

## Summary

The results and interpretation system transforms raw statistical output into
meaningful, anchored language that tells the player where they stand. It renders
the results dashboard, applies labeled performance spectrums, and manages the
emotional scaffolding for colorblind screening results. This is where numbers
become self-knowledge.

> **Quick reference** — Layer: `Feature` · Priority: `MVP` · Key deps: `Statistical Analysis Engine`

## Overview

After the Statistical Analysis Engine computes percentiles, bands, and trends, this
system decides how to present those findings. It translates a percentile rank into
anchored language ("About average — faster than most casual users, slower than
trained gamers"), renders the results dashboard with charts and metrics, and applies
the three-step emotional scaffolding for CVD screening results. The player interacts
with this system by reading and exploring their results. This is the system that
delivers on Pillar 1 — the player sees data, not flattery — and Pillar 3 — each
session's results are contextualized against the player's history.

## Player Fantasy

The results screen is the payoff. You've done the work — focused, reacted, been
measured — and now the instrument reveals what it found. The number sits large and
unambiguous: **247ms — 61st percentile — Average**. Below it, a clean spectrum
shows exactly where you fall. The fantasy is **seeing yourself quantified** — the
same satisfaction as stepping on a precise scale or checking your resting heart
rate. The number is not flattering or punishing. It simply is.

## Detailed Design

### Core Rules

1. **Primary metric display**: The corrected median RT is the headline number,
   displayed in 48px JetBrains Mono at screen center. The ±5ms precision band
   appears below in 16px. The raw (uncorrected) value is available as secondary
   detail on tap/hover.
2. **Percentile display**: Shown as "Xth percentile" in 24px JetBrains Mono
   below the primary metric. Accompanied by a horizontal spectrum bar showing
   the player's position.
3. **Performance band label**: Band boundaries are defined in
   `statistical-analysis-engine.md` Core Rule 6. Display styling per band:
   - "Elite" — `--precision-cyan`
   - "Fast" — `--precision-cyan` at 70% opacity
   - "Average" — `--signal-white`
   - "Slow" — `--alert-amber`
4. **Anchored interpretation text**: Below the band label, a one-sentence
   interpretation that anchors the result to real-world context:
   - Elite: "Exceptional — consistent with trained competitive gamers and athletes."
   - Fast: "Above average — faster than most people. Your reflexes are sharp."
   - Average: "Within normal range — consistent with most healthy adults."
   - Slow: "Below average — could reflect fatigue, distraction, or natural variation. Reaction time improves with practice."
5. **Consistency display**: IQR shown as "Consistency: Xms spread" with
   interpretation:
   - IQR ≤ 30ms: "Very consistent"
   - IQR 31–60ms: "Consistent"
   - IQR 61–100ms: "Variable — try testing when more focused"
   - IQR > 100ms: "Highly variable — results may reflect distraction"
6. **Trend display** (requires ≥3 sessions): Arrow indicator with label:
   - Improving (↑): trend arrow pointing up, `--precision-cyan`
   - Stable (→): horizontal dash, `--signal-white`
   - Declining (↓): trend arrow pointing down, `--alert-amber`
   - Insufficient data: "—" with "Need [N] more sessions for trend"
7. **Confidence display**: Progress bar showing screening confidence percentage.
   Label: "Data confidence: X%" with explanation "Based on [N] total trials."
8. **CVD results presentation**: Uses the three-step emotional scaffolding
   defined in the Color Perception System GDD. This system owns the rendering
   of those steps — the Color Perception System defines the content, this system
   defines the layout and timing.
9. **Historical comparison** (when history exists): "Today vs. your median:
   [+/-Xms]" with directional indicator. Shows whether this session is above
   or below the player's rolling average.
10. **No false precision**: Percentiles displayed as integers (not "61.25th").
    RT displayed as integers with ±5ms band. No decimal places anywhere in
    player-facing results.

### Results Screen Layout

Three-zone vertical stack (per art bible specification):

**Zone 1 — Header (fixed 64px)**:
- Session date/time, left-aligned, 12px Inter, `--neutral-text`
- Module type label ("Reaction Time" / "Color Perception"), right-aligned

**Zone 2 — Primary Metrics (flex, center-weighted)**:
- Corrected median RT in 48px JetBrains Mono, center
- ±5ms band in 12px, below RT, `--neutral-text`
- Percentile in 24px JetBrains Mono, below band
- Performance band label in 16px Inter 600, uppercase, below percentile
- Anchored interpretation text in 14px Inter, below band label, `--neutral-text`
- Horizontal spectrum bar (full width minus margins) showing position
- Consistency metric, left-aligned below spectrum
- Trend indicator, right-aligned below spectrum (same line as consistency)

**Zone 3 — Historical Chart (bottom, full width)**:
- Time-series chart of corrected median RT per session
- X-axis: session dates (labeled ticks, no grid lines)
- Y-axis: RT in ms, three reference lines (population 25th/50th/75th percentile)
- Current session highlighted as filled dot; previous sessions as line
- Chart follows art bible data visualization rules (1.5px lines, square caps)

### CVD Results Sub-Screen

Separate view (not mixed with RT results). Accessed via tab or swipe.

**Layout:**
- Pre-framing text at top (if first time seeing CVD results)
- CVD screening status: "No indicators" or "Possible [type] color vision difference"
- Per-category breakdown: "Red-green plates: [X]/[Y] identified correctly"
- Confidence bar: "[Z]% screening confidence"
- If indicators present: contextual text + resource links
- No color coding in CVD results (the tested colors should not appear in the results UI)

### States and Transitions

| State | Entry Condition | Exit Condition | Behavior |
|-------|----------------|----------------|----------|
| `LOADING` | Module completes, `AnalysisResult` received | All metrics computed and laid out | Brief (< 100ms) — compute layout, populate data |
| `DISPLAYING_RT` | Layout complete | User navigates to CVD tab or exits | RT results screen visible. Interactive (hover for raw values, tap for detail). |
| `DISPLAYING_CVD` | User selects CVD tab | User navigates back to RT or exits | CVD screening results visible. Emotional scaffolding applied. |
| `DISPLAYING_HISTORY` | User selects history view | User navigates back or exits | Full historical chart with session selector. |

### Interactions with Other Systems

| System | Direction | Interface |
|--------|-----------|-----------|
| **Statistical Analysis Engine** | Upstream (consumes) | Receives `AnalysisResult`: `median_rt`, `corrected_median_rt`, `percentile`, `band`, `iqr`, `confidence_pct`, `trend_direction`, `valid_trial_count`, `timeout_count`. Also receives CVD screening data. |
| **Session Persistence** | Upstream (reads) | Reads historical session data for trend charts and historical comparison. |
| **Test Sequencing** | Downstream (triggers) | User can start a new module from the results screen. Fires `start_new_module(type)` to Test Sequencing. |

## Formulas

### Anchored Language Selection

No mathematical formula — rule-based mapping from band to text. The interpretation
text is selected from a lookup table:

`interpretation_text = band_text_table[band]`

The text table is externally configured (not hardcoded), enabling localization
and A/B testing of language framings.

### Historical Comparison

`session_delta_ms = current_corrected_median - rolling_median`
`rolling_median = median(last_N_corrected_medians)` where N = `trend_window` from Stats

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| current_corrected_median | RT_c | int | 50–2000 | This session's corrected median RT |
| rolling_median | RT_roll | int | 50–2000 | Median of last N sessions' corrected medians |
| session_delta_ms | Δ | int | -1000–1000 | Positive = slower than average, negative = faster |

**Display:** "Today: [Δ]ms [vs. your average]" with arrow. Negative Δ shown without
minus sign, just "faster" label. Positive Δ shown as "slower."

**Example:** Current session median = 241ms, rolling median = 255ms.
Δ = 241 - 255 = -14. Display: "14ms faster than your average ↑"

### Spectrum Bar Position

`spectrum_position = 1.0 - (percentile / 100.0)`

Maps percentile (0–100) to horizontal position (0.0 = left/slow, 1.0 = right/fast).
Inverted because lower RT = higher percentile = further right on the "fast" end.

Band boundaries are marked on the spectrum at fixed positions:
- Slow/Average boundary: position 0.75 (25th percentile)
- Average/Fast boundary: position 0.25 (75th percentile)
- Fast/Elite boundary: position 0.05 (95th percentile)

## Edge Cases

| Scenario | Expected Behavior | Rationale |
|----------|------------------|-----------|
| Percentile = exactly 50 | Display "50th percentile" and band "Average." Interpretation: "Right at the median — exactly average." | Special language for the mathematical midpoint. |
| Percentile changes band between sessions (e.g., Fast → Average) | No alarm or special notice. New band displayed normally. Historical chart shows the change. | Band changes are normal variation. Drawing attention to them risks anxiety. Pillar 1: report, don't judge. |
| First ever session (no history) | Trend: "Need 2 more sessions for trend." Historical chart shows single data point. No historical comparison. | Cannot compute trends or comparisons without history. Be explicit about what data is needed. |
| All trials were timeouts | Display "Insufficient data — no valid responses recorded." No percentile, no band, no interpretation. Suggest: "Try again in a comfortable, distraction-free environment." | Cannot interpret nothing. Don't fabricate results. |
| Very low confidence (< 20%) | Display confidence bar. Add note: "Low confidence — results will become more reliable with additional sessions." | Set expectations about data quality early. |
| CVD indicators present for first time | Full three-step scaffolding. Pre-framing → result → resources. Scaffolding displayed regardless of whether player has seen it before (they may not remember). | The emotional weight of CVD information requires careful framing every time. |
| CVD confidence reaches 100% | Display "Screening complete" instead of confidence bar. Note: "Additional sessions will not significantly change this result." | Once confidence is maxed, continuing to show the bar implies more data is needed. |
| Player hovers/taps the raw RT value | Show tooltip or expanded detail: "Raw measurement: [X]ms. Browser correction: -[Y]ms. Corrected: [Z]ms." | Transparency about the correction builds trust. Pillar 1. |

## Dependencies

| System | Direction | Nature of Dependency |
|--------|-----------|---------------------|
| Statistical Analysis Engine | This depends on Stats | Requires `AnalysisResult` to display anything. Cannot function without it. Hard dependency. |
| Session Persistence | This depends on SP | Reads historical data for trends and charts. Can display current-session-only results without it. Soft dependency. |
| Test Sequencing | This depends on TS (soft) | Results screen can trigger new modules via Test Sequencing. TS can function without this trigger. Soft dependency. |

## Tuning Knobs

| Parameter | Current Value | Safe Range | Effect of Increase | Effect of Decrease |
|-----------|--------------|------------|-------------------|-------------------|
| `primary_metric_size_px` | 48 | 36–72 | Larger headline number; more dramatic | Smaller; less emphasis on single number |
| `spectrum_bar_height_px` | 8 | 4–16 | Taller spectrum; more prominent | Thinner; more subtle |
| `result_display_duration_ms` | ∞ (until navigation) | N/A | N/A | N/A |
| `chart_session_limit` | 30 | 10–100 | More history visible; denser chart | Less history; cleaner but less context |
| `iqr_thresholds` | [30, 60, 100] | Adjustable per bracket | Changes consistency label boundaries | — |

## Visual/Audio Requirements

| Event | Visual Feedback | Audio Feedback | Priority |
|-------|----------------|---------------|----------|
| Results screen enters | Metrics appear with 180ms horizontal slide (art bible transition). No fade. Numbers at final values. | None — silence after the test. | High |
| Band label appears | Label appears with the slide. Color matches band (cyan/white/amber). | None. | Medium |
| Historical chart draws | Chart renders instantly — no animated drawing/tracing. All data points visible immediately. | None. | Medium |
| CVD scaffolding pre-frame | Text card appears with 180ms slide. Neutral tone. | None. | High |
| CVD result reveal | Result text appears. If indicators: contextual text fades in after 1-second delay to allow reading pre-frame. | None. | Critical |

## UI Requirements

| Information | Display Location | Update Frequency | Condition |
|-------------|-----------------|-----------------|-----------|
| Corrected median RT | Center screen, Zone 2 | Once per results screen | Always on RT results |
| ±5ms precision band | Below RT number | Once | Always |
| Percentile | Below precision band | Once | Always (or "N/A" if insufficient data) |
| Performance band label | Below percentile | Once | Always |
| Anchored interpretation | Below band label | Once | Always |
| Spectrum bar | Full width, below interpretation | Once | Always |
| Consistency (IQR) | Left, below spectrum | Once | Always |
| Trend indicator | Right, below spectrum | Once | When ≥3 sessions exist |
| Confidence bar | Below consistency/trend | Once | Always |
| Historical chart | Zone 3, full width | Once | When ≥1 prior session exists |
| Session delta | Near chart, inline | Once | When ≥1 prior session exists |
| CVD screening status | CVD tab | Once | After color perception module |
| Navigation tabs | Top or bottom edge | Persistent | When both RT and CVD results exist |

## Cross-References

| This Document References | Target GDD | Specific Element Referenced | Nature |
|--------------------------|-----------|----------------------------|--------|
| Receives analysis results | `design/gdd/statistical-analysis-engine.md` | `AnalysisResult` data structure, band definitions, percentile table | Data dependency |
| Renders CVD emotional scaffolding | `design/gdd/color-perception-system.md` | Three-step scaffolding content, CVD threshold definitions | Rule dependency |
| Reads historical sessions | `design/gdd/session-persistence.md` | Stored `SessionStats` for trend chart | Data dependency |
| Can trigger new modules | `design/gdd/test-sequencing.md` | `start_new_module(type)` interface | State trigger |
| Uses art bible visual rules | Art Bible (`design/art/art-bible.md`) | Typography, color tokens, transition timing, chart style | Rule dependency |

## Acceptance Criteria

- [ ] GIVEN an AnalysisResult with percentile=61 and band="Average", WHEN results display, THEN the headline shows corrected median RT, "61st percentile", "Average" label, and the anchored interpretation text for Average band
- [ ] GIVEN a percentile of 97, WHEN band label renders, THEN it displays "Elite" in `--precision-cyan` color
- [ ] GIVEN an IQR of 45ms, WHEN consistency is displayed, THEN label shows "Consistent"
- [ ] GIVEN 5 sessions with improving trend, WHEN trend indicator renders, THEN upward arrow in `--precision-cyan` with "Improving" label
- [ ] GIVEN first-ever session, WHEN results display, THEN trend shows "Need 2 more sessions" and no historical comparison is shown
- [ ] GIVEN CVD indicators present, WHEN CVD tab is viewed, THEN three-step scaffolding is applied: pre-framing text → contextual result → resource links
- [ ] GIVEN no CVD indicators, WHEN CVD results display, THEN language says "No indicators of color vision deficiency detected" (not "you are not colorblind")
- [ ] GIVEN a player taps the RT number, WHEN detail is revealed, THEN raw measurement, browser correction, and corrected value are all shown
- [ ] GIVEN all trials were timeouts, WHEN results display, THEN "Insufficient data" is shown with no percentile, band, or interpretation
- [ ] GIVEN the spectrum bar, WHEN rendered, THEN the player's position marker sits at the correct horizontal position matching their percentile
- [ ] No hardcoded interpretation text — all anchored language loaded from external configuration

## Open Questions

| Question | Owner | Deadline | Resolution |
|----------|-------|----------|-----------|
| Should results auto-navigate to the CVD tab when indicators are first detected? Or let the player discover it? | UX Designer | Sprint 2 | Auto-navigation ensures the player sees important health information. Manual navigation respects autonomy. |
| Should the spectrum bar use continuous gradient or discrete band zones? | Art Director | Sprint 1 | Art bible says "no gradients" — discrete zones with hard boundaries. |
| Should historical chart show individual trial dots or only session medians? | Game Designer | Sprint 2 | Session medians only — individual trials are noise at the history timescale. |
