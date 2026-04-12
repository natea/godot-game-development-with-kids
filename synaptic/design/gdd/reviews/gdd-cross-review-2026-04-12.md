# Cross-GDD Review Report

**Date**: 2026-04-12
**GDDs Reviewed**: 7
**Systems Covered**: Stimulus-Response Engine, Statistical Analysis Engine, Color Perception System, Feedback System, Results & Interpretation, Session Persistence, Test Sequencing
**Pillars**: Measure Don't Guess, Every Tap Feels Crisp, Reveal Through Repetition
**Anti-Pillars**: NOT social/competitive, NOT narrative, NOT casual idle
**Entity Registry**: Empty — consistency checks rely on full GDD reads only.

---

## Consistency Issues

### Blocking (must resolve before architecture begins)

🔴 **CPS Inherits SRE Timeout State But Disables Timeout Behavior**

`color-perception-system.md` Core Rule 6: "No time pressure: Plates remain visible until the player responds. There is no timeout."

`color-perception-system.md` States Table: "All other states (IDLE, WAITING, TRIAL_COMPLETE, MODULE_COMPLETE) are inherited from the Stimulus-Response Engine."

`stimulus-response-engine.md` defines a `TIMEOUT` state that fires after 2000ms.

**Problem**: CPS says it inherits "all other states" from SRE, which includes the TIMEOUT state. But CPS explicitly says there is no timeout. The inherited TIMEOUT state would fire after 2000ms during plate display, contradicting the "no time pressure" rule.

**Resolution**: CPS must explicitly state that the `TIMEOUT` state is **removed** (not inherited) for the color perception module. Add to CPS states table: "TIMEOUT — Not inherited. Disabled for color perception (no time pressure)."

---

### Warnings (should resolve, but won't block)

⚠️ **W-01: Performance Band Definitions Duplicated Across Two GDDs**

`statistical-analysis-engine.md` Core Rule 6 defines band boundaries:
- Elite: ≥95th (≤185ms), Fast: 75th–94th (186–225ms), Average: 25th–74th (226–310ms), Slow: <25th (>310ms)

`results-interpretation.md` Core Rule 3 repeats the band definitions:
- Elite: ≥95th, Fast: 75th–94th, Average: 25th–74th, Slow: <25th

**Problem**: Two GDDs define the same band boundaries. If one is updated and the other isn't, they'll contradict. Stats should be the authoritative owner; R&I should reference Stats' definitions rather than redefining them.

**Resolution**: In R&I Core Rule 3, replace the inline band definitions with "Performance bands as defined in `statistical-analysis-engine.md` Core Rule 6" and keep only the display-specific information (colors, labels).

⚠️ **W-02: R&I Dependency Direction on Test Sequencing Is Ambiguously Worded**

`results-interpretation.md` Dependencies: "Test Sequencing | Depended on by TS (soft) | Results screen can trigger new modules via Test Sequencing."

The description says R&I triggers TS (R&I → TS), which means R&I has a soft dependency ON TS. But the table's direction column says "Depended on by TS" which implies TS depends on R&I. These are opposite directions.

`test-sequencing.md` Dependencies: "Results & Interpretation | Depended on by R&I (soft)" — says R&I depends on TS.

**Problem**: R&I's dependency table direction contradicts its own description. TS's table is correct.

**Resolution**: In R&I, change the direction from "Depended on by TS (soft)" to "This depends on TS (soft)."

⚠️ **W-03: SRE Lists Stats as Downstream But Interaction Table Omits It**

`stimulus-response-engine.md` Dependencies section lists "Statistical Analysis Engine | This is depended on by Stats" but the "Interactions with Other Systems" table does not include Statistical Analysis Engine as a consumer of the `module_completed` signal.

**Problem**: The Interactions table lists Color Perception, Feedback, and Test Sequencing as consumers, but Stats — which receives the primary `module_data` payload — is missing from the interactions table even though it's in the dependencies table.

**Resolution**: Add Statistical Analysis Engine to SRE's "Interactions with Other Systems" table as a downstream consumer of `module_completed`.

Wait — re-checking: SRE's Interactions table DOES list Stats: "Statistical Analysis Engine | Downstream (consumes) | Receives module_completed signal with module_data: Array[TrialResult]."

**Retraction**: This issue does not exist. SRE's interactions table correctly lists Stats. Removing this warning.

⚠️ **W-03 (revised): Anchored Interpretation Text Hardcoded in R&I GDD**

`results-interpretation.md` Core Rule 4 defines specific interpretation text strings ("Exceptional — consistent with trained competitive gamers..."). The Formulas section says the text table is "externally configured (not hardcoded), enabling localization." But the Acceptance Criteria don't verify that the text is loaded from config.

**Problem**: Minor inconsistency between the design intent (configurable text) and the acceptance criteria (no criterion verifying external loading of interpretation text).

**Resolution**: Add acceptance criterion: "Anchored interpretation text is loaded from external configuration, not hardcoded."

---

## Game Design Issues

### Blocking

None.

### Warnings

⚠️ **W-04: No Explicit Signal Ordering Between Concurrent Listeners**

When `module_completed` fires from SRE, three systems listen simultaneously: Feedback System, Statistical Analysis Engine, and Test Sequencing. No GDD specifies the processing order.

**Risk**: In Godot, signal callback order depends on connection order (tree order). If Feedback fires before Stats computes the analysis, Feedback cannot display analysis-dependent information. Currently Feedback only shows per-trial data (not module-level), so this is safe. But if any future change makes Feedback depend on Stats output, the ordering becomes critical.

**Recommendation**: Add a note to SRE's Interactions section specifying that `module_completed` listeners are order-independent (each system processes the raw data independently, no system depends on another system's processing of the same signal).

⚠️ **W-05: CPS Response Time Recorded But Never Used**

`color-perception-system.md` Per-plate data includes `response_time_ms`, but no system processes or displays this value. Stats only computes `cvd_miss_rate` from correctness, not response time. R&I doesn't display color test response times.

**Risk**: Recording unused data adds implementation complexity with no player value.

**Recommendation**: Either remove `response_time_ms` from PlateResult (simplify) or add a design intent for how it will be used in a future tier. If kept for future analysis, note it as "reserved for Vertical Slice — response time correlation with CVD."

---

## Cross-System Scenario Issues

**Scenarios walked**: 3

1. **Complete RT Module → Stats → Results** (RT-only individual test)
2. **Full Battery Flow** (RT → transition → Color → Results with both tabs)
3. **First-Time User Onboarding** (session count 0 → welcome → full battery → first results)

### Blockers

None.

### Warnings

⚠️ **Scenario 2: Last RT Trial Feedback vs. Module Transition Timing**

When the final RT trial completes, Feedback shows the RT number for 1.5 seconds. Simultaneously, SRE fires `module_completed`, which triggers TS to show a 3-second transition card. The transition should not start until the feedback display completes.

**Systems involved**: Feedback System, Test Sequencing, Stimulus-Response Engine

**Issue**: No GDD specifies the sequencing between Feedback's 1.5s display and TS's transition card. If TS immediately shows the transition card, the final trial's RT feedback is obscured.

**Recommendation**: Add to TS's inter-module transition rules: "Transition card appears after `inter_trial_delay_ms` (800ms from SRE) following the final trial, ensuring per-trial feedback completes before the transition."

### Info

ℹ️ **Scenario 3: Onboarding Instruction Assumes Mouse/Keyboard**

TS's onboarding instruction says "Click or press any key as fast as you can." SRE's input capture handles `InputEventMouseButton` and `InputEventKey`. Both are consistent. However, the game concept mentions "Web (Browser) + PC Desktop" as platform, and touch support is deferred. If touch is ever added, the instruction text would need updating.

**No action needed now** — just noting that the instruction text is hardcoded to mouse/keyboard input methods.

---

## GDDs Flagged for Revision

| GDD | Reason | Type | Priority |
|-----|--------|------|----------|
| color-perception-system.md | Timeout state inheritance contradiction | Consistency | Blocking |
| results-interpretation.md | Band definition duplication (W-01) and dependency direction (W-02) | Consistency | Warning |

---

## Verdict: CONCERNS

No fundamental design issues. One blocking consistency issue (CPS timeout inheritance) requires a one-line fix. Three warnings are minor wording/duplication issues. The 7 GDDs are coherent, pillar-aligned, and present a unified player fantasy of precision self-measurement.

### Required actions before proceeding:
1. **[BLOCKING]** Add explicit TIMEOUT state exclusion to CPS states table
2. **[WARNING]** Fix R&I band definition duplication (W-01) — 30-second edit
3. **[WARNING]** Fix R&I dependency direction wording (W-02) — 30-second edit
4. **[WARNING]** Add signal ordering note to SRE (W-04) — optional
5. **[WARNING]** Decide on CPS response_time_ms usage (W-05) — optional

### Entity Registry Note
Entity registry is empty. Run `/consistency-check` after this review to populate the registry with cross-system formulas and constants (browser_offset_ms, cvd_threshold, performance band boundaries, trial data structures).
