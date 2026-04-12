# ADR-0001: State Management Pattern — Signal-Based Decoupled Architecture

## Status

Accepted

## Date

2026-04-12

## Last Verified

2026-04-12

## Decision Makers

Technical Director, Game Designer

## Summary

Synaptic needs a communication pattern between its 7 systems that keeps them decoupled while ensuring same-frame responsiveness for input feedback. We chose Godot's native signal system for event broadcasting and direct method calls via injected references for synchronous data requests.

## Engine Compatibility

| Field | Value |
|-------|-------|
| **Engine** | Godot 4.6 |
| **Domain** | Core / Scripting |
| **Knowledge Risk** | LOW — signal system unchanged since 4.0 |
| **References Consulted** | `docs/engine-reference/godot/modules/input.md` |
| **Post-Cutoff APIs Used** | None |
| **Verification Required** | None |

## ADR Dependencies

| Field | Value |
|-------|-------|
| **Depends On** | None |
| **Enables** | ADR-0002, ADR-0003, ADR-0005 |
| **Blocks** | All implementation epics |
| **Ordering Note** | Must be accepted before any system implementation begins |

## Context

### Problem Statement

Synaptic has 7 interconnected systems. The Stimulus-Response Engine emits events that Feedback, Stats, and Test Sequencing all consume. The Feedback System must render in the same frame as input capture. We need a communication pattern that is both decoupled (systems don't know about each other's internals) and synchronous (no frame delay between event and response).

### Current State

No implementation exists. This is a greenfield decision.

### Constraints

- Same-frame guarantee: Feedback must render in the same `_process()` frame as input (Pillar 2)
- Godot signals are synchronous by default (callbacks execute inline)
- Systems must be independently testable
- Solo developer — no bus infrastructure worth maintaining separately

### Requirements

- Event broadcasting without tight coupling (TR-SRE-006)
- Same-frame visual/audio response to input (TR-FB-001, TR-FB-002)
- State machine transitions triggered by events (TR-SRE-003, TR-CPS-002)
- Data retrieval from other systems (Stats reads from SessionPersistence)

## Decision

Use **Godot's native signal system** for all event communication and **direct method calls via typed references** for synchronous data requests.

### Architecture

```
StimulusResponseEngine
  ├── signal response_recorded ──→ FeedbackSystem (same frame)
  ├── signal module_completed  ──→ TestSequencing (orchestration)
  │                            ──→ StatisticalAnalysisEngine (computation)
  └── signal early_response    ──→ FeedbackSystem (same frame)

StatisticalAnalysisEngine
  └── called directly by TestSequencing: analyze_module(data) -> result
      called directly by ResultsInterpretation: for display data

SessionPersistence
  └── called directly by TestSequencing: save_session(data)
      called directly by ResultsInterpretation: load_history()
```

### Key Interfaces

```gdscript
# Event pattern (fire-and-forget, no return value)
signal response_recorded(rt_ms: int, trial_index: int, is_outlier: bool)

# Data request pattern (synchronous, typed return)
var _stats_engine: StatisticalAnalysisEngine
func _get_analysis() -> Dictionary:
    return _stats_engine.analyze_module(_trial_data)
```

### Implementation Guidelines

- Systems connect to signals in `_ready()` via typed callable connections
- No string-based `connect()` — use `signal_name.connect(callable)` pattern
- StatisticalAnalysisEngine extends `RefCounted`, not `Node` — no scene tree dependency
- Systems receive references via constructor injection or `@export` node paths
- No Autoload singletons — all references are explicit
- Signal callbacks execute synchronously (Godot default) — do not use `CONNECT_DEFERRED`

## Alternatives Considered

### Alternative 1: Centralized Event Bus (Autoload)

- **Description**: A global Autoload singleton that all systems emit to and subscribe from
- **Pros**: Single connection point, easy to add new listeners
- **Cons**: Global state, harder to test, overkill for 7 systems, hides dependencies
- **Rejection Reason**: Synaptic has only 7 systems with clear, static relationships. An event bus adds indirection without benefit. Direct signal connections are more explicit and equally decoupled.

### Alternative 2: Observer Pattern with Custom EventManager

- **Description**: Custom observer pattern implementation managing subscriptions
- **Pros**: Full control over dispatch order and filtering
- **Cons**: Reinventing what Godot signals already provide, maintenance overhead
- **Rejection Reason**: Godot signals ARE the observer pattern, natively integrated with the engine.

## Consequences

### Positive

- Zero additional infrastructure — uses engine primitives only
- Signal connections are visible in code (explicit `connect()` calls)
- StatisticalAnalysisEngine is trivially unit-testable (RefCounted, no scene tree)
- Same-frame guarantee is automatic (Godot signals are synchronous by default)

### Negative

- Signal connection order determines callback execution order (tree order)
- No built-in event replay or logging (acceptable for this scope)

### Neutral

- Systems must be in the scene tree to emit/receive signals (except RefCounted pure-data systems)

## Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| Signal connection order causes subtle bugs | Low | Medium | Document that listeners are order-independent |
| Adding a 3rd test module requires new signal connections | Medium | Low | Connection setup is trivial — 2-3 lines per listener |

## Performance Implications

| Metric | Before | Expected After | Budget |
|--------|--------|---------------|--------|
| CPU (signal dispatch) | N/A | < 0.01ms per signal | 2ms frame budget for input |
| Memory | N/A | ~0 overhead | 64MB ceiling |

## Migration Plan

N/A — greenfield implementation.

## Validation Criteria

- [ ] Feedback visual/audio appears in same render frame as input (0 additional frames)
- [ ] StatisticalAnalysisEngine can be instantiated and tested without a SceneTree
- [ ] No Autoload singletons in the project
- [ ] All signal connections use typed callable syntax (no string-based connect)

## GDD Requirements Addressed

| GDD Document | System | Requirement | How This ADR Satisfies It |
|-------------|--------|-------------|--------------------------|
| stimulus-response-engine.md | SRE | TR-SRE-006: Signal emission for trial events | Native Godot signals for all trial events |
| feedback-system.md | Feedback | TR-FB-001: Same-frame visual response | Synchronous signal callbacks ensure same-frame |
| feedback-system.md | Feedback | TR-FB-002: Same-frame audio response | Synchronous signal callbacks ensure same-frame |
| stimulus-response-engine.md | SRE | TR-SRE-003: State machine | State transitions are internal; signals broadcast state changes |
| color-perception-system.md | CPS | TR-CPS-002: Extends SRE state machine | CPS extends SRE class, inheriting signal definitions |

## Related

- ADR-0005 (Scene Architecture) — determines how systems are arranged in the scene tree, affecting signal connection setup
