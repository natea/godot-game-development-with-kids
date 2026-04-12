# ADR-0004: Data Serialization Format — JSON for Persistence, .tres for Config

## Status

Accepted

## Date

2026-04-12

## Last Verified

2026-04-12

## Decision Makers

Technical Director, Engine Programmer

## Summary

Synaptic uses two serialization formats: JSON for all player-facing session data (persistence, export) and Godot `.tres` resource files for static configuration (reference distributions, interpretation language, plate definitions). JSON is chosen for portability and browser compatibility; `.tres` is chosen for engine-native loading and type safety. Schema versioning uses an integer version field with a forward-only migration chain.

## Engine Compatibility

| Field | Value |
|-------|-------|
| **Engine** | Godot 4.6 |
| **Domain** | Core / Data |
| **Knowledge Risk** | LOW — JSON class unchanged since 4.0; Resource loading stable |
| **References Consulted** | `docs/engine-reference/godot/modules/input.md`, Godot docs |
| **Post-Cutoff APIs Used** | None |
| **Verification Required** | None |

## ADR Dependencies

| Field | Value |
|-------|-------|
| **Depends On** | ADR-0002 (Storage Abstraction) |
| **Enables** | Session Persistence implementation, Results Interpretation config loading |
| **Blocks** | None |

## Context

### Problem Statement

Synaptic stores two categories of data: (1) player session history that must be portable, exportable, and browser-compatible, and (2) static configuration like reference distribution tables and interpretation language that ships with the game. These have different requirements — session data must survive round-trips through localStorage and file downloads; config data needs type safety and engine-native loading.

### Constraints

- Session data passes through `StorageAbstraction` which stores raw strings (ADR-0002)
- HTML5 localStorage stores strings — binary formats are impractical
- JSON export is a player-facing feature (downloadable file)
- Config data is read-only at runtime — never modified by the player
- Schema must support forward migration as the game evolves

### Requirements

- JSON serialization for session records (TR-SP-003)
- Schema versioning with migration chain (TR-SP-005)
- Reference distribution loaded from external config (TR-STAT-004)
- Interpretation language loaded from external config (TR-RI-003)

## Decision

### Session Data: JSON with Schema Versioning

All player session data uses JSON serialization via Godot's `JSON` class. Every stored JSON object includes a `schema_version` integer at the root level. Migration functions are chained: `v1→v2→v3→...`, each transforming the dictionary in place.

```gdscript
# Session data schema (v1)
{
    "schema_version": 1,
    "player_id": "uuid-string",
    "sessions": [
        {
            "timestamp": 1712937600,
            "module": "reaction_time",
            "trials": [
                {"rt_ms": 247, "trial_index": 0, "is_outlier": false},
                {"rt_ms": 312, "trial_index": 1, "is_outlier": false}
            ],
            "summary": {
                "median_rt": 265,
                "iqr": 42,
                "valid_trials": 18,
                "total_trials": 20
            }
        }
    ]
}
```

### Schema Migration

```gdscript
class_name SchemaMigrator extends RefCounted

const CURRENT_VERSION: int = 1

func migrate(data: Dictionary) -> Dictionary:
    var version: int = data.get("schema_version", 0)
    while version < CURRENT_VERSION:
        data = _migrations[version].call(data)
        version += 1
    return data

var _migrations: Dictionary = {
    # 0: func(d: Dictionary) -> Dictionary:
    #     d["schema_version"] = 1
    #     d["player_id"] = _generate_uuid()
    #     return d
}
```

### Config Data: Godot .tres Resources

Static configuration uses custom `Resource` subclasses saved as `.tres` files. These are loaded via `load()` or `preload()` and provide type-safe access.

```gdscript
# Reference distribution config
class_name ReferenceDistribution extends Resource

@export var percentile_table: Dictionary = {
    5: 150, 10: 165, 25: 190, 50: 225,
    75: 270, 90: 320, 95: 370
}
@export var source_citation: String = "Luce 1986, Jain 2015"
@export var age_group: String = "18-35"

# Interpretation language config
class_name InterpretationLanguage extends Resource

@export var band_descriptions: Dictionary = {
    "elite": "Exceptionally fast — consistent with trained professionals",
    "fast": "Faster than most — above the 75th percentile",
    "average": "Within normal range — consistent with most healthy adults",
    "slow": "Below average — may reflect fatigue, distraction, or device latency"
}
```

### Implementation Guidelines

- Use `JSON.stringify()` for serialization and `JSON.parse_string()` for deserialization
- Always validate `schema_version` before accessing any session data fields
- Migration functions are pure — they take a Dictionary and return a Dictionary
- Config `.tres` files live in `assets/data/` and are preloaded at scene ready
- Never mix formats — JSON for mutable player data, `.tres` for immutable config
- `SchemaMigrator` extends `RefCounted` for testability (no scene tree dependency)

## Alternatives Considered

### Alternative 1: JSON for Everything (Including Config)

- **Description**: Use JSON files for config data too, loaded via `FileAccess`
- **Pros**: Single format, simpler mental model
- **Cons**: Loses type safety, no `preload()` support, must parse at runtime, no editor integration
- **Rejection Reason**: Config data benefits from Godot's resource system — type checking, editor preview, preloading. Using JSON for config would be fighting the engine.

### Alternative 2: Godot Resources for Everything (Including Session Data)

- **Description**: Use `.tres` for session data too
- **Pros**: Type-safe throughout, engine-native
- **Cons**: `.tres` cannot be stored in localStorage, cannot be exported as a human-readable download, binary format is opaque to players
- **Rejection Reason**: Session data must pass through localStorage (string-only) and be downloadable as a readable file. `.tres` fails both requirements.

### Alternative 3: MessagePack or BSON

- **Description**: Use a binary format for compact storage
- **Pros**: Smaller storage footprint, faster parsing
- **Cons**: Not human-readable (violates export requirement), requires external library, overkill for ~3KB per session
- **Rejection Reason**: Storage size is not a concern (< 5MB total). Human-readable export is a feature requirement.

## Consequences

### Positive

- Clear separation: JSON for portable data, `.tres` for engine-native config
- Schema migration ensures forward compatibility as the game evolves
- Player export files are human-readable JSON
- Config resources get type safety and editor integration

### Negative

- Two serialization formats to understand (minor cognitive overhead)
- Schema migration code must be maintained as the schema evolves

## Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| Schema migration breaks existing data | Low | High | Unit test every migration function with real v(N) data |
| JSON parsing performance on large histories | Low | Low | Monitor; if > 100ms, consider pagination |
| `.tres` format changes between Godot versions | Very Low | Medium | Pin engine version; test on upgrade |

## Performance Implications

| Metric | Before | Expected After | Budget |
|--------|--------|---------------|--------|
| JSON serialize | N/A | < 10ms for 50 sessions | 100ms acceptable |
| JSON parse | N/A | < 20ms for 50 sessions | 200ms at startup |
| Config load | N/A | < 5ms (preloaded) | Negligible |

## Validation Criteria

- [ ] Session data round-trips through JSON without data loss (save → load → compare)
- [ ] Schema migration correctly transforms v0 data to current version
- [ ] Config `.tres` files load correctly via `preload()` and `load()`
- [ ] Exported JSON file is valid, human-readable JSON
- [ ] Special characters in session data survive JSON encoding

## GDD Requirements Addressed

| GDD Document | System | Requirement | How This ADR Satisfies It |
|-------------|--------|-------------|--------------------------|
| session-persistence.md | SP | TR-SP-003: JSON serialization | `JSON.stringify()` / `JSON.parse_string()` for all session data |
| session-persistence.md | SP | TR-SP-005: Schema versioning | Integer version field with forward migration chain |
| statistical-analysis-engine.md | Stats | TR-STAT-004: Reference distribution from config | Custom `Resource` subclass loaded as `.tres` |
| results-interpretation.md | R&I | TR-RI-003: Interpretation language from config | Custom `Resource` subclass loaded as `.tres` |

## Related

- ADR-0002 (Storage Abstraction) — defines the save/load API that carries JSON strings
- ADR-0001 (State Management) — StatisticalAnalysisEngine consumes config resources
