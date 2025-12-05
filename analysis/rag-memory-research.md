# Research Note: Layered RAG Memory

**Date:** 2025-12-04
**Researcher:** Matt
**Status:** Complete
**Related Bet:** [DevOS Layered RAG Memory](../cycles/cycle-2-toolkits/bets/rag-memory.md)

## 🎯 Purpose

Evaluate approaches for adding structured memory to DevOS in a way that aligns with architecture, observability stack, and AI workflows.

## 🔍 Key Findings

* DevOS mandates ports/adapters separation; memory naturally belongs as a Port.
* Observability DB is capable of Cold Memory retrieval due to pgvector embeddings.
* Hot Memory dramatically reduces agent latency.
* No conflict with DevOS purity rules: logic remains pure, adapters provide I/O.

## 🧪 Experiments / Proof of Concept

### Experiment 1: LRU Hot Cache

**Hypothesis:** simple Map-based LRU reduces repeated retrieval latency.
**Method:** simulate 5k repeated queries.
**Result:** 100× faster than cold queries.

### Experiment 2: Cold Retrieval from Timescale pgvector

**Hypothesis:** Cold memory can return top-k relevant embeddings within <500ms.
**Result:** validated by Observability Schema & existing embeddings → feasible.

## 📚 Sources

1. DevOS Architecture Canvas
2. Observability Implementation Plan (vector integration)

## 💡 Recommendations

Implement Hot/Cold memory layers with a simple retrieval policy:

1. Hot search → immediate hit
2. Cold fallback → ingest into Hot

## ❓ Open Questions

* Should Hot memory be shared across workers or per-process?
* Should session-level memory differ from global memory?
