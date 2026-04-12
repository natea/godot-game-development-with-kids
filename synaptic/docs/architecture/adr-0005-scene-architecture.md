# ADR-0005: Scene Architecture — In-Scene Node Swapping

## Status

Accepted

## Date

2026-04-12

## Last Verified

2026-04-12

## Decision Makers

Technical Director, Game Designer

## Summary

Synaptic uses a single persistent scene with node visibility swapping for all screen transitions (menu, test modules, results). We rejected `SceneTree.change_scene_to_packed()` because Synaptic's systems (SRE, Stats, Feedback) must persist across screens, and scene changes would destroy and recreate them. Node swapping keeps all systems alive while changing only the visible content layer.

## Engine Compatibility

| Field | Value |
|-------|-------|
| **Engine** | Godot 4.6 |
| **Domain** | Core / Scene Management |
| **Knowledge Risk** | LOW — Node visibility and scene tree manipulation unchanged since 4.0 |
| **References Consulted** | `docs/engine-reference/godot/modules/ui.md` |
| **Post-Cutoff APIs Used** | None |
| **Verification Required** | None |

## ADR Dependencies

| Field | Value |
|-------|-------|
| **Depends On** | ADR-0001 (State Management) |
| **Enables** | Test Sequencing implementation, all screen transitions |
| **Blocks** | None |

## Context

### Problem Statement

Synaptic has four distinct screens: Main Menu, Reaction Time Test, Color Perception Test, and Results. The Test Sequencing system orchestrates transitions between them. We need to decide whether to use Godot's scene-change API or manage screens as nodes within a single scene.

### Constraints

- StimulusResponseEngine, StatisticalAnalysisEngine, and SessionPersistence must persist across screen transitions (they accumulate state during a test battery)
- FeedbackSystem tweens must not be interrupted by scene changes
- Transitions between modules happen mid-battery (RT → CPS) — systems must survive
- HTML5 export: scene loading causes visible frame drops in browsers
- The game has only 4 screens — minimal complexity

### Requirements

- Scene transitions between menu, test, and results screens (TR-TS-001)
- Countdown timer with skip-on-input for module transitions (TR-TS-002)
- Systems persist across module transitions (architectural requirement from ADR-0001)

## Decision

Use a **single persistent root scene** with screen nodes swapped via visibility. TestSequencing owns the transition logic, showing/hiding screen containers. All persistent systems are children of the root scene and are never freed during gameplay.

### Scene Tree Structure

```
GameRoot (Node)
├── Systems (Node) — always active, never hidden
│   ├── StimulusResponseEngine (Node)
│   ├── FeedbackSystem (Node)
│   └── SessionPersistence (Node)
│   # StatisticalAnalysisEngine is RefCounted, not in tree
│
├── Screens (Node) — only one child visible at a time
│   ├── MainMenuScreen (Control) — visible: true/false
│   ├── ReactionTimeScreen (Control) — visible: true/false
│   ├── ColorPerceptionScreen (Control) — visible: true/false
│   └── ResultsScreen (Control) — visible: true/false
│
├── Overlays (CanvasLayer) — always on top
│   ├── TransitionOverlay (Control) — countdown, fade
│   └── ModalOverlay (Control) — warnings, confirmations
│
└── AudioPlayers (Node) — always active
    ├── TapAudioPlayer (AudioStreamPlayer)
    └── UIAudioPlayer (AudioStreamPlayer)
```

### Transition Logic

```gdscript
# In TestSequencing:
var _current_screen: Control = null

func transition_to(screen_name: StringName) -> void:
    if _current_screen:
        _current_screen.visible = false
    _current_screen = _screens[screen_name]
    _current_screen.visible = true
    screen_changed.emit(screen_name)

func transition_with_countdown(screen_name: StringName, duration: float = 3.0) -> void:
    _transition_overlay.start_countdown(duration)
    await _transition_overlay.countdown_finished
    transition_to(screen_name)
```

### Implementation Guidelines

- All screens are children of the `Screens` node — only one is visible at a time
- `transition_to()` is the single entry point for all screen changes
- Systems node is never hidden — `_process()` continues for all systems
- Screens connect to system signals in `_ready()` once and stay connected
- Screen nodes use `set_process(false)` when hidden to avoid unnecessary work
- TransitionOverlay renders above all screens via CanvasLayer
- No `change_scene_to_packed()` or `change_scene_to_file()` anywhere in the project

### Screen Lifecycle

| Event | What Happens |
|-------|-------------|
| Screen shown | `visible = true`, `set_process(true)`, screen receives `screen_entered` signal |
| Screen hidden | `visible = false`, `set_process(false)`, screen receives `screen_exited` signal |
| Battery starts | MainMenu hidden, first test screen shown, systems begin recording |
| Module transition | Current test screen hidden, countdown overlay shown, next test screen shown |
| Battery ends | Last test screen hidden, ResultsScreen shown with accumulated data |

## Alternatives Considered

### Alternative 1: SceneTree.change_scene_to_packed()

- **Description**: Use Godot's built-in scene change for each screen
- **Pros**: Clean separation, each screen is an independent scene, familiar Godot pattern
- **Cons**: Destroys the current scene tree — all systems would need to be Autoloads to survive; systems lose signal connections on each change; causes frame drops in HTML5 exports
- **Rejection Reason**: Synaptic's systems accumulate state across a multi-module battery. Scene changes would force every system into an Autoload singleton, violating ADR-0001's "no Autoload singletons" rule. The cure creates worse problems than the disease.

### Alternative 2: Additive Scene Loading (add_child of packed scenes)

- **Description**: Load screen scenes additively and free them when done
- **Pros**: Screens are separate scenes (cleaner files), root persists
- **Cons**: Requires managing scene instantiation/freeing, signal reconnection on each load, memory allocation churn
- **Rejection Reason**: With only 4 screens, the overhead of loading/freeing scenes is unnecessary. All screens fit comfortably in memory simultaneously (~2MB total). Node swapping is simpler and avoids reconnection logic.

## Consequences

### Positive

- Zero frame drops on transition — visibility toggle is instant
- All systems persist naturally — no Autoload hacks needed
- Signal connections established once in `_ready()` and never broken
- Simple mental model — one scene, swap what's visible

### Negative

- All screens are in memory simultaneously (~2MB, well within 64MB budget)
- Single scene file could become large — mitigated by using `@export` node paths and separate scripts
- Screens must manually disable processing when hidden

## Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| Scene file becomes unwieldy with all screens | Low | Low | Each screen is a separate scene instanced as a child; only the root layout is one file |
| Hidden screens accidentally processing | Medium | Low | `set_process(false)` in `screen_exited`; enforce in code review |

## Performance Implications

| Metric | Before | Expected After | Budget |
|--------|--------|---------------|--------|
| Transition time | N/A | < 1ms (visibility toggle) | 16ms (one frame) |
| Memory (all screens loaded) | N/A | ~2MB | 64MB ceiling |
| Startup load | N/A | All screens instantiated at launch | < 500ms |

## Validation Criteria

- [ ] All screen transitions use `transition_to()` — no `change_scene` calls in project
- [ ] Systems node children persist across all transitions (verified by checking node references)
- [ ] Hidden screens do not run `_process()` (verified by profiler or debug print)
- [ ] Transition countdown overlay renders above all screen content
- [ ] Signal connections survive full battery cycle (Menu → RT → CPS → Results → Menu)

## GDD Requirements Addressed

| GDD Document | System | Requirement | How This ADR Satisfies It |
|-------------|--------|-------------|--------------------------|
| test-sequencing.md | TS | TR-TS-001: Scene transitions between screens | Node visibility swapping within persistent scene |
| test-sequencing.md | TS | TR-TS-002: Countdown timer for transitions | TransitionOverlay with countdown, skip-on-input |

## Related

- ADR-0001 (State Management) — signal connections persist because nodes are never freed
- ADR-0002 (Storage Abstraction) — SessionPersistence lives in Systems node, always available
