# Control Manifest

> **Engine**: Godot 4.6 (GDScript, Compatibility renderer)
> **Last Updated**: 2026-04-12
> **Manifest Version**: 2026-04-12
> **ADRs Covered**: ADR-0001, ADR-0002, ADR-0003, ADR-0004, ADR-0005
> **Status**: Active — regenerate with `/create-control-manifest update` when ADRs change

`Manifest Version` is the date this manifest was generated. Story files embed
this date when created. `/story-readiness` compares a story's embedded version
to this field to detect stories written against stale rules. Always matches
`Last Updated` — they are the same date, serving different consumers.

This manifest is a programmer's quick-reference extracted from all Accepted ADRs,
technical preferences, and engine reference docs. For the reasoning behind each
rule, see the referenced ADR.

---

## Foundation Layer Rules

*Applies to: event architecture, save/load, storage, data serialization*

### Required Patterns

- **Use Godot's native signal system for all event communication** — source: ADR-0001
- **Use direct method calls via typed references for synchronous data requests** — source: ADR-0001
- **Connect signals in `_ready()` via typed callable syntax: `signal_name.connect(callable)`** — source: ADR-0001
- **StatisticalAnalysisEngine extends `RefCounted`, not `Node`** — no scene tree dependency — source: ADR-0001
- **Receive system references via constructor injection or `@export` node paths** — source: ADR-0001
- **StorageAbstraction detects platform with `OS.has_feature("web")`** — source: ADR-0002
- **Use `c_escape()` on data before embedding in JavaScript eval strings** — source: ADR-0002
- **`save()` returns `false` on failure; caller shows user-facing warning** — source: ADR-0002
- **All session data stored as JSON strings via `JSON.stringify()` / `JSON.parse_string()`** — source: ADR-0004
- **Every stored JSON object includes `schema_version` integer at root level** — source: ADR-0004
- **Schema migration functions are chained: v1→v2→v3, each transforming Dictionary in place** — source: ADR-0004
- **SchemaMigrator extends `RefCounted` for testability** — source: ADR-0004
- **Static config uses custom `Resource` subclasses saved as `.tres` files** — source: ADR-0004
- **Config `.tres` files live in `assets/data/` and are preloaded at scene ready** — source: ADR-0004

### Forbidden Approaches

- **Never use Autoload singletons** — hides dependencies, harder to test — source: ADR-0001
- **Never use string-based `connect("signal", obj, "method")` syntax** — deprecated since 4.0 — source: ADR-0001
- **Never use `CONNECT_DEFERRED` for signal connections** — breaks same-frame guarantee — source: ADR-0001
- **Never use Godot `ConfigFile` for persistence** — INI format, no localStorage access, no JSON export — source: ADR-0002
- **Never use IndexedDB via JavaScriptBridge** — async complexity not justified for < 5MB — source: ADR-0002
- **Never use `.tres` for session data** — cannot be stored in localStorage or exported as readable file — source: ADR-0004
- **Never use JSON for static config** — loses type safety, no preload(), no editor integration — source: ADR-0004
- **Never use binary formats (MessagePack, BSON) for persistence** — not human-readable, violates export requirement — source: ADR-0004

### Performance Guardrails

- **Signal dispatch**: < 0.01ms per signal — source: ADR-0001
- **Save time**: < 50ms (localStorage is synchronous) — source: ADR-0002
- **Load time**: < 50ms — source: ADR-0002
- **Storage**: ~3KB per session, ~5MB total limit (HTML5) — source: ADR-0002
- **JSON serialize**: < 10ms for 50 sessions — source: ADR-0004
- **JSON parse**: < 20ms for 50 sessions — source: ADR-0004
- **Config load**: < 5ms (preloaded) — source: ADR-0004

---

## Core Layer Rules

*Applies to: stimulus-response engine, statistical analysis, color perception, input timing*

### Required Patterns

- **Capture `Time.get_ticks_msec()` at the TOP of `_input()` before any processing** — source: ADR-0003
- **Use `_input()` callback for both stimulus and response timestamps** — source: ADR-0003
- **Display all reaction times with "±5ms" uncertainty band** — fixed display suffix, not computed per-trial — source: ADR-0003
- **Store raw integer RT in trial data** — uncertainty is a display concern, not data concern — source: ADR-0003
- **Use `Time.get_ticks_msec()` (not `Time.get_ticks_usec()`)** — usec offers no real advantage in browsers — source: ADR-0003
- **Always validate `schema_version` before accessing session data fields** — source: ADR-0004

### Forbidden Approaches

- **Never use `_process()` or `_physics_process()` for timing measurements** — adds up to 16ms delay — source: ADR-0003
- **Never use `Time.get_ticks_usec()` for display** — sub-ms precision is false precision in browsers (±5ms floor) — source: ADR-0003
- **Never use `JavaScriptBridge` to call `Performance.now()`** — bridge call adds latency, cure is worse than disease — source: ADR-0003

### Performance Guardrails

- **Input capture time**: < 0.1ms per `Time.get_ticks_msec()` call — within 2ms input budget — source: ADR-0003

---

## Feature Layer Rules

*Applies to: results interpretation, session persistence, test sequencing*

### Required Patterns

- **Use a single persistent root scene with node visibility swapping for all screen transitions** — source: ADR-0005
- **`transition_to()` is the single entry point for all screen changes** — source: ADR-0005
- **Systems node children persist across all transitions — never freed during gameplay** — source: ADR-0005
- **Screens use `set_process(false)` when hidden to avoid unnecessary work** — source: ADR-0005
- **TransitionOverlay renders via CanvasLayer above all screen content** — source: ADR-0005
- **Desktop export opens user data folder after writing (for discoverability)** — source: ADR-0002

### Forbidden Approaches

- **Never use `SceneTree.change_scene_to_packed()` or `change_scene_to_file()`** — destroys scene tree, breaks system persistence — source: ADR-0005
- **Never use additive scene loading (instantiate/free per screen)** — unnecessary complexity for 4 screens — source: ADR-0005

### Performance Guardrails

- **Transition time**: < 1ms (visibility toggle) — source: ADR-0005
- **Memory (all screens loaded)**: ~2MB — within 64MB ceiling — source: ADR-0005
- **Startup load**: all screens instantiated at launch, < 500ms — source: ADR-0005

---

## Presentation Layer Rules

*Applies to: feedback system, UI rendering, audio, animations*

### Required Patterns

- **Feedback visual/audio must appear in the same render frame as input** — 0 additional frames — source: ADR-0001
- **Signal callbacks execute synchronously (Godot default)** — ensures same-frame guarantee — source: ADR-0001
- **Only one screen visible at a time under the Screens node** — source: ADR-0005
- **AudioStreamPlayers live under AudioPlayers node, always active** — source: ADR-0005

### Forbidden Approaches

- **Never add frame-delayed feedback (deferred signals, next-frame callbacks)** — violates Pillar 2 "Every Tap Feels Crisp" — source: ADR-0001

---

## Global Rules (All Layers)

### Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Classes | PascalCase | `StimulusResponseEngine` |
| Variables | snake_case | `_stimulus_timestamp` |
| Signals | snake_case, past tense | `response_recorded` |
| Files | snake_case | `stimulus_response_engine.gd` |
| Scenes | snake_case | `main_menu_screen.tscn` |
| Constants | SCREAMING_SNAKE_CASE | `CURRENT_VERSION` |

### Performance Budgets

| Target | Value |
|--------|-------|
| Framerate | 60 fps |
| Frame budget | 16ms |
| Memory ceiling | 64MB |
| Input budget | 2ms (from input event to timestamp capture) |

### Approved Libraries / Addons

- None configured yet — all systems use engine-native APIs only

### Forbidden APIs (Godot 4.6)

These APIs are deprecated or renamed — never use them:

| Deprecated | Use Instead | Source |
|------------|-------------|--------|
| `TileMap` | `TileMapLayer` | deprecated-apis.md |
| `yield()` | `await signal` | deprecated-apis.md |
| `connect("signal", obj, "method")` | `signal.connect(callable)` | deprecated-apis.md |
| `instance()` | `instantiate()` | deprecated-apis.md |
| `OS.get_ticks_msec()` | `Time.get_ticks_msec()` | deprecated-apis.md |
| `duplicate()` for nested resources | `duplicate_deep()` | deprecated-apis.md |
| `$NodePath` in `_process()` | `@onready var` cached reference | deprecated-apis.md |
| Untyped `Array` / `Dictionary` | `Array[Type]`, typed variables | deprecated-apis.md |
| `Texture2D` in shader parameters | `Texture` base type | deprecated-apis.md |

### Cross-Cutting Constraints

- **No Autoload singletons anywhere in the project** — source: ADR-0001
- **All gameplay values must be data-driven (external config), never hardcoded** — source: coding-standards
- **All public methods must be unit-testable (dependency injection over singletons)** — source: coding-standards
- **JSON for mutable player data, `.tres` for immutable config — never mix formats** — source: ADR-0004
- **All signal connections use typed callable syntax** — source: ADR-0001
