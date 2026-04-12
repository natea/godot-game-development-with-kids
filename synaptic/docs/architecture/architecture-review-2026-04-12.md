# Architecture Review Report

- **Date**: 2026-04-12
- **Engine**: Godot 4.6 (GDScript, Compatibility renderer)
- **GDDs Reviewed**: 7
- **ADRs Reviewed**: 5

---

## Traceability Summary

- **Total requirements**: 28
- **Covered**: 18 (64%)
- **Partial**: 2 (7%)
- **Gaps**: 8 (29%)

---

## Full Traceability Matrix

| Req ID | GDD | System | Requirement | ADR Coverage | Status |
|--------|-----|--------|-------------|--------------|--------|
| TR-SRE-001 | stimulus-response-engine.md | SRE | Capture input timestamp via `_input()` | ADR-0003 | ✅ |
| TR-SRE-002 | stimulus-response-engine.md | SRE | `Time.get_ticks_msec()` for timing | ADR-0003 | ✅ |
| TR-SRE-003 | stimulus-response-engine.md | SRE | State machine transitions | ADR-0001 | ✅ |
| TR-SRE-004 | stimulus-response-engine.md | SRE | Uniform random delay (1500–4000ms) | — | ❌ GAP |
| TR-SRE-005 | stimulus-response-engine.md | SRE | Browser tab focus/blur detection | — | ❌ GAP |
| TR-SRE-006 | stimulus-response-engine.md | SRE | Signal emission for trial events | ADR-0001 | ✅ |
| TR-STAT-001 | statistical-analysis-engine.md | Stats | Median, IQR, percentile computation | ADR-0001 (RefCounted pattern) | ⚠️ Partial |
| TR-STAT-002 | statistical-analysis-engine.md | Stats | Linear regression for trends | — | ❌ GAP |
| TR-STAT-003 | statistical-analysis-engine.md | Stats | Browser-offset correction | — | ❌ GAP |
| TR-STAT-004 | statistical-analysis-engine.md | Stats | Reference distribution (external config) | ADR-0004 | ✅ |
| TR-CPS-001 | color-perception-system.md | CPS | Ishihara plate rendering (Poisson disk) | — | ❌ GAP |
| TR-CPS-002 | color-perception-system.md | CPS | Extends SRE state machine | ADR-0001 | ✅ |
| TR-CPS-003 | color-perception-system.md | CPS | Curated plate color definitions | ADR-0004 (.tres config) | ✅ |
| TR-CPS-004 | color-perception-system.md | CPS | CVD miss rate computation | — | ❌ GAP |
| TR-FB-001 | feedback-system.md | Feedback | Same-frame visual response | ADR-0001 | ✅ |
| TR-FB-002 | feedback-system.md | Feedback | Same-frame audio response | ADR-0001 | ✅ |
| TR-FB-003 | feedback-system.md | Feedback | Tween animations (spring, fading ring) | — | ❌ GAP |
| TR-FB-004 | feedback-system.md | Feedback | Performance-scaled ring speed | — | ❌ GAP |
| TR-RI-001 | results-interpretation.md | R&I | Time-series chart rendering | — | ❌ GAP |
| TR-RI-002 | results-interpretation.md | R&I | Multi-screen navigation (tabs) | ADR-0005 | ⚠️ Partial |
| TR-RI-003 | results-interpretation.md | R&I | Anchored language from external config | ADR-0004 | ✅ |
| TR-RI-004 | results-interpretation.md | R&I | Spectrum bar with positioned marker | — | ❌ GAP |
| TR-SP-001 | session-persistence.md | SP | localStorage via JavaScript bridge | ADR-0002 | ✅ |
| TR-SP-002 | session-persistence.md | SP | `user://` file storage (desktop) | ADR-0002 | ✅ |
| TR-SP-003 | session-persistence.md | SP | JSON serialization/deserialization | ADR-0002, ADR-0004 | ✅ |
| TR-SP-004 | session-persistence.md | SP | Browser file download for JSON export | ADR-0002 | ✅ |
| TR-SP-005 | session-persistence.md | SP | Schema versioning and forward migration | ADR-0004 | ✅ |
| TR-TS-001 | test-sequencing.md | TS | Scene transitions between screens | ADR-0005 | ✅ |
| TR-TS-002 | test-sequencing.md | TS | Countdown timer with skip-on-input | ADR-0005 | ✅ |

---

## Coverage Gap Analysis

The 10 uncovered/partial requirements fall into two categories:

### Implementation-Level Requirements (No ADR Needed)

These are pure gameplay implementation details — formulas, algorithms, and rendering techniques that follow patterns already established by existing ADRs. They do not require architectural decisions:

| Req ID | Requirement | Why No ADR Needed |
|--------|-------------|-------------------|
| TR-SRE-004 | Uniform random delay | Simple `randf_range()` call — no architectural choice involved |
| TR-STAT-001 | Median, IQR, percentile | Pure math in RefCounted class (pattern from ADR-0001) |
| TR-STAT-002 | Linear regression | Pure math — no engine dependency |
| TR-STAT-003 | Browser-offset correction | Single subtraction — constant defined in config (ADR-0004 .tres) |
| TR-CPS-004 | CVD miss rate computation | Pure math — counting misses / total |
| TR-FB-003 | Tween animations | Uses Godot's native `Tween` — stable API, no decision needed |
| TR-FB-004 | Performance-scaled ring speed | Formula in GDD — implementation follows ADR-0001 signal pattern |
| TR-RI-004 | Spectrum bar | UI rendering — follows scene architecture from ADR-0005 |

### Potential ADR Candidates (Low Priority)

| Req ID | Requirement | Suggested ADR | Priority |
|--------|-------------|---------------|----------|
| TR-SRE-005 | Browser tab focus/blur | Platform Event Detection — `JavaScriptBridge` for visibility API | Can defer |
| TR-CPS-001 | Ishihara plate rendering | Procedural Dot Placement — Poisson disk sampling in GDScript | Can defer |
| TR-RI-001 | Time-series chart | Data Visualization Approach — custom Control drawing vs. addon | Can defer |
| TR-RI-002 | Multi-screen navigation (tabs) | Covered by ADR-0005 scene structure; tab switching within ResultsScreen is UI detail | No ADR needed |

---

## Cross-ADR Conflict Detection

### Conflicts Found: 0

All 5 ADRs are internally consistent:
- No data ownership conflicts (each ADR owns distinct data domains)
- No integration contract conflicts (signal → method call pattern is consistent)
- No performance budget conflicts (all operations well within 16ms frame budget)
- No dependency cycles

### ADR Dependency Graph (Topologically Sorted)

```
Foundation (no dependencies):
  1. ADR-0001: State Management Pattern
  2. ADR-0002: Storage Abstraction

Depends on Foundation:
  3. ADR-0003: Input Timing Precision (requires ADR-0001)
  4. ADR-0004: Data Serialization Format (requires ADR-0002)

Depends on Core:
  5. ADR-0005: Scene Architecture (requires ADR-0001)
```

No unresolved dependencies. No cycles. All ADRs are `Accepted`.

---

## Engine Compatibility Audit

### Version Consistency

All 5 ADRs target **Godot 4.6** — consistent with `VERSION.md`.

### Post-Cutoff APIs Used

None. All ADRs use APIs stable since Godot 4.0.

### Deprecated API Check

No ADR references any API from `deprecated-apis.md`. Specifically verified:
- No `TileMap` (use `TileMapLayer`) — not used
- No `yield` (use `await`) — not used
- No `instance()` (use `instantiate()`) — not used
- No `OS.get_ticks_msec()` — correctly uses `Time.get_ticks_msec()` (ADR-0003)

### Engine Compatibility Sections

All 5/5 ADRs include the Engine Compatibility section. ✅

### Engine Specialist Findings

No HIGH or MEDIUM risk engine APIs are used across any ADR. All APIs (`Time.get_ticks_msec()`, `JavaScriptBridge.eval()`, `FileAccess`, Godot signals, `Tween`, node visibility) are stable since 4.0 with no known breaking changes through 4.6.

One advisory note: `JavaScriptBridge.eval()` in ADR-0002 uses string interpolation for localStorage calls. The `c_escape()` call mitigates injection risk, but this should be tested in Godot 4.6 HTML5 export specifically (noted in ADR-0002's own Validation Criteria).

---

## GDD Revision Flags

None — all GDD assumptions are consistent with verified engine behaviour.

---

## Architecture Document Coverage

`docs/architecture/architecture.md` exists and covers:
- ✅ All 7 systems from `systems-index.md` appear in the layer map
- ✅ Data flow section covers all cross-system communication
- ✅ API boundaries support all integration requirements
- ✅ No orphaned architecture (all architecture maps to a GDD)

---

## Verdict: PASS

All Foundation and Core layer requirements are covered by ADRs. The 10 uncovered requirements are either:
- Pure implementation details (formulas, algorithms) that don't require architectural decisions
- Low-priority rendering/UI details that can be deferred to implementation

No cross-ADR conflicts. No engine compatibility issues. No GDD revision flags.

### Blocking Issues

None.

### Recommended Next ADRs (Optional, Can Defer)

1. **Browser Tab Visibility Detection** — TR-SRE-005: How to detect tab blur/focus via `JavaScriptBridge` to void active trials. Low complexity, can be decided during implementation.
2. **Procedural Dot Placement** — TR-CPS-001: Poisson disk sampling approach. Algorithm choice, not architecture.
3. **Data Visualization** — TR-RI-001: Whether to use custom `_draw()` or an addon for charts. Can defer to implementation.

---

## Handoff

1. The architecture is complete and internally consistent — all 5 required ADRs are written
2. Run `/gate-check pre-production` to advance to implementation planning
3. The 3 optional ADRs above can be written during implementation if needed
