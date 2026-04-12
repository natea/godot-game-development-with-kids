# Systems Index: Synaptic

> **Status**: Draft
> **Created**: 2026-04-12
> **Last Updated**: 2026-04-12
> **Source Concept**: design/gdd/game-concept.md

---

## Overview

Synaptic is a minimalist cognitive testing game built around two core test modules
(reaction time and color perception) with statistical analysis, interpretive results,
and session tracking. The mechanical scope is narrow but deep — precision measurement,
statistical rigor, and satisfying feedback are the systems that make the game work.
The core loop is: stimulus → response → measurement → feedback → interpretation → trend.

All MVP systems serve the three pillars: Measure Don't Guess (statistical accuracy),
Every Tap Feels Crisp (input fidelity and feedback), and Reveal Through Repetition
(session persistence and trend analysis).

---

## Systems Enumeration

| # | System Name | Category | Priority | Status | Design Doc | Depends On |
|---|-------------|----------|----------|--------|------------|------------|
| 1 | Stimulus-Response Engine | Core | MVP | Designed | design/gdd/stimulus-response-engine.md | — |
| 2 | Color Perception System | Gameplay | MVP | Designed | design/gdd/color-perception-system.md | Stimulus-Response Engine |
| 3 | Statistical Analysis Engine | Core | MVP | Designed | design/gdd/statistical-analysis-engine.md | — |
| 4 | Feedback System | Presentation | MVP | Designed | design/gdd/feedback-system.md | Stimulus-Response Engine |
| 5 | Results & Interpretation | Gameplay | MVP | Designed | design/gdd/results-interpretation.md | Statistical Analysis Engine |
| 6 | Session Persistence | Persistence | MVP | Designed | design/gdd/session-persistence.md | Statistical Analysis Engine |
| 7 | Test Sequencing | Core | MVP | Designed | design/gdd/test-sequencing.md | Stimulus-Response Engine, Color Perception System |

---

## Categories

| Category | Description | Typical Systems |
|----------|-------------|-----------------|
| **Core** | Foundation systems everything depends on | Stimulus-response engine, statistical analysis, test sequencing |
| **Gameplay** | The systems that make the game meaningful | Color perception screening, results interpretation |
| **Persistence** | Save state and continuity | Session storage, trend tracking, data export |
| **Presentation** | Player-facing feedback and display | Feedback juice, results dashboard |

---

## Priority Tiers

| Tier | Definition | Target Milestone | Design Urgency |
|------|------------|------------------|----------------|
| **MVP** | Required for core loop: test → measure → interpret → persist | First playable | Design FIRST |
| **Vertical Slice** | Complete polished experience with onboarding and advanced stats | Demo-ready | Design SECOND |
| **Alpha** | Choice reaction module, CVD type classification, advanced modules | Feature-complete | Design THIRD |
| **Full Vision** | Daily challenges, mobile layout, adaptive difficulty, share/export | Release | Design as needed |

---

## Dependency Map

### Foundation Layer (no dependencies)

1. **Stimulus-Response Engine** — the atomic unit of the game: present stimulus, capture input, measure time. Everything builds on this.
2. **Statistical Analysis Engine** — pure math: percentile computation, confidence intervals, browser-offset correction. No UI or input dependencies.

### Core Layer (depends on foundation)

3. **Color Perception System** — depends on: Stimulus-Response Engine. Extends the stimulus-response pattern with Ishihara plate generation and color-science logic.
4. **Feedback System** — depends on: Stimulus-Response Engine. Triggers audio/visual juice from stimulus-response events (needs to know when a response occurs and how fast it was).

### Feature Layer (depends on core)

5. **Results & Interpretation** — depends on: Statistical Analysis Engine. Transforms raw statistics into anchored language and visual dashboard. Needs percentiles and norms to produce interpretive text.
6. **Session Persistence** — depends on: Statistical Analysis Engine. Stores computed statistics per session, builds trend data over time. Needs the stats engine to know what to store.
7. **Test Sequencing** — depends on: Stimulus-Response Engine, Color Perception System. Orchestrates modules into batteries, manages trial flow, handles module selection and onboarding. Needs the test modules to exist before it can sequence them.

---

## Recommended Design Order

| Order | System | Priority | Layer | Agent(s) | Est. Effort |
|-------|--------|----------|-------|----------|-------------|
| 1 | Stimulus-Response Engine | MVP | Foundation | game-designer, systems-designer | M |
| 2 | Statistical Analysis Engine | MVP | Foundation | systems-designer | M |
| 3 | Color Perception System | MVP | Core | game-designer, systems-designer | M |
| 4 | Feedback System | MVP | Core | game-designer, sound-designer | S |
| 5 | Results & Interpretation | MVP | Feature | game-designer, ux-designer | M |
| 6 | Session Persistence | MVP | Feature | systems-designer | S |
| 7 | Test Sequencing | MVP | Feature | game-designer | S |

---

## Circular Dependencies

- None found. The dependency graph is a clean DAG — foundation systems have no
  upstream dependencies, and no feature-layer system depends on another feature-layer system.

---

## High-Risk Systems

| System | Risk Type | Risk Description | Mitigation |
|--------|-----------|-----------------|------------|
| Stimulus-Response Engine | Technical | Browser input timing has ~5ms precision floor. Godot HTML5 export adds processing overhead. Actual measurement precision per browser is unknown. | Prototype first. Benchmark Chrome/Firefox/Safari. Display ±5ms uncertainty band. |
| Color Perception System | Design + Technical | Ishihara plate color pairings require color science expertise. Consumer displays are uncalibrated — 30-40% false negative rate possible for mild CVD on TN panels. | Use curated (not procedural) color pairings from published sources. Position as "screening indicator" not diagnosis. |
| Statistical Analysis Engine | Technical | Browser-to-lab norm offset correction (+15ms) is an estimate. Per-browser correction factors need empirical validation. | Build correction as a configurable parameter. Start with +15ms, validate with prototype data. |

---

## Progress Tracker

| Metric | Count |
|--------|-------|
| Total systems identified | 7 |
| Design docs started | 7 |
| Design docs reviewed | 0 |
| Design docs approved | 0 |
| MVP systems designed | 7/7 |
| Vertical Slice systems designed | 0/0 |

---

## Next Steps

- [x] Review and approve this systems enumeration
- [ ] Design MVP-tier systems first (use `/design-system [system-name]`)
- [ ] Run `/design-review` on each completed GDD
- [ ] Run `/gate-check pre-production` when MVP systems are designed
- [ ] Prototype the Stimulus-Response Engine early (`/prototype stimulus-response`)
