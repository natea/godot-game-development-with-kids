# ADR-0002: Storage Abstraction — Unified localStorage / user:// API

## Status

Accepted

## Date

2026-04-12

## Last Verified

2026-04-12

## Decision Makers

Technical Director, Engine Programmer

## Summary

Synaptic targets both HTML5 (browser) and desktop, which use different storage mechanisms. We chose a StorageAbstraction class that routes to `JavaScriptBridge` localStorage calls on HTML5 and `FileAccess` on desktop, exposing a unified `save(key, data)` / `load(key)` API.

## Engine Compatibility

| Field | Value |
|-------|-------|
| **Engine** | Godot 4.6 |
| **Domain** | Core / Platform |
| **Knowledge Risk** | MEDIUM — JavaScriptBridge behavior in 4.6 HTML5 export needs verification |
| **References Consulted** | `docs/engine-reference/godot/modules/input.md`, Godot docs |
| **Post-Cutoff APIs Used** | `JavaScriptBridge.eval()` — API exists since 4.0 but HTML5 export behavior may vary in 4.6 |
| **Verification Required** | Test `JavaScriptBridge.eval("localStorage.setItem(...)")` and `getItem()` in Godot 4.6 HTML5 export across Chrome, Firefox, Safari |

## ADR Dependencies

| Field | Value |
|-------|-------|
| **Depends On** | None |
| **Enables** | ADR-0004 (Data Serialization) |
| **Blocks** | Session Persistence implementation |
| **Ordering Note** | Must prototype HTML5 localStorage access before committing |

## Context

### Problem Statement

Synaptic stores session history locally. On HTML5, the only viable mechanism is `localStorage`. On desktop, Godot's `user://` path provides file-based storage. The game needs a single API so that SessionPersistence doesn't contain platform-specific branching.

### Constraints

- HTML5 localStorage: ~5MB limit, synchronous, string-only, cleared by browser policies
- Desktop `user://`: unlimited (practical), file-based, survives indefinitely
- No server — all data is local
- `JavaScriptBridge` is only available in HTML5 exports

### Requirements

- Unified save/load API (TR-SP-001, TR-SP-002)
- JSON string storage (TR-SP-003)
- File download trigger for export (TR-SP-004)
- Platform detection without caller awareness

## Decision

Create a `StorageAbstraction` class (extends `RefCounted`) that detects the platform at initialization and routes all operations to the appropriate backend.

### Key Interfaces

```gdscript
class_name StorageAbstraction extends RefCounted

var _is_html5: bool

func _init() -> void:
    _is_html5 = OS.has_feature("web")

func save(key: String, data: String) -> bool:
    if _is_html5:
        var js_code := "localStorage.setItem('%s', '%s')" % [
            key, data.c_escape()
        ]
        JavaScriptBridge.eval(js_code)
        return true
    else:
        var file := FileAccess.open("user://%s.json" % key, FileAccess.WRITE)
        if file == null:
            return false
        file.store_string(data)
        return true

func load(key: String) -> String:
    if _is_html5:
        var result: Variant = JavaScriptBridge.eval(
            "localStorage.getItem('%s')" % key
        )
        return result if result is String else ""
    else:
        var path := "user://%s.json" % key
        if not FileAccess.file_exists(path):
            return ""
        var file := FileAccess.open(path, FileAccess.READ)
        return file.get_as_text()

func is_available() -> bool:
    if _is_html5:
        var result: Variant = JavaScriptBridge.eval(
            "typeof(Storage) !== 'undefined'"
        )
        return result == true
    return true

func export_file(data: String, filename: String) -> void:
    if _is_html5:
        JavaScriptBridge.download_buffer(
            data.to_utf8_buffer(), filename, "application/json"
        )
    else:
        var path := OS.get_user_data_dir() + "/" + filename
        var file := FileAccess.open(path, FileAccess.WRITE)
        file.store_string(data)
        OS.shell_open(OS.get_user_data_dir())
```

### Implementation Guidelines

- `StorageAbstraction` is instantiated once by `SessionPersistence`
- All data is stored as JSON strings — the abstraction handles raw strings only
- `c_escape()` on data before embedding in JS eval to prevent injection
- Desktop export opens the user data folder after writing (for discoverability)
- Error handling: `save()` returns `false` on failure; caller shows user-facing warning

## Alternatives Considered

### Alternative 1: Godot ConfigFile

- **Description**: Use `ConfigFile` which works on all platforms
- **Pros**: Native Godot API, no JavaScript bridge needed
- **Cons**: `ConfigFile` uses INI format (not JSON), no control over storage location on HTML5, cannot trigger browser file download
- **Rejection Reason**: Doesn't support the JSON export requirement and doesn't give us localStorage access for HTML5

### Alternative 2: IndexedDB via JavaScriptBridge

- **Description**: Use IndexedDB instead of localStorage for larger storage
- **Pros**: Much larger storage limit (hundreds of MB), async API
- **Cons**: Async API adds complexity, overkill for < 5MB of session data
- **Rejection Reason**: localStorage's 5MB is sufficient for ~1600 sessions. IndexedDB's async nature would complicate the synchronous data flow.

## Consequences

### Positive

- SessionPersistence has zero platform-specific code
- JSON export works on both platforms
- Simple, testable API

### Negative

- `JavaScriptBridge.eval()` with string interpolation requires careful escaping
- localStorage is subject to browser storage policies (Safari 7-day)

## Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| `JavaScriptBridge.eval()` behaves differently in Godot 4.6 HTML5 | Low | High | Prototype and test before committing |
| localStorage cleared by browser | Medium | Medium | Export button prominently placed; first-launch tip |
| Large JSON strings cause performance issues in eval() | Low | Medium | Monitor save time; if > 100ms, chunk writes |

## Performance Implications

| Metric | Before | Expected After | Budget |
|--------|--------|---------------|--------|
| Save time | N/A | < 50ms (localStorage is synchronous) | 100ms acceptable |
| Load time | N/A | < 50ms | 200ms at startup acceptable |
| Storage | N/A | ~3KB per session, ~5MB total limit | 5MB hard limit (HTML5) |

## Validation Criteria

- [ ] `save()` and `load()` work correctly in Chrome, Firefox, Safari HTML5 export
- [ ] `save()` and `load()` work correctly on macOS/Windows desktop export
- [ ] `export_file()` triggers browser download dialog on HTML5
- [ ] `is_available()` returns `false` in private browsing (if applicable)
- [ ] JSON strings with special characters survive round-trip (save → load)

## GDD Requirements Addressed

| GDD Document | System | Requirement | How This ADR Satisfies It |
|-------------|--------|-------------|--------------------------|
| session-persistence.md | SP | TR-SP-001: localStorage via JavaScript bridge | `JavaScriptBridge.eval()` for localStorage access |
| session-persistence.md | SP | TR-SP-002: user:// file storage | `FileAccess` for desktop exports |
| session-persistence.md | SP | TR-SP-003: JSON serialization | Stores/retrieves raw JSON strings |
| session-persistence.md | SP | TR-SP-004: Browser file download | `JavaScriptBridge.download_buffer()` for export |

## Related

- ADR-0004 (Data Serialization Format) — defines what format the stored data uses
