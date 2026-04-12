# Statistical Analysis Engine

> **Status**: Designed
> **Author**: systems-designer + game-designer
> **Last Updated**: 2026-04-12
> **Implements Pillar**: Pillar 1 (Measure, Don't Guess), Pillar 3 (Reveal Through Repetition)

## Summary

The statistical analysis engine transforms raw reaction time trial data into
meaningful percentile rankings, confidence intervals, and trend metrics. It applies
browser-offset correction to align browser-measured times with published laboratory
norms, enabling the game to make scientifically grounded claims about where the
player sits in the population distribution.

> **Quick reference** — Layer: `Foundation` · Priority: `MVP` · Key deps: `None`

## Overview

After each test module produces a set of reaction times, the statistical analysis
engine computes what those numbers mean. It calculates median RT, filters outliers,
applies a browser-offset correction factor to align measurements with published lab
norms, computes the player's percentile rank against the population, and determines
whether the player's performance falls in the Slow, Average, Fast, or Elite band.
Over multiple sessions, it tracks rolling statistics and determines trend direction
(improving, declining, or stable). The player never interacts with this system
directly — they experience its output through the Results & Interpretation system.
This system exists because raw millisecond values are meaningless without context,
and Pillar 1 demands that context be statistically rigorous.

## Player Fantasy

This is infrastructure the player experiences indirectly. The fantasy it enables:
**your number means something real.** When the game says "faster than 73% of
people," that claim is backed by proper statistics — not a random encouragement
message. The player feels the precision of the instrument, not the instrument
itself. This system is invisible when working correctly and would be immediately
noticed if absent (raw ms with no context) or wrong (inflated percentiles).

## Detailed Design

### Core Rules

1. **Input**: An array of `TrialResult` objects from the Stimulus-Response Engine,
   each containing `rt_ms`, `is_timeout`, `is_early`, `stimulus_timestamp`,
   `response_timestamp`.
2. **Outlier filtering**: Before computing statistics, remove trials where
   `is_timeout=true` or `is_early=true`. These are recorded for completeness
   metrics but excluded from RT calculations.
3. **Central tendency**: Use **median** as the primary metric, not mean. Median is
   robust to the right-skewed distribution of human reaction times and resistant
   to single-trial outliers.
4. **Browser-offset correction**: Subtract `browser_offset_ms` (default: 15ms)
   from the median RT before percentile lookup. This corrects for systematic
   measurement overhead in browser-based timing vs. laboratory hardware timers.
   The corrected value is called `corrected_median_rt`.
5. **Percentile computation**: Look up `corrected_median_rt` against a reference
   distribution table (published norms). Use linear interpolation between table
   entries for values between reference points.
6. **Performance band assignment**: Map the percentile to a labeled band:
   - Elite: ≥95th percentile (corrected median ≤ 185ms)
   - Fast: 75th–94th percentile (corrected median 186ms–225ms)
   - Average: 25th–74th percentile (corrected median 226ms–310ms)
   - Slow: <25th percentile (corrected median > 310ms)
7. **Consistency score**: Compute the interquartile range (IQR) of valid trials.
   Lower IQR = more consistent. Reported alongside median RT.
8. **Confidence level**: After each session, compute a cumulative confidence metric
   for the player's "true" RT based on total valid trials across all sessions.
   More trials = higher confidence that the reported percentile reflects stable
   ability rather than noise.
9. **Trend computation**: Over the last N sessions (default: 5), compute the
   direction of the rolling median. If the slope of the linear regression through
   session medians is negative by more than `trend_threshold_ms` per session,
   report "Improving." If positive by more than threshold, report "Declining."
   Otherwise, "Stable."

### States and Transitions

This system is stateless — it is a pure computation engine. It receives data,
processes it, and returns results. No persistent state between calls (session
persistence is handled by the Session Persistence system).

### Interactions with Other Systems

| System | Direction | Interface |
|--------|-----------|-----------|
| **Stimulus-Response Engine** | Upstream (provides data) | Receives `module_data: Array[TrialResult]` via `module_completed` signal. |
| **Color Perception System** | Upstream (provides data) | Receives `plate_results: Array[PlateResult]` with correctness data. Computes separate color perception metrics (not RT-based). |
| **Results & Interpretation** | Downstream (consumes) | Provides `AnalysisResult` containing: `median_rt`, `corrected_median_rt`, `percentile`, `band`, `iqr`, `confidence_pct`, `trend_direction`, `valid_trial_count`, `timeout_count`. |
| **Session Persistence** | Downstream (consumes) | Provides per-session `SessionStats` for storage: `date`, `module_type`, `median_rt`, `corrected_median_rt`, `percentile`, `band`, `iqr`, `trial_count`. |

## Formulas

### Browser-Offset Correction

The corrected_median_rt formula is defined as:

`corrected_median_rt = raw_median_rt - browser_offset_ms`

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| raw_median_rt | RT_raw | float | 100–2000 | Median of valid trial RTs as measured by browser |
| browser_offset_ms | B | int | 0–50 | Systematic browser timing overhead correction |
| corrected_median_rt | RT_c | float | 50–2000 | Lab-equivalent median RT |

**Output Range:** 50ms to 2000ms. Values below 100ms after correction are flagged
as likely measurement artifacts.

**Example:** Raw median = 262ms, browser offset = 15ms. Corrected = 262 - 15 = 247ms.

### Percentile Lookup

The percentile formula uses a reference distribution table derived from published
norms (Luce 1986, Jain et al. 2015), adjusted for age group 18–39:

| Percentile | Corrected Median RT (ms) |
|------------|-------------------------|
| 99th | ≤ 155 |
| 95th | 156–185 |
| 90th | 186–200 |
| 75th | 201–225 |
| 50th | 226–265 |
| 25th | 266–310 |
| 10th | 311–370 |
| 5th | 371–420 |
| 1st | > 420 |

For values between reference points, use linear interpolation:

`percentile = lower_pct + (upper_pct - lower_pct) * (upper_rt - corrected_rt) / (upper_rt - lower_rt)`

**Example:** Corrected RT = 247ms falls between 50th (265ms) and 75th (225ms).
Percentile = 50 + (75 - 50) * (265 - 247) / (265 - 225) = 50 + 25 * 18/40 = 61.25 → 61st percentile.

### Interquartile Range (Consistency)

`iqr = Q3(valid_rts) - Q1(valid_rts)`

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| Q1 | Q1 | float | 100–2000 | 25th percentile of this module's valid RTs |
| Q3 | Q3 | float | 100–2000 | 75th percentile of this module's valid RTs |
| iqr | IQR | float | 0–1000 | Spread of middle 50% of responses |

**Output Range:** Typical IQR for focused adults: 20ms–60ms. IQR > 100ms suggests
distraction or fatigue.

**Example:** Q1 = 235ms, Q3 = 271ms. IQR = 271 - 235 = 36ms (consistent).

### Confidence Level

`confidence_pct = min(100, (total_valid_trials / confidence_ceiling) * 100)`

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| total_valid_trials | N | int | 0–∞ | Cumulative valid trials across all sessions |
| confidence_ceiling | C | int | 60–200 | Trials needed for maximum confidence |
| confidence_pct | conf | float | 0–100 | How confident the system is in the percentile |

**Output Range:** 0% (no data) to 100% (fully confident). Reaches 50% at
`confidence_ceiling / 2` trials.

**Example:** Player has completed 3 sessions of 20 trials each = 54 valid trials
(after filtering 6 timeouts). Confidence = min(100, 54/100 * 100) = 54%.

### Trend Direction

`trend_slope = linear_regression_slope(session_medians[-trend_window:])`

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| session_medians | M[] | float[] | — | Array of corrected median RTs per session |
| trend_window | W | int | 3–10 | Number of recent sessions to analyze |
| trend_slope | s | float | -∞–∞ | ms change per session (negative = improving) |
| trend_threshold_ms | T | float | 1–10 | Minimum slope magnitude to declare a trend |

**Interpretation:**
- slope < -`trend_threshold_ms`: "Improving" (getting faster)
- slope > +`trend_threshold_ms`: "Declining" (getting slower)
- |slope| ≤ `trend_threshold_ms`: "Stable"

**Example:** Last 5 session medians: [261, 255, 249, 252, 244]. Slope ≈ -3.8ms/session.
With threshold = 3ms, trend = "Improving."

## Edge Cases

| Scenario | Expected Behavior | Rationale |
|----------|------------------|-----------|
| All trials in a module are timeouts | Return `AnalysisResult` with `median_rt=null`, `percentile=null`, `band="Insufficient Data"`. `valid_trial_count=0`. | Cannot compute statistics without valid data. Don't fabricate results. Pillar 1. |
| Only 1 valid trial in module | Compute median from single value. Set `iqr=0`. Flag `low_confidence=true`. | One data point is better than none but insufficient for robust statistics. |
| Corrected RT is negative (offset > raw) | Clamp to `min_corrected_rt` (50ms). Log as measurement anomaly. | Physically impossible to have negative RT. Clamping preserves data while flagging the issue. |
| Corrected RT below 100ms | Record normally but flag `is_suspicious=true`. Do not exclude from calculations. | Sub-100ms simple RT is physiologically rare but possible. Let the stats speak — repeated sub-100ms values will raise confidence or reveal a measurement issue. |
| Player has only 1 session (no trend data) | Trend = "Not enough data" (not "Stable"). Display requires minimum `trend_min_sessions` (3) sessions. | "Stable" implies evidence of stability. With 1 data point, there is no evidence either way. |
| Percentile lookup RT exactly matches a reference boundary | Assign the percentile of the boundary value. No interpolation needed. | Eliminates ambiguity at boundaries. |
| Two sessions recorded on the same calendar day | Both contribute to trend data independently. Sessions are identified by timestamp, not date. | Multiple daily sessions are valid and expected. |
| Browser offset correction changes between sessions | Recalculate all historical corrected RTs when offset changes. Notify player: "Calibration updated — historical results recalculated." | Historical percentiles must remain consistent with current calibration. |

## Dependencies

| System | Direction | Nature of Dependency |
|--------|-----------|---------------------|
| Stimulus-Response Engine | This depends on SRE | Requires `module_data: Array[TrialResult]` as input. Cannot function without trial data. Hard dependency. |
| Color Perception System | This depends on CPS (soft) | Receives plate results for color perception analysis. Can function without it (RT-only mode). Soft dependency. |
| Results & Interpretation | Depended on by R&I | Provides `AnalysisResult` for interpretive language generation. R&I cannot function without this output. |
| Session Persistence | Depended on by SP | Provides `SessionStats` for storage. SP cannot compute trends without this data. |

**Upstream dependencies:** Stimulus-Response Engine (hard), Color Perception System (soft).

## Tuning Knobs

| Parameter | Current Value | Safe Range | Effect of Increase | Effect of Decrease |
|-----------|--------------|------------|-------------------|-------------------|
| `browser_offset_ms` | 15 | 0–50 | Higher correction = lower corrected RT = higher percentile. Over-correction inflates results. | Lower correction = higher corrected RT = lower percentile. Under-correction deflates results. |
| `confidence_ceiling` | 100 | 60–200 | Harder to reach 100% confidence; more sessions needed. More conservative. | Easier to reach full confidence; fewer sessions. May report high confidence prematurely. |
| `trend_window` | 5 | 3–10 | Trend smoothed over more sessions; slower to detect changes. More stable. | Trend more responsive to recent sessions; noisier. |
| `trend_threshold_ms` | 3 | 1–10 | Harder to trigger "Improving" or "Declining"; more data needed. | Easier to trigger trend labels; may react to noise. |
| `min_corrected_rt` | 50 | 30–100 | More generous floor; allows lower corrected values through. | Stricter floor; more aggressive anomaly clamping. |

**Interaction warnings:**
- `browser_offset_ms` directly shifts all percentile assignments. A 5ms change can move a player 3-5 percentile points. Must be validated empirically, not tuned casually.
- `trend_threshold_ms` and `trend_window` interact: a large window with a small threshold will detect slow trends; a small window with a large threshold will only detect dramatic changes.

## Visual/Audio Requirements

This system has no direct visual or audio output — it is a pure computation engine.
All presentation is handled by the Results & Interpretation system and the Feedback
System. However, it provides data that shapes visual presentation:

| Data Output | Consuming System | Visual Use |
|-------------|-----------------|------------|
| `percentile` | Results & Interpretation | Percentile number display, spectrum position |
| `band` | Results & Interpretation | Performance band color coding (cyan/white/amber) |
| `trend_direction` | Results & Interpretation | Trend arrow direction (↑/→/↓) |
| `confidence_pct` | Results & Interpretation | Confidence bar fill level |
| `iqr` | Results & Interpretation | Consistency indicator |

## UI Requirements

No direct UI. All data is consumed by the Results & Interpretation system for display.

## Cross-References

| This Document References | Target GDD | Specific Element Referenced | Nature |
|--------------------------|-----------|----------------------------|--------|
| Trial data input format | `design/gdd/stimulus-response-engine.md` | `TrialResult` data structure and `module_completed` signal | Data dependency |
| Plate results for color analysis | `design/gdd/color-perception-system.md` | `PlateResult` data structure | Data dependency |
| Analysis results consumed by interpretation | `design/gdd/results-interpretation.md` | `AnalysisResult` output format | Data dependency |
| Session stats stored by persistence | `design/gdd/session-persistence.md` | `SessionStats` output format | Data dependency |

## Acceptance Criteria

- [ ] GIVEN a module of 20 trials with 2 timeouts, WHEN analysis runs, THEN median is computed from the 18 valid trials only
- [ ] GIVEN a raw median RT of 262ms and browser offset of 15ms, WHEN correction is applied, THEN corrected median = 247ms
- [ ] GIVEN a corrected median of 247ms, WHEN percentile is computed against the reference table, THEN result is approximately 61st percentile (within ±1 due to interpolation)
- [ ] GIVEN a corrected median of 180ms, WHEN band is assigned, THEN band = "Elite" (≥95th percentile)
- [ ] GIVEN 5 sessions with declining median RTs, WHEN trend is computed, THEN trend_direction = "Improving"
- [ ] GIVEN a player with 54 valid trials and confidence_ceiling of 100, WHEN confidence is computed, THEN confidence_pct = 54%
- [ ] GIVEN all trials are timeouts, WHEN analysis runs, THEN median_rt = null and band = "Insufficient Data"
- [ ] GIVEN browser_offset_ms changes, WHEN historical data is present, THEN all historical corrected RTs are recalculated
- [ ] Performance: Full analysis of a 20-trial module completes in < 1ms
- [ ] No hardcoded reference distribution values — percentile table loaded from external configuration

## Open Questions

| Question | Owner | Deadline | Resolution |
|----------|-------|----------|-----------|
| Should the reference distribution table be age-stratified? Current design uses 18-39 age group only. | Game Designer | Before Vertical Slice | May need age input from player for accurate percentiles |
| What is the actual browser offset for Chrome/Firefox/Safari? 15ms is estimated from literature. | Technical Director | Before Alpha | Prototype benchmarking needed |
| Should we build browser-specific norm tables instead of using a single offset correction? | Systems Designer | Before Alpha | Depends on variance between browsers |
| Should IQR thresholds for "consistent" vs "inconsistent" be tunable or fixed? | Game Designer | Sprint 2 | Currently not banded — just raw IQR reported |
