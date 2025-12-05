# Bet: DevOS Layered RAG Memory

**Status:** Shaping
**Cycle:** Cycle 2
**Appetite:** 🐎 Medium (half-day)
**Owner:** Matt
**Origin:** DevOS Architecture Canvas + Observability Plan
**Goal:** Provide Hot/Cold contextual memory for agents across the DevOS toolchain, enabling fast local recall + deep semantic retrieval.

---

## 1. Shaping (Planning)

### Context & Problem
DevOS introduces large-scale telemetry, embeddings, vector memory, and agentic reasoning.
**Current gap:** agents lack a stable, structured *short-term memory* layer for fast contextual reasoning, and the observability vector DB (Timescale + pgvector) is too heavy for low-latency queries.

From DevOS Canvas:
- Agents require **vector memory for retrieval**
- DevOS relies on **contracts, ports, and deterministic logic** for safe agent modification of code
- Observability DB stores embeddings for logs, test failures, and proposals, but retrieval still hits Timescale directly (cold path)

**Problem:**
Agents repeatedly reload context from scratch, causing high latency, duplicated embedding calls, and unnecessary DB traffic.

### Scope

#### In Scope
- New `MemoryPort` definition under `src/ports/`
- In-process **Hot Memory Cache** (LRU + TTL)
- Bridge to Observability Vector DB for **Cold Memory**
- Logic functions for:
  - scoring,
  - deduplication,
  - relevance thresholds
- Integration with AgentPort request context
- Unit tests for memory retrieval policy

#### Out of Scope
- No UI changes
- No self-healing integration (Phase 7 work)
- No GPU/embedding model changes
- No new ingestion to observability DB

### Research & Options

- [x] Research conducted [Research Note](../../../analysis/rag-memory-research.md)
  - Hot memory missing ❗
  - Cold memory usable but slow
  - Telemetry + embeddings already operational

#### Option A: Pure DB Retrieval
- **Pros:** No extra infra
- **Cons:** Slow, expensive, no session-awareness

#### Option B: Layered Hot + Cold Memory (recommended)
- **Pros:** Fast, agent-friendly, session-scoped
- **Cons:** Needs new port and DI

#### Decision
**Choose Option B:** Implement **Layered RAG Memory** inside DevOS.

### Solution Design

High-level structure:

```text
Mastra Memory Provider (DevOSMemory)
 ├── HotMemoryAdapter (in-process LRU, TTL)
 └── ColdMemoryAdapter (Timescale/pgvector)
```

Retrieval pipeline:

```text
query → Mastra.search() → HotMemory → (fallback) → ColdMemory → merge → rerank → return
```

Use Cases:
* improving agent reasoning
* shaping self-heal proposals
* avoiding repetitive re-embedding
* compression of session context

---

## 2. Execution (Building)

### Implementation Plan

#### Phase 1 — Core Interfaces
* [ ] Add `MemoryPort` under `src/ports/`
* [ ] Define types: `MemoryRecord`, `MemoryQuery`, `MemorySearchResult`

#### Phase 2 — Hot Memory
* [ ] Create `HotMemoryAdapter` using pure LRU logic
* [ ] Add TTL + namespace support
* [ ] Add tests in `tests/unit/` using in-memory doubles

#### Phase 3 — Cold Memory
* [ ] Implement `ColdMemoryAdapter` using `EmbeddingStore` + Timescale queries
* [ ] Add similarity scoring + metadata filtering
* [ ] Write integration tests

#### Phase 4 — Retrieval Logic
* [ ] Add `MemoryService` (logic + ports orchestrator)
* [ ] Dedupe + rerank results
* [ ] Configurable similarity threshold

#### Phase 5 — Agent Integration
* [ ] Inject MemoryService into AgentPort context
* [ ] Add “memory retrieval” step before agent reasoning

### Acceptance Criteria
* [ ] `MemoryPort` with Hot/Cold implementation
* [ ] LRU Hot cache (100–500 records) with TTL
* [ ] Retrieval pipeline merges + reranks results
* [ ] Cold retrieval uses pgvector cosine search
* [ ] 90% of repeated queries resolve from Hot layer
* [ ] All logic pure; adapters isolated
* [ ] Tests: unit + integration pass
* [ ] No changes to self-healing system

### AI-Readiness Checklist
* [x] Problem clearly defined
* [x] Acceptance criteria testable
* [x] Repos/files linked below
* [x] Example behavior described
* [x] No ambiguous requirements

### Files/Repos Affected
* `toolkit-standards/src/ports/MemoryPort.ts`
* `toolkit-standards/src/services/MemoryService.ts`
* `toolkit-standards/src/adapters/HotMemoryAdapter.ts`
* `toolkit-standards/src/adapters/ColdMemoryAdapter.ts`
* `toolkit-standards/tests/unit/memory/*.test.ts`
* Observability DB used for cold layer (schema verified)

### References
* DevOS Architecture Canvas (Memory + Telemetry + Embeddings)
* Unified Observability Platform Schema (vector DB)
* [ADR-004: Layered RAG Memory](../../../../adr/ADR-004-LAYERED-RAG-MEMORY.md)

### Notes for AI
Use deterministic logic only.
Hot memory sits inside Node process; Cold memory uses the EmbeddingStore port.
Return top-k (≤10) relevancy results with metadata.

---

## 3. Review (Cool-Down)

### Verification
* [ ] All unit tests pass
* [ ] Integration tests against Timescale vector queries pass
* [ ] MemoryService returns ≤200ms responses for Hot hits
* [ ] Correct fallback behavior verified

### Retrospective
* **Successes**: ...
* **Challenges**: ...
* **Learnings**: ...
