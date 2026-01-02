# Memory Service Architecture

This document describes the **sidecar memory service** used by Archie and other potential agents in Open WebUI.

It focuses on:
- responsibilities and boundaries
- storage and recall semantics
- failure behavior
- long‑term extensibility

This is not a setup guide.

---

## Purpose

The memory service provides **durable, semantic memory** for agents that require long‑term context across sessions.

Its role is intentionally narrow:

- store distilled memory entries
- generate embeddings
- persist data to disk
- return semantically relevant memories

All decision‑making about *when* and *how* to use memory lives outside this service.

---

## Why a Separate Service?

Memory is externalized for three reasons:

1. **Durability**
   - Memory survives UI refreshes and container restarts
   - Memory is not bounded by prompt length

2. **Separation of Concerns**
   - Open WebUI orchestrates
   - The LLM reasons
   - The memory service stores and recalls

3. **Replaceability**
   - The vector store can be swapped without touching Open WebUI
   - Embedding models can change independently

This avoids hidden state and tight coupling.

---

## High‑Level Architecture

```
Agent (Archie)
  │
  │ recall(query)
  ▼
Memory Service
├── FastAPI
├── Embedding Model
└── Vector Store (persistent)
```

The service is accessed via HTTP and is stateless aside from its persistent storage.

---

## API Surface

The service exposes a minimal API:

### `/store`
Stores a memory entry.

Typical payload:
- persona
- memory type (preference, decision, context)
- content
- confidence
- tags

### `/recall`
Returns semantically relevant memories.

Typical payload:
- query text
- persona
- top‑K
- optional thresholds

The service does not interpret or modify memory meaning.

---

## Persistence Model

- Memory is stored in a **disk‑backed vector store**
- Embeddings are generated at write time
- Storage survives container restarts
- Memory is scoped by persona

Persistence is explicit and inspectable.

---

## Recall Semantics

Recall is **best‑effort**:

- If recall succeeds, results are returned
- If recall fails (timeout, crash), the caller proceeds normally

This guarantees:
- no hard dependency
- no chat outages
- graceful degradation

Memory enhances behavior; it never blocks it.

---

## What the Memory Service Does *Not* Do

The service intentionally does not:

- decide when memory should be recalled
- modify prompts
- write memory automatically
- summarize conversations
- interpret model output

Those concerns belong to the orchestrator (Open WebUI).

---

## Extensibility

This service can evolve independently to support:

- different vector databases
- hybrid search (BM25 + vectors)
- memory decay or TTL
- confidence weighting
- multi‑tenant isolation

None of these changes require modifying Open WebUI.

---

## Design Principle

> **Memory should be explicit, durable, and replaceable.**

This service enforces that principle by staying simple and focused.
