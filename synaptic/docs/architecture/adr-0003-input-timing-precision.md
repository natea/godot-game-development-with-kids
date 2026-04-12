# ADR-0003: Input Timing Precision — Time.get_ticks_msec() with Uncertainty Band

## Status

Accepted

## Date

2026-04-12

## Last Verified

2026-04-12

## Decision Makers

Technical Director, Systems Designer

## Summary

Synaptic measures reaction time to millisecond precision. We chose `Time.get_ticks_msec()` captured in the `_input()` callback as the timing source, with a displayed ±5ms uncertainty band to honestly represent browser measurement limitations.

## Engine Compatibility

| Field | Value |
|-------|-------|
| **Engine** | Godot 4.6 |
| **Domain** | Input / Core |
| **Knowledge Risk** | LOW — `Time.get_ticks_msec()` unchanged since 4.0 |
| **References Consulted** | `docs/engine-reference/godot/modules/input.md`, `docs/engine-reference/godot/deprecated-apis.md` |
| **Post-Cutoff APIs Used** | None |
| **Verification Required** | Benchmark actual precision of `Time.get_ticks_msec()` in Godot 4.6 HTML5 export across browsers |

## ADR Dependencies

| Field | Value |
|-------|-------|
| **Depends On** | ADR-0001 (State Management) |
| **Enables** | All SRE and Stats implementation |
| **Blocks** | Stimulus-Response Engine implementation |

## Context

### Problem Statement

Reaction time measurement requires the highest timing precision available. In browser environments, JavaScript event loop quantization limits timing to approximately ±5ms. Godot's HTML5 export adds its own processing overhead. We need to select the right timing API and communicate measurement uncertainty honestly to the player.

### Constraints

- Browser event loop quantization: ~5ms floor on timing precision
- `Time.get_ticks_msec()` returns integer milliseconds (no sub-ms precision)
- `Time.get_ticks_usec()` returns microseconds but browser precision is still ~5ms
- `_input()` fires before `_process()` — earliest possible callback for input events
- Published RT norms used laboratory hardware with ~1ms precision

### Requirements

- Capture input timestamp as close to the hardware event as possible (TR-SRE-001)
- Use a reliable, stable timing API (TR-SRE-002)
- Display measurement uncertainty to maintain scientific honesty (Pillar 1)

## Decision

Use `Time.get_ticks_msec()` in the `_input()` callback for both stimulus and response timestamps. Display all reaction times with a "±5ms" uncertainty band. Do NOT use `_process()` or `_physics_process()` — `_input()` fires earlier in the frame and is closest to the actual input event.

### Key Interfaces

```gdscript
# In StimulusResponseEngine:
var _stimulus_timestamp: int = 0
var _response_timestamp: int = 0

func _show_stimulus() -> void:
    _stimulus_timestamp = Time.get_ticks_msec()
    _stimulus_node.visible = true
    _state = State.STIMULUS_PRESENTED

func _input(event: InputEvent) -> void:
    if _state != State.STIMULUS_PRESENTED:
        return
    if event is InputEventMouseButton and event.pressed:
        _response_timestamp = Time.get_ticks_msec()
        _record_response()
    elif event is InputEventKey and event.pressed:
        _response_timestamp = Time.get_ticks_msec()
        _record_response()

func _record_response() -> void:
    var rt_ms: int = _response_timestamp - _stimulus_timestamp
    response_recorded.emit(rt_ms, _trial_index, rt_ms > _timeout_threshold)
```

### Implementation Guidelines

- Always capture `Time.get_ticks_msec()` at the TOP of `_input()` before any processing
- Never use `_process()` or `_physics_process()` for timing — they add up to 16ms of delay
- Display RT as "247ms ±5ms" — the ±5ms is a fixed display suffix, not computed per-trial
- Store raw integer RT in trial data — uncertainty is a display concern, not data concern
- `Time.get_ticks_usec()` offers no real advantage in browsers (still ~5ms precision); use msec for simplicity

## Alternatives Considered

### Alternative 1: Time.get_ticks_usec() for Higher Precision

- **Description**: Use microsecond timing for sub-millisecond measurements
- **Pros**: Theoretically higher precision, could show tenths of ms
- **Cons**: Browser event loop still quantizes to ~5ms; displaying sub-ms precision would be false precision
- **Rejection Reason**: Showing 247.392ms when the actual precision is ±5ms would violate Pillar 1 (Measure, Don't Guess). The display precision must not exceed measurement precision.

### Alternative 2: JavaScript Performance.now() via JavaScriptBridge

- **Description**: Use browser's `performance.now()` API for sub-ms precision
- **Pros**: Potentially higher precision than Godot's timing in HTML5
- **Cons**: Cross-frame JavaScript calls add latency, async overhead, platform-specific code in the hot path
- **Rejection Reason**: Adding a JavaScript bridge call in `_input()` would ADD latency instead of reducing it. The cure is worse than the disease.

## Consequences

### Positive

- Simple, reliable implementation using a single stable API
- Honest uncertainty communication builds player trust (Pillar 1)
- No platform-specific code in the timing-critical path

### Negative

- Cannot achieve sub-millisecond precision in browser (inherent platform limitation)
- Published lab norms have ~1ms precision; our ~5ms precision means percentile placement has ~2-3 percentile points of uncertainty

## Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| Actual HTML5 precision is worse than ±5ms in some browsers | Medium | Medium | Benchmark Chrome/Firefox/Safari; adjust uncertainty band if needed |
| Godot 4.6 HTML5 export adds unexpected timing overhead | Low | High | Prototype and benchmark before full implementation |

## Performance Implications

| Metric | Before | Expected After | Budget |
|--------|--------|---------------|--------|
| Input capture time | N/A | < 0.1ms (`Time.get_ticks_msec()` call) | 2ms input budget |

## Validation Criteria

- [ ] RT measurement is captured in `_input()`, not `_process()`
- [ ] `Time.get_ticks_msec()` is called at the top of `_input()` before any other processing
- [ ] All displayed RTs include the "±5ms" uncertainty band
- [ ] Benchmark: 100 trials in HTML5 export show timing distribution consistent with ±5ms quantization

## GDD Requirements Addressed

| GDD Document | System | Requirement | How This ADR Satisfies It |
|-------------|--------|-------------|--------------------------|
| stimulus-response-engine.md | SRE | TR-SRE-001: Capture input timestamp in same frame | `_input()` callback with `Time.get_ticks_msec()` |
| stimulus-response-engine.md | SRE | TR-SRE-002: Sub-frame timing API | `Time.get_ticks_msec()` provides ms-level precision |

## Related

- ADR-0001 (State Management) — signals used to broadcast the captured RT
