# ADR‑004: Hybrid Memory Recall Policy

## Status

Accepted

---

## Context

Memory recall has cost and complexity:

- embedding computation
- vector search
- prompt injection
- token usage

Always performing deep recall risks:
- latency
- prompt bloat
- irrelevant context

Never performing recall risks:
- inconsistent behavior
- forgotten preferences

A policy decision was required.

---

## Decision

We adopted a **hybrid memory recall policy**:

- **Cheap recall on every request**
- **Heavier recall only when signals indicate it is useful**

---

## Considered Options

### Option A — Always Recall Everything

Pros:
- Maximum context
- Simple implementation

Cons:
- High cost
- Prompt bloat
- Irrelevant memory pollution
- Reduced model focus

---

### Option B — Recall Only on Explicit User Request

Pros:
- Minimal overhead
- Predictable behavior

Cons:
- Misses implicit context
- Places burden on the user
- Feels “forgetful”

---

### Option C — Hybrid Recall (Chosen)

Pros:
- Balanced cost vs benefit
- Memory feels “ambient”
- Prompt remains focused
- Easy to evolve

Cons:
- Slightly more logic
- Requires heuristics

---

## Implementation Strategy

### Cheap Recall (Default)

Performed on every Archie request:

- Small `k` (e.g., 3)
- Query = latest user message
- Low latency
- Injected into system prompt

Purpose:
- Surface obvious preferences or decisions
- Maintain behavioral consistency

---

### Heavy Recall (Conditional)

Triggered only when:

- User references past decisions
- Conversation is new
- Cheap recall returns strong signals
- Explicit “remember” or “as we discussed” language appears

Heavier recall may include:
- Larger `k`
- Summarized conversation context
- Higher confidence thresholds

---

## Consequences

### Positive

- Memory feels present but not noisy
- Cost is controlled
- Behavior scales with complexity

### Negative

- Requires tuning over time
- Heuristics may evolve

This tradeoff is acceptable.

---

## Principle

> **Memory should be available, not overwhelming.**

Recall is a policy decision, not a storage decision.
