# Phase 6 Strategy — Standards Adoption Cascade

**Last updated:** 2025-12-03
**Status:** Active (Cycle 1-3)

## Overview
This document outlines the strategic approach for **Phase 6: Repository Standardization + Toolkit-First Adoption**. It assumes the broader multi-repo, multi-organization vision defined in:

- `goals/yahwehos-integrated-vision.md` – YahwehOS / DevOS / Covenant ERP integrated vision

Execution is tracked in 2-week cycles:
*   [Cycle 1: Foundation](./cycles/cycle-1-foundation/README.md)
*   [Cycle 2: Toolkits](./cycles/cycle-2-toolkits/README.md)
*   [Cycle 3: Services](./cycles/cycle-3-services/README.md)

---

## Repository Classification

### Toolkits (Reusable Testbeds)
**Purpose:** Experimental testbeds where patterns emerge and mature for reuse.

*   **`toolkit-standards`** - Policy enforcement framework
*   **`hjps-toolkit`** - H&J infrastructure patterns
*   **`vector-toolkit`** - Generic vector/embedding patterns
*   **`frontend-toolkit`** - UI components + build tools

### Platforms (Deployable Services/Data)
**Purpose:** Production systems that consume toolkits.

*   **`hjps-ops`** - Platform-agnostic operational services
*   **`dataops`** - Multi-source data warehouse

---

## Adoption Strategy: Toolkit-First

**Rationale:** Fix toolkit issues before they propagate to consuming platforms.

### Order
1.  **toolkit-standards** (Cycle 1-2) - Dogfood first
2.  **Toolkits** (Cycle 2) - hjps-toolkit, vector-toolkit, frontend-toolkit
3.  **Ops** (Cycle 3) - hjps-ops (services + GUIs)
4.  **Dataops** (Cycle 3) - dataops (data platform)

### Each Cycle Includes
*   ✅ Publishing (if toolkit)
*   ✅ Standards enforcement (all applicable skills)
*   ✅ Core/IO refactoring
*   ✅ Feature development
*   🔄 **Lessons learned → update standards-cli**

---

## Key Decisions

### Architecture
*   **No Rust refactor** - Stay in TypeScript for Phase 6-7 (spike/exploration mode)
*   **No repository consolidation** - All 7 repos stay separate (distinct purposes)
*   **Core/IO separation order:** toolkit-standards → toolkits → ops → dataops
*   **Extraction timing:** Cycle 3+ (after boundary separation, not during)

### Frontend Strategy
*   **CLI → GUI transition** - Build web UIs in Cycle 2/3
*   **GUI drives better business logic** - Exposes complexity CLI masks
*   **Browser telemetry** - Feeds to observability platform for Phase 7

### Extraction Strategy
*   **When to extract:** Duplicated (≥2 repos), stable, clear boundary
*   **When NOT to extract:** Experimental, one-off, platform-specific
*   **Timing:** After Core/IO boundaries are clean

---

## Success Metrics

### Cycle 1-2
*   [ ] All toolkits published to GitHub Packages
*   [ ] Zero `file:` dependencies across all repos
*   [ ] `arch:layer-check` skill operational
*   [ ] ≥2 GUI applications built and deployed

### Cycle 3+
*   [ ] All repos pass standards enforcement (advisory mode)
*   [ ] ≥3 patterns extracted from ops/dataops to toolkits
*   [ ] Violation rate < 5% for 2 consecutive weeks
