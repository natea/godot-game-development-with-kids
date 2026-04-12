# Session Persistence

> **Status**: Designed
> **Author**: systems-designer
> **Last Updated**: 2026-04-12
> **Implements Pillar**: Pillar 3 (Reveal Through Repetition)

## Summary

The session persistence system stores test results in browser localStorage,
manages session history for trend analysis, and provides one-click JSON export
to protect player data from browser storage policies. It is the foundation for
Pillar 3 — without persistence, every session is an island.

> **Quick reference** — Layer: `Feature` · Priority: `MVP` · Key deps: `Statistical Analysis Engine`

## Overview

After each test battery, the session persistence system saves the computed
statistics to the browser's localStorage. It maintains a chronological array of
session records that the Statistical Analysis Engine reads for trend computation
and the Results & Interpretation system reads for historical charts. The player
never interacts with this system directly — they experience it through the trend
arrows, historical charts, and growing confidence metrics that emerge over multiple
sessions. The system also provides a one-click JSON export to guard against data
loss from browser storage clearing (private browsing, Safari 7-day policy, user
action). This system exists because Pillar 3 requires data to accumulate across
sessions, and browser localStorage is the only viable storage mechanism for a
free, no-account, single-player web game.

## Player Fantasy

This is pure infrastructure. The player never thinks about persistence — they
think about their trend line growing, their confidence increasing, and their
history being available when they return tomorrow. The fantasy this enables is
**continuity** — the game remembers you without asking you to create an account.
Your data is yours, on your machine, and it's still there when you come back.

## Detailed Design

### Core Rules

1. **Storage mechanism**: Browser `localStorage` via Godot's JavaScript bridge
   (or Godot's built-in `user://` path for desktop exports, which maps to the
   user data directory).
2. **Storage key**: `synaptic_session_data` — a single key containing the entire
   JSON payload.
3. **Data structure**: A JSON object containing:
   - `version`: schema version (integer, starting at 1)
   - `sessions`: array of `SessionRecord` objects, chronologically ordered
   - `player_id`: a locally-generated UUID (no server, no account — just for
     data export identification)
   - `created_at`: ISO 8601 timestamp of first session
   - `last_updated`: ISO 8601 timestamp of most recent session
4. **SessionRecord structure**:
   ```
   {
     session_id: string (UUID),
     timestamp: string (ISO 8601),
     modules: [
       {
         type: "reaction_time" | "color_perception",
         corrected_median_rt: int | null,
         raw_median_rt: int | null,
         percentile: int | null,
         band: string | null,
         iqr: int | null,
         valid_trial_count: int,
         timeout_count: int,
         trials: [ { rt_ms, is_timeout, is_early } ],
         plate_results: [ { plate_id, is_correct, player_answer } ] | null
       }
     ],
     cvd_screening: {
       protan_deutan_miss_rate: float | null,
       tritan_miss_rate: float | null,
       cvd_indicator: string | null,
       cumulative_confidence: float
     }
   }
   ```
5. **Save trigger**: Data is saved after each module completes (not just after
   the full battery). This prevents data loss if the player closes the tab
   mid-battery.
6. **Read trigger**: Data is loaded on application start and whenever the Results
   & Interpretation system requests historical data.
7. **JSON export**: A button on the results or history screen triggers a browser
   file download of the complete JSON payload. Filename format:
   `synaptic_data_YYYY-MM-DD.json`.
8. **No import at MVP**: JSON export is one-way at MVP. Import functionality is
   deferred to Vertical Slice tier.
9. **No server communication**: All data stays local. No analytics, no telemetry,
   no cloud sync. The player's data is never transmitted.
10. **Schema migration**: When loading data, check `version` against current
    schema version. If older, run migration logic to transform the data structure
    to the current format. Never discard old data — always migrate forward.

### States and Transitions

| State | Entry Condition | Exit Condition | Behavior |
|-------|----------------|----------------|----------|
| `UNINITIALIZED` | App first launch, no stored data | First session saved | `player_id` generated, empty `sessions` array created, data written to storage. |
| `LOADED` | Data found in localStorage on app start | App closed or data cleared | Session history available for reads. New sessions appended on save. |
| `SAVE_ERROR` | localStorage write fails (quota, private browsing) | User acknowledges or retries | Error surfaced to user: "Unable to save data. Your results from this session may not be preserved. Consider exporting your data." Current session data held in memory. |
| `CORRUPTED` | Stored JSON fails to parse | User acknowledges | Error surfaced: "Stored data could not be read. Starting fresh. If you have a previous export, you can restore it later." New empty storage initialized. Old corrupted data logged to console for debugging. |

### Interactions with Other Systems

| System | Direction | Interface |
|--------|-----------|-----------|
| **Statistical Analysis Engine** | Upstream (provides data) | Receives `SessionStats` after each module analysis. Packages into `SessionRecord` and saves. |
| **Results & Interpretation** | Downstream (provides data) | Serves historical `SessionRecord` array for trend charts and historical comparisons on request. |
| **Test Sequencing** | Downstream (read) | Provides session count for onboarding logic ("is this the first session?") and module unlock tracking (future). |

### Storage Size Management

- Each session record is approximately 2–4KB (including raw trial data).
- localStorage limit is typically 5MB per origin.
- At 3KB average per session, the system supports ~1,600 sessions before hitting
  the limit.
- If storage approaches 80% capacity (4MB), show a warning: "Storage is nearly
  full. Export your data to prevent loss. Oldest sessions may be removed."
- If storage exceeds 90% (4.5MB), automatically archive oldest sessions: remove
  raw trial arrays (keep computed statistics only) from the oldest 50% of sessions.
  This preserves trend data while freeing space.

## Formulas

### Storage Size Estimation

`estimated_size_bytes = session_count * avg_session_bytes`

| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| session_count | N | int | 0–∞ | Total sessions stored |
| avg_session_bytes | B | int | 2000–4000 | Average bytes per session record |
| estimated_size_bytes | S | int | 0–5,000,000 | Estimated total storage usage |

**Output Range:** 0 to ~5MB (localStorage limit).

**Example:** 100 sessions × 3000 bytes = 300KB (well within limit).

### Archive Threshold

`should_warn = estimated_size_bytes > (storage_limit * warn_threshold)`
`should_archive = estimated_size_bytes > (storage_limit * archive_threshold)`

| Variable | Type | Value | Description |
|----------|------|-------|-------------|
| storage_limit | int | 5,242,880 | 5MB in bytes |
| warn_threshold | float | 0.80 | Warn at 80% capacity |
| archive_threshold | float | 0.90 | Auto-archive at 90% capacity |

## Edge Cases

| Scenario | Expected Behavior | Rationale |
|----------|------------------|-----------|
| localStorage not available (private browsing in some browsers) | Detect on startup. Show notice: "Private browsing mode detected — your results won't be saved between sessions. Switch to normal browsing to keep your data." Current session still works, data held in memory only. | Don't break the experience — just inform. The game is still playable without persistence. |
| Safari 7-day localStorage policy | No direct mitigation possible from JavaScript. Export button is prominently placed. On first launch, show: "Tip: Export your data periodically to prevent loss." | Safari deletes localStorage for sites not visited in 7 days. Export is the only reliable protection. |
| Stored data is from an older schema version | Run migration function chain: v1→v2, v2→v3, etc. Each migration is a pure function that transforms the data shape. Never drop fields — add defaults for new fields. | Forward compatibility ensures no data loss across updates. |
| localStorage write fails mid-save (tab crash, quota) | Data in memory is preserved for the current session. On next successful write, retry with the full dataset. | Transient failures shouldn't permanently lose data. |
| Two browser tabs open simultaneously | Each tab reads on startup and writes on save. Last-write-wins. No conflict resolution. | This is a single-player tool — simultaneous tabs are an edge case not worth the complexity of conflict resolution. |
| Player clears browser data | All stored data is lost. No recovery possible unless player has a JSON export. | This is a known limitation. The export feature exists specifically for this scenario. |
| JSON export file is manually edited by user | On import (future feature): validate JSON schema before accepting. Reject invalid data with clear error. | Users who export may tinker. Don't crash on malformed input. |
| Session with 0 valid trials (all timeouts) | Save the session record with null statistical fields. It counts toward session count but does not affect trend calculations. | Even empty sessions are data — they record that the player attempted but produced no valid responses. |

## Dependencies

| System | Direction | Nature of Dependency |
|--------|-----------|---------------------|
| Statistical Analysis Engine | This depends on Stats | Requires `SessionStats` to create session records. Cannot save meaningful data without statistical output. Hard dependency for save. |
| Results & Interpretation | Depended on by R&I | Provides historical data for trend charts. R&I can display current-session-only results without history. Soft dependency. |
| Test Sequencing | Depended on by TS (soft) | Provides session count and history for onboarding decisions. TS can function without it. Soft dependency. |

## Tuning Knobs

| Parameter | Current Value | Safe Range | Effect of Increase | Effect of Decrease |
|-----------|--------------|------------|-------------------|-------------------|
| `storage_key` | "synaptic_session_data" | Any valid string | N/A — just a namespace | N/A |
| `warn_threshold` | 0.80 | 0.60–0.95 | Later warning; more storage used before notice | Earlier warning; more conservative |
| `archive_threshold` | 0.90 | 0.70–0.98 | Later auto-archive; risk of hitting hard limit | Earlier archive; more aggressive space management |
| `max_sessions_before_archive` | 1600 | 500–2000 | More sessions kept with full trial data | Fewer sessions; earlier archival of raw trials |
| `export_filename_format` | "synaptic_data_YYYY-MM-DD" | Any valid format | N/A — cosmetic | N/A |

## Visual/Audio Requirements

This system has minimal direct visual output:

| Event | Visual Feedback | Audio Feedback | Priority |
|-------|----------------|---------------|----------|
| Export button pressed | Brief button press animation (scale 0.95, 80ms). Browser download dialog appears. | None. | Low |
| Storage warning | Banner at top of results screen: "Storage nearly full — export recommended." `--alert-amber` text. | None. | Medium |
| Save error | Inline notice on results screen: "Unable to save — data may not persist." `--alert-amber`. | None. | High |
| First launch detection | No visual change — handled by Test Sequencing for onboarding. | None. | Low |

## UI Requirements

| Information | Display Location | Update Frequency | Condition |
|-------------|-----------------|-----------------|-----------|
| Export button | Results screen and history screen, bottom-right | Persistent | Always visible when data exists |
| Storage warning | Top banner on results screen | On results display | When storage > 80% capacity |
| Session count | History screen header: "X sessions recorded" | On history view | Always |
| Data age | History screen: "First session: [date]" | On history view | Always |

## Cross-References

| This Document References | Target GDD | Specific Element Referenced | Nature |
|--------------------------|-----------|----------------------------|--------|
| Receives session stats for storage | `design/gdd/statistical-analysis-engine.md` | `SessionStats` output format | Data dependency |
| Provides historical data for display | `design/gdd/results-interpretation.md` | Historical session array read | Data dependency |
| Provides session context for sequencing | `design/gdd/test-sequencing.md` | Session count, first-launch detection | Data dependency |

## Acceptance Criteria

- [ ] GIVEN a module completes with valid results, WHEN the session is saved, THEN a `SessionRecord` is written to localStorage containing all statistical fields
- [ ] GIVEN localStorage contains previous sessions, WHEN the app loads, THEN all historical sessions are available for trend computation
- [ ] GIVEN the player clicks the export button, WHEN the browser download triggers, THEN a valid JSON file is downloaded containing the complete session history
- [ ] GIVEN localStorage is unavailable (private browsing), WHEN the app starts, THEN a notice is shown and the current session still functions (data in memory only)
- [ ] GIVEN stored data is from schema v1 and current is v2, WHEN data loads, THEN migration transforms v1 data to v2 format without data loss
- [ ] GIVEN storage exceeds 90% capacity, WHEN a new session is saved, THEN oldest sessions have raw trial arrays removed to free space
- [ ] GIVEN a session with 0 valid trials, WHEN saved, THEN the session record exists with null statistical fields and does not affect trends
- [ ] GIVEN two modules in one battery, WHEN each completes, THEN data is saved after each module (not only after the full battery)
- [ ] Performance: Save operation completes within 50ms (localStorage write is synchronous)
- [ ] No hardcoded storage keys or thresholds — all parameters from configuration

## Open Questions

| Question | Owner | Deadline | Resolution |
|----------|-------|----------|-----------|
| Should we implement JSON import at MVP or defer to Vertical Slice? | Game Designer | Sprint 1 | Current decision: defer. Export-only at MVP. |
| Should data include a checksum/hash to detect tampering? | Systems Designer | Before Alpha | Low priority for single-player self-assessment tool. |
| Should we use IndexedDB instead of localStorage for larger storage? | Engine Programmer | Before Alpha | localStorage is simpler but limited to ~5MB. IndexedDB supports much more but adds async complexity. |
| How to handle Godot `user://` path vs browser localStorage in a unified API? | Godot Specialist | Before implementation | Need an abstraction layer that works for both HTML5 and desktop exports. |
