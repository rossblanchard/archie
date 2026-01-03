# ADR‑01: External Semantic Memory Service

## Status

Accepted

---

## Context

The system requires **durable memory** for certain agents (notably Archie) that:

- persists across sessions
- is not bounded by prompt size
- can evolve independently
- does not affect all agents globally

Open WebUI provides built‑in retrieval‑augmented generation (RAG) capabilities that could potentially satisfy this need.

A decision was required.

---

## Decision

We chose to implement **an external, sidecar semantic memory service** instead of relying on Open WebUI’s built‑in RAG features.

---

## Considered Options

### Option A — Use Open WebUI Built‑In RAG

Pros:
- Integrated UI
- Less initial infrastructure
- Faster initial setup

Cons:
- Memory tightly coupled to Open WebUI internals
- Less transparent storage semantics
- Harder to scope memory per agent
- Harder to replace or evolve independently
- Less instructional value for understanding how memory systems work

---

### Option B — External Memory Service (Chosen)

Pros:
- Explicit, inspectable memory
- Durable across restarts
- Clear separation of concerns
- Agent‑scoped behavior
- Replaceable vector store
- Educational value (learning how memory systems work end‑to‑end)

Cons:
- Additional service to operate
- Slightly more backend complexity
- Requires explicit integration

---

## Rationale

The external memory service was chosen because:

1. **Architectural clarity**
   - Memory becomes infrastructure, not a feature toggle

2. **Agent isolation**
   - Only Archie receives memory behavior
   - Other agents remain unaffected

3. **Long‑term flexibility**
   - Vector stores, embeddings, and policies can change independently

4. **Learning objective**
   - Understanding how semantic memory works in practice was a primary goal
   - The system is educational as well as functional

5. **Upgrade safety**
   - Open WebUI upgrades are less risky with minimal internal coupling

---

## Consequences

### Positive

- Memory behavior is explicit and testable
- Failure modes are well‑defined
- Architecture scales cleanly to multiple agents
- Knowledge gained applies beyond Open WebUI

### Negative

- One additional container to run
- Slightly higher operational overhead

These tradeoffs are acceptable for the system’s goals.

---

## Notes

This decision does not preclude using Open WebUI’s built‑in RAG in the future.

If requirements change (e.g., tighter UI integration, simpler deployment), the architecture allows reevaluating this choice without rewriting the entire system.

---

## Principle

> **We favored architectural clarity and learning over minimal configuration.**
