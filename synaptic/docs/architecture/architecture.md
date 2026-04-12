# Synaptic — Master Architecture

## Document Status

- Version: 1
- Last Updated: 2026-04-12
- Engine: Godot 4.6 (GDScript, Compatibility renderer)
- Target Platforms: Web (HTML5) + PC Desktop
- GDDs Covered: stimulus-response-engine, statistical-analysis-engine, color-perception-system, feedback-system, results-interpretation, session-persistence, test-sequencing
- ADRs Referenced: ADR-0001 (State Management), ADR-0002 (Storage Abstraction), ADR-0003 (Input Timing), ADR-0004 (Data Serialization), ADR-0005 (Scene Architecture)
- Architecture Review: 2026-04-12 — PASS
- Technical Director Sign-Off: 2026-04-12 — APPROVED
- Lead Programmer Feasibility: FEASIBLE (Lean mode — skipped formal review)

---

## Engine Knowledge Gap Summary

**Engine**: Godot 4.6 | **LLM Training Covers**: ~4.3 | **Post-Cutoff**: 4.4, 4.5, 4.6

### HIGH RISK Domains

- **UI/Focus**: Dual-focus system in 4.6 separates mouse/touch from keyboard/gamepad focus. Synaptic is mouse/keyboard only — dual-focus won't cause issues but must not be assumed to behave like pre-4.6 focus.
- **GDScript**: Variadic arguments (`...`) and `@abstract` decorator added in 4.5. Available for use but not in LLM training data.

### MEDIUM RISK Domains

- **Rendering**: Glow processes before tonemapping in 4.6. Irrelevant for Synaptic (no glow effects). D3D12 default on Windows — desktop export only, not a factor for HTML5.
- **Core**: Quaternion identity initialization change in 4.6. Irrelevant (no 3D/quaternions in Synaptic).

### LOW RISK Domains (game-critical systems)

- **Input**: API unchanged since 4.0. `_input()`, `_unhandled_input()`, `InputEvent` classes all stable. ✅
- **Audio**: No breaking changes in 4.4–4.6. `AudioStreamPlayer`, bus system, pooling all stable. ✅
- **Time**: `Time.get_ticks_msec()` stable since 4.0 (moved from `OS`). ✅
- **2D Rendering**: No changes affecting Synaptic's flat 2D UI rendering. ✅

### Systems touching HIGH/MEDIUM risk domains

None. Synaptic uses only LOW RISK engine domains (input, audio, time, 2D UI).

---

## Technical Requirements Baseline

Extracted from 7 GDDs | 28 total requirements

| Req ID | GDD | System | Requirement | Domain |
|--------|-----|--------|-------------|--------|
| TR-SRE-001 | stimulus-response-engine.md | SRE | Capture input timestamp via `_input()` within same frame | Input |
| TR-SRE-002 | stimulus-response-engine.md | SRE | `Time.get_ticks_msec()` for sub-frame timing | Core |
| TR-SRE-003 | stimulus-response-engine.md | SRE | State machine: IDLE→WAITING→STIMULUS→RESPONSE→COMPLETE | Core |
| TR-SRE-004 | stimulus-response-engine.md | SRE | Uniform random delay generation (1500–4000ms) | Core |
| TR-SRE-005 | stimulus-response-engine.md | SRE | Browser tab focus/blur detection | Platform |
| TR-SRE-006 | stimulus-response-engine.md | SRE | Signal emission for trial events | Core |
| TR-STAT-001 | statistical-analysis-engine.md | Stats | Median, IQR, percentile computation | Core |
| TR-STAT-002 | statistical-analysis-engine.md | Stats | Linear regression for trend detection | Core |
| TR-STAT-003 | statistical-analysis-engine.md | Stats | Browser-offset correction subtraction | Core |
| TR-STAT-004 | statistical-analysis-engine.md | Stats | Reference distribution table (external config) | Data |
| TR-CPS-001 | color-perception-system.md | CPS | Ishihara plate rendering (Poisson disk dot placement) | Rendering |
| TR-CPS-002 | color-perception-system.md | CPS | Extends SRE state machine (replaces stimulus type) | Core |
| TR-CPS-003 | color-perception-system.md | CPS | Curated plate color definitions (stim-* tokens) | Data |
| TR-CPS-004 | color-perception-system.md | CPS | CVD miss rate computation | Core |
| TR-FB-001 | feedback-system.md | Feedback | Same-frame visual response (scale pulse, ring) | Rendering |
| TR-FB-002 | feedback-system.md | Feedback | Audio playback within same frame as input | Audio |
| TR-FB-003 | feedback-system.md | Feedback | Tween animations (spring curve, fading ring) | Rendering |
| TR-FB-004 | feedback-system.md | Feedback | Performance-scaled ring speed | Core |
| TR-RI-001 | results-interpretation.md | R&I | Data visualization (time-series chart) | Rendering |
| TR-RI-002 | results-interpretation.md | R&I | Multi-screen navigation (RT/CVD/History tabs) | UI |
| TR-RI-003 | results-interpretation.md | R&I | Anchored language text from external config | Data |
| TR-RI-004 | results-interpretation.md | R&I | Spectrum bar with positioned marker | Rendering |
| TR-SP-001 | session-persistence.md | SP | localStorage via JavaScript bridge (HTML5) | Platform |
| TR-SP-002 | session-persistence.md | SP | `user://` file storage (desktop) | Platform |
| TR-SP-003 | session-persistence.md | SP | JSON serialization/deserialization | Core |
| TR-SP-004 | session-persistence.md | SP | Browser file download for JSON export | Platform |
| TR-SP-005 | session-persistence.md | SP | Schema versioning and forward migration | Core |
| TR-TS-001 | test-sequencing.md | TS | Scene transitions between menu/test/results | UI |
| TR-TS-002 | test-sequencing.md | TS | Countdown timer with skip-on-input | UI |

---

## System Layer Map

```
┌─────────────────────────────────────────────────────────┐
│  PRESENTATION LAYER                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ Feedback      │  │ Results &    │  │ Test         │  │
│  │ System        │  │ Interpretation│  │ Sequencing   │  │
│  │ (juice/SFX)   │  │ (dashboard)  │  │ (flow/menu)  │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  │
├─────────┼──────────────────┼──────────────────┼─────────┤
│  FEATURE LAYER             │                  │         │
│  ┌──────┴───────┐  ┌──────┴───────┐  ┌──────┴───────┐  │
│  │ Color        │  │ Session      │  │              │  │
│  │ Perception   │  │ Persistence  │  │              │  │
│  │ System       │  │ (storage)    │  │              │  │
│  └──────┬───────┘  └──────┬───────┘  └──────────────┘  │
├─────────┼──────────────────┼────────────────────────────┤
│  CORE LAYER                │                            │
│  ┌──────┴───────┐  ┌──────┴───────┐                    │
│  │ Stimulus-    │  │ Statistical  │                    │
│  │ Response     │  │ Analysis     │                    │
│  │ Engine       │  │ Engine       │                    │
│  └──────┬───────┘  └──────────────┘                    │
├─────────┼──────────────────────────────────────────────┤
│  FOUNDATION LAYER                                      │
│  ┌──────┴───────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ Input        │  │ Storage      │  │ Config       │  │
│  │ Capture      │  │ Abstraction  │  │ Loader       │  │
│  │ (timing)     │  │ (local/web)  │  │ (JSON/tres)  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
├─────────────────────────────────────────────────────────┤
│  PLATFORM LAYER                                        │
│  Godot 4.6 · GDScript · Compatibility Renderer         │
│  HTML5 Export · Browser APIs · user:// filesystem       │
└─────────────────────────────────────────────────────────┘
```

### Layer Assignments

| System | Layer | Rationale |
|--------|-------|-----------|
| Input Capture | Foundation | Wraps `_input()` and `Time.get_ticks_msec()` — pure engine integration |
| Storage Abstraction | Foundation | Abstracts localStorage (HTML5) vs `user://` (desktop) behind unified API |
| Config Loader | Foundation | Loads tuning knobs, reference tables, interpretation text from external files |
| Stimulus-Response Engine | Core | The atomic measurement unit — all test modules extend it |
| Statistical Analysis Engine | Core | Pure computation — no UI, no input, no engine dependencies beyond math |
| Color Perception System | Feature | Extends SRE with Ishihara-specific stimulus and evaluation logic |
| Session Persistence | Feature | Uses Storage Abstraction to save/load session history |
| Feedback System | Presentation | Visual/audio effects triggered by Core layer events |
| Results & Interpretation | Presentation | Dashboard UI, charts, interpretive text display |
| Test Sequencing | Presentation | Scene flow, menus, transitions, onboarding |

---

## Module Ownership Map

### Foundation Layer

| Module | Owns | Exposes | Consumes | Engine APIs |
|--------|------|---------|----------|-------------|
| **InputCapture** | Input timestamps, input state | `get_response_timestamp() -> int`, input event signals | Raw Godot input events | `_input(event)`, `Time.get_ticks_msec()` ✅ stable |
| **StorageAbstraction** | Platform detection, read/write routing | `save(key, data) -> bool`, `load(key) -> Variant`, `export_file(data, filename)` | Platform environment | `JavaScriptBridge` (HTML5), `FileAccess` (desktop) ✅ stable |
| **ConfigLoader** | Config file paths, cached config data | `get_config(section, key) -> Variant`, `get_table(name) -> Array` | `.tres` or `.json` config files from `assets/data/` | `ResourceLoader`, `JSON.parse_string()` ✅ stable |

### Core Layer

| Module | Owns | Exposes | Consumes | Engine APIs |
|--------|------|---------|----------|-------------|
| **StimulusResponseEngine** | Trial state machine, trial data array, timing state | Signals: `response_recorded`, `early_response`, `timeout`, `module_completed`. Methods: `start_module(config)`, `get_trial_data()` | InputCapture timestamps | `Timer` node, custom signals ✅ stable |
| **StatisticalAnalysisEngine** | Computation results, reference tables | `analyze_module(data) -> AnalysisResult`, `analyze_cvd(data) -> CVDResult`, `compute_trend(sessions) -> TrendResult` | ConfigLoader for reference tables, raw trial/plate data | None (pure GDScript math) ✅ |

### Feature Layer

| Module | Owns | Exposes | Consumes | Engine APIs |
|--------|------|---------|----------|-------------|
| **ColorPerceptionSystem** | Plate definitions, plate rendering, correctness evaluation | Signals: `plate_responded`. Methods: `start_module(config)` (overrides SRE) | SRE state machine (extends), ConfigLoader for plate colors | `CanvasItem.draw_*()` for plate rendering ✅ stable |
| **SessionPersistence** | Session records, schema version | `save_session(stats)`, `load_history() -> Array[SessionRecord]`, `export_json()`, `get_session_count() -> int` | StorageAbstraction, StatisticalAnalysisEngine output | None (uses StorageAbstraction) |

### Presentation Layer

| Module | Owns | Exposes | Consumes | Engine APIs |
|--------|------|---------|----------|-------------|
| **FeedbackSystem** | Active tweens, audio players | Signal connections only (no public API) | SRE signals, CPS signals | `Tween`, `AudioStreamPlayer` ✅ stable |
| **ResultsInterpretation** | Results screen nodes, chart rendering | `show_results(analysis)`, navigation signals | StatisticalAnalysisEngine output, SessionPersistence history, ConfigLoader for text | `Control` nodes, `CanvasItem.draw_*()` for charts ✅ stable |
| **TestSequencing** | Scene flow state, onboarding state | `start_battery()`, `start_module(type)`, navigation signals | SRE/CPS module_completed signals, SessionPersistence session count | `SceneTree.change_scene_to_packed()` or node swapping ✅ stable |

---

## Data Flow

### 1. Frame Update Path: Stimulus → Response → Feedback

```
Player Input (_input callback)
  │
  ▼
InputCapture: timestamp = Time.get_ticks_msec()
  │
  ▼
StimulusResponseEngine._input(event)
  ├── State == WAITING? → early_response signal → FeedbackSystem (amber flash)
  ├── State == STIMULUS_PRESENTED? → compute RT = timestamp - stimulus_timestamp
  │     ├── response_recorded signal ──→ FeedbackSystem (pop + ring + RT number)
  │     └── trial_data appended
  └── State == other? → ignore
```

**All synchronous within single frame.** No deferred calls, no async. Feedback
renders in the same `_process()` frame that input was captured.

### 2. Module Completion Path

```
StimulusResponseEngine: last trial complete
  │
  ▼ module_completed(trial_data: Array[TrialResult])
  │
  ├──→ StatisticalAnalysisEngine.analyze_module(trial_data)
  │      Returns: AnalysisResult (synchronous — pure math, < 1ms)
  │
  ├──→ TestSequencing: advance to next module or results
  │
  └──→ SessionPersistence.save_session(analysis_result)
         └──→ StorageAbstraction.save("synaptic_session_data", json)
```

**Signal order**: Godot processes signal callbacks in connection order. All
listeners receive the same signal data. No listener depends on another
listener's processing of the same signal — each processes raw data independently.

### 3. Save/Load Path

```
SAVE (after each module):
  StatisticalAnalysisEngine → AnalysisResult
    → SessionPersistence.save_session()
      → Serialize to JSON (Dictionary → JSON.stringify())
        → StorageAbstraction.save()
          ├── HTML5: JavaScriptBridge.eval("localStorage.setItem(...)")
          └── Desktop: FileAccess.store_string() to user://synaptic_data.json

LOAD (on app start):
  TestSequencing._ready()
    → SessionPersistence.load_history()
      → StorageAbstraction.load()
        ├── HTML5: JavaScriptBridge.eval("localStorage.getItem(...)")
        └── Desktop: FileAccess.get_as_text() from user://synaptic_data.json
      → JSON.parse_string() → validate schema version → migrate if needed
```

### 4. Initialization Order

```
1. ConfigLoader (loads tuning knobs, reference tables, plate definitions)
2. StorageAbstraction (detects platform: HTML5 vs desktop)
3. SessionPersistence (loads history via StorageAbstraction)
4. StatisticalAnalysisEngine (loads reference tables via ConfigLoader)
5. TestSequencing (reads session count from SessionPersistence, shows menu/onboarding)
6. StimulusResponseEngine (created when module starts, not at app launch)
7. FeedbackSystem (connects to SRE signals when SRE is created)
```

Foundation modules boot first (1–2), then Feature (3–4), then Presentation (5).
Core modules (SRE, CPS) are instantiated on-demand when a test module starts.

---

## API Boundaries

### InputCapture → StimulusResponseEngine

```gdscript
# InputCapture is not a separate node — SRE handles _input() directly.
# The "InputCapture" foundation concept is implemented as SRE's _input method.

# StimulusResponseEngine.gd
func _input(event: InputEvent) -> void:
    if not _accepting_input:
        return
    if event is InputEventMouseButton and event.pressed:
        _handle_response(Time.get_ticks_msec())
    elif event is InputEventKey and event.pressed:
        _handle_response(Time.get_ticks_msec())
```

### StimulusResponseEngine API

```gdscript
class_name StimulusResponseEngine extends Node

signal response_recorded(rt_ms: int, trial_index: int, is_outlier: bool)
signal early_response()
signal timeout()
signal module_started()
signal module_completed(trial_data: Array[Dictionary])

func start_module(config: Dictionary) -> void: ...
func abort_module() -> void: ...
func get_state() -> StringName: ...
```

### StatisticalAnalysisEngine API

```gdscript
class_name StatisticalAnalysisEngine extends RefCounted

func analyze_module(trial_data: Array[Dictionary]) -> Dictionary: ...
    # Returns: { median_rt, corrected_median_rt, percentile, band, iqr,
    #            valid_trial_count, timeout_count }

func analyze_cvd(plate_results: Array[Dictionary]) -> Dictionary: ...
    # Returns: { protan_deutan_miss_rate, tritan_miss_rate,
    #            cvd_indicator, cumulative_confidence }

func compute_trend(session_medians: Array[int], window: int) -> Dictionary: ...
    # Returns: { slope, direction, label }
```

**Note**: StatisticalAnalysisEngine extends `RefCounted`, not `Node`. It has no
scene tree presence — it is a pure computation object instantiated by whoever
needs it. This keeps it testable without a scene tree.

### StorageAbstraction API

```gdscript
class_name StorageAbstraction extends RefCounted

func save(key: String, data: String) -> bool: ...
func load(key: String) -> String: ...
func is_available() -> bool: ...
func export_file(data: String, filename: String) -> void: ...
func get_platform() -> StringName: ...  # &"html5" or &"desktop"
```

### SessionPersistence API

```gdscript
class_name SessionPersistence extends Node

func save_session(module_result: Dictionary) -> bool: ...
func load_history() -> Array[Dictionary]: ...
func get_session_count() -> int: ...
func export_json() -> void: ...
func get_cumulative_trials(module_type: StringName) -> int: ...
```

### ColorPerceptionSystem API

```gdscript
class_name ColorPerceptionSystem extends StimulusResponseEngine

signal plate_responded(plate_result: Dictionary)

func start_module(config: Dictionary) -> void: ...  # override
func _generate_plate(plate_def: Dictionary) -> void: ...
func _evaluate_response(player_answer: int, correct_answer: int) -> Dictionary: ...
```

### ConfigLoader API

```gdscript
class_name ConfigLoader extends Node

func get_tuning(system: StringName, key: StringName) -> Variant: ...
func get_reference_table(name: StringName) -> Array: ...
func get_text(category: StringName, key: StringName) -> String: ...
```

---

## ADR Audit

No ADRs exist yet. The following are required.

---

## Required ADRs

### Must have before coding starts (Foundation & Core)

1. **ADR-0001: State Management Pattern** — Signal-based decoupled architecture vs. centralized state store. Covers: TR-SRE-006, all inter-system communication.

2. **ADR-0002: Storage Abstraction — localStorage vs user:// Unified API** — How to abstract platform-specific persistence behind a single interface. Covers: TR-SP-001, TR-SP-002, TR-SP-003, TR-SP-004.

3. **ADR-0003: Input Timing Precision Strategy** — `Time.get_ticks_msec()` vs `Time.get_ticks_usec()`, HTML5 precision limitations, measurement uncertainty reporting. Covers: TR-SRE-001, TR-SRE-002.

4. **ADR-0004: Data Serialization Format** — JSON for persistence, `.tres` resources for config, schema versioning strategy. Covers: TR-SP-003, TR-SP-005, TR-STAT-004, TR-RI-003.

5. **ADR-0005: Scene Architecture — Node Swapping vs Scene Changes** — Whether to use `SceneTree.change_scene_to_packed()` or in-scene node visibility/swapping for screen transitions. Covers: TR-TS-001.

### Should have before the relevant system is built

6. **ADR-0006: Ishihara Plate Rendering — Runtime Generation vs Pre-baked** — Runtime `CanvasItem.draw_circle()` Poisson disk sampling vs pre-rendered PNG textures. Covers: TR-CPS-001.

7. **ADR-0007: Chart Rendering — Custom draw_*() vs Addon** — Build time-series charts with `_draw()` calls or use a charting addon. Covers: TR-RI-001, TR-RI-004.

### Can defer to implementation

8. **ADR-0008: Audio Pooling Strategy** — Pre-instantiated AudioStreamPlayer pool size and bus configuration for SFX. Covers: TR-FB-002.

---

## Architecture Principles

1. **Measurement before presentation**: The Core layer (SRE, Stats) must complete all computation before the Presentation layer renders. No async computation that could delay displayed results.

2. **Stateless computation, stateful storage**: StatisticalAnalysisEngine is a pure function — same input always produces same output. All state lives in SessionPersistence. This makes Stats trivially testable.

3. **Platform abstraction at the bottom**: StorageAbstraction is the only module that knows whether we're running in a browser or on desktop. Everything above it is platform-agnostic GDScript.

4. **External configuration for all tuning**: No magic numbers in code. All tuning knobs, reference tables, interpretation text, and plate definitions live in config files loaded by ConfigLoader. This enables balance changes without code modifications.

5. **Signals for decoupling, direct calls for data**: Systems communicate events via Godot signals (fire-and-forget, no return value). Systems request data via direct method calls on injected references (synchronous, typed return). Never use signals for request-response patterns.

---

## Open Questions

| Question | Owner | Target Resolution |
|----------|-------|-------------------|
| Should SRE be an Autoload singleton or instantiated per-module? | Technical Director | ADR-0001 |
| Does `JavaScriptBridge.eval()` for localStorage work reliably in Godot 4.6 HTML5 export? | Engine Programmer | ADR-0002 prototype |
| What is the actual `Time.get_ticks_msec()` precision in HTML5 export? | Engine Programmer | ADR-0003 prototype |
| Should ConfigLoader use `.tres` resources (type-safe, editor-friendly) or `.json` (portable, human-readable)? | Technical Director | ADR-0004 |
| Is `_draw()` + `queue_redraw()` performant enough for 200+ dots per Ishihara plate at 60fps? | Technical Artist | ADR-0006 prototype |
