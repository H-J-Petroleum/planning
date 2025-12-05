# ADR-004: Layered RAG Memory for DevOS

**Date:** 2025-12-04
**Status:** Proposed
**Context:** From DevOS Canvas (vector memory requirement) + Observability Schema

## 📝 Context

Agents require structured, repeatable access to context—too slow to fetch exclusively from the Timescale vector DB.
DevOS has no short-term memory layer; only long-term incident memory exists.

## 🤔 Options Considered

### Option 1: DB-Only Retrieval

**Pros:** simple
**Cons:** slow, repeated embedding calls, expensive, no session-awareness

### Option 2: Layered Hot + Cold Memory

**Pros:**

* fast
* predictable
* plays well with DevOS purity + DI
* improves agent reasoning
* reduces DB load

**Cons:**

* requires new port + services

## 🎯 Decision

**Use layered memory:**

* Hot memory = in-process LRU
* Cold memory = observability vector DB

Aligns with DevOS architectural constraints (ports + adapters) and AI reasoning patterns.

## 🔮 Consequences

**Positive:**

* Faster agent loops
* Reduced costs
* Deterministic structure for memory recall

**Negative:**

* Must manage TTL + cache invalidation
* More moving parts
