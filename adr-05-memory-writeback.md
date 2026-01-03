# ADR‑005: Memory Write‑Back Policy

## Status

Accepted

---

## Context

The system includes a durable semantic memory service used by certain agents (notably Archie).

A decision was required regarding **how and when new information should be written back into memory**.

Key concerns included:
- accidental or noisy memory growth
- storing incorrect or transient information
- loss of user trust due to “over‑remembering”
- unclear ownership of what constitutes memory‑worthy information

---

## Decision

We chose an **explicit, user‑confirmed memory write‑back policy**.

Memory is written **only when explicitly requested or confirmed**, never automatically.

---

## Considered Options

### Option A — Automatic Write‑Back on Every Interaction

Pros:
- Maximum recall
- Minimal user effort
- Simple implementation

Cons:
- Memory pollution
- Incorrect assumptions stored as fact
- Difficult to debug or prune
- Loss of user trust
- Hard to distinguish signal from noise

---

### Option B — Heuristic‑Based Automatic Write‑Back

Pros:
- Reduced noise compared to full automation
- Some contextual intelligence

Cons:
- Heuristics are brittle
- False positives are inevitable
- Behavior becomes opaque
- Users cannot easily predict what is remembered

---

### Option C — Explicit, User‑Confirmed Write‑Back (Chosen)

Pros:
- High signal‑to‑noise ratio
- User retains control
- Memory remains intentional and inspectable
- Trust is preserved
- Easier to reason about and debug

Cons:
- Requires explicit user action
- Slightly more interaction overhead

---

## Rationale

The explicit write‑back policy was chosen because:

1. **Memory implies authority**
   - Stored memory is treated as fact by the agent
   - Incorrect memory is worse than missing memory

2. **User intent matters**
   - Only the user can reliably determine what is worth remembering
   - Silent memory creation erodes trust

3. **System transparency**
   - Explicit write‑back makes memory behavior predictable
   - Users can reason about what the system “knows”

4. **Operational hygiene**
   - Memory remains small, high‑quality, and durable
   - Pruning and inspection remain tractable

5. **Alignment with architectural philosophy**
   - Memory is infrastructure, not emergent behavior
   - Decisions should be deliberate, not inferred

---

## Write‑Back Flow (Conceptual)

1. User states or implies something potentially memory‑worthy
2. Agent proposes a memory candidate
3. User explicitly confirms (e.g., “remember that”)
4. Memory is distilled and stored via `/store`
5. Memory becomes available for future recall

At no point does the agent write memory silently.

---

## What Qualifies as Memory

Typical memory candidates include:

- Stable preferences
- Long‑term workflow habits
- Architectural decisions
- Explicit rules or constraints

Non‑candidates include:

- Transient tasks
- One‑off instructions
- Speculative ideas
- Unverified assumptions

---

## Consequences

### Positive

- High‑quality memory corpus
- Strong user trust
- Predictable system behavior
- Easier long‑term maintenance

### Negative

- Slightly higher interaction cost
- Requires user education and habit formation

These tradeoffs are acceptable and intentional.

---

## Notes

This policy does not preclude future enhancements such as:
- memory proposal ranking
- confidence scoring
- memory review or pruning tools

However, **explicit user consent remains the core invariant**.

---

## Principle

> **Memory should be deliberate, not accidental.**

A system that remembers without permission will eventually be distrusted.
