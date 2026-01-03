# ADR‑03: Single Open WebUI Instance with Multiple Agents

## Status

Accepted

---

## Context

The system supports multiple agents with different behaviors, memory policies, and tools.

A decision was required on how these agents should be hosted and presented:

- Separate Open WebUI instances per agent  
- Or a single Open WebUI instance hosting multiple agents

---

## Decision

We chose to use **a single Open WebUI instance hosting multiple agents**, differentiated by model/persona and backend behavior.

---

## Considered Options

### Option A — Separate Open WebUI Instances per Agent

Pros:
- Strong isolation
- Simpler mental model per agent
- No cross‑agent risk

Cons:
- Multiple URLs
- Duplicated infrastructure
- Fragmented user experience
- Harder to share context or tooling
- Operational overhead grows linearly with agents

---

### Option B — Single Open WebUI Instance, Multiple Agents (Chosen)

Pros:
- One UI, one URL
- Shared authentication and chat history
- Centralized orchestration
- Agent behavior controlled explicitly in backend
- Easier to add new agents
- Better user ergonomics

Cons:
- Requires careful agent gating
- Requires discipline in backend logic
- Bugs could affect multiple agents if poorly scoped

---

## Rationale

The single‑instance approach was chosen because:

1. **User experience**
   - Switching agents is easier than switching UIs
   - Conversations remain discoverable in one place

2. **Architectural clarity**
   - Open WebUI acts as an orchestrator
   - Agents are behavioral configurations, not separate systems

3. **Scalability**
   - Adding agents is additive, not multiplicative
   - Shared services (memory, tools) can be reused

4. **Intentional isolation**
   - Agent behavior is gated explicitly by model/persona
   - No implicit global behavior

---

## Consequences

### Positive

- Cleaner operational model
- Faster iteration on new agents
- Easier documentation and onboarding
- Enables agent‑specific memory and tools

### Negative

- Requires strict discipline in backend hooks
- Testing must verify agent isolation

These tradeoffs are acceptable and intentional.

---

## Principle

> **Agents are behaviors, not deployments.**

A single orchestrator with explicit agent rules scales better than many isolated UIs.
