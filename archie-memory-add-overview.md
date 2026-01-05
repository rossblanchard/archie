# Archie memory project general overview for knowledge/context injection in OpenWebUI

This information shall inform and guide model "Archie" for future projects and general communication

Other models may use this information but know that their persona did not contribute to this project.

# Knowledge Summary: Building Archie — A Memory‑Aware Agent in Open WebUI

This document is a distilled knowledge summary of a complete project to design, implement, and document **Archie**, a memory‑aware agent running inside **Open WebUI**. It is intended for **context injection** into future conversations so that the system immediately understands the project’s goals, architecture, constraints, and philosophy without replaying raw chat history.

This summary captures **what was built**, **why decisions were made**, and **how the system is meant to behave**.

---

## Project Intent

The primary goal of this project was to move beyond a stateless chatbot and build an **agent with durable, intentional memory** that:

- Persists across chats and restarts
- Behaves consistently over time
- Is scoped to a specific agent (Archie)
- Is architecturally understandable and maintainable
- Can scale to multiple agents without duplication or fragility

A secondary goal was **learning**: to understand how memory, orchestration, and agent behavior actually work under the hood, rather than relying on opaque built‑in features.

---

## Core Architectural Philosophy

The system is guided by a few explicit principles:

- **Models are stateless. Systems carry memory.**
- **Memory is infrastructure, not prompt text.**
- **Behavior should be explicit, inspectable, and reversible.**
- **User trust requires deliberate memory write‑back.**
- **Agent behavior is a configuration, not a deployment.**

These principles shaped every major decision.

---

## High‑Level System Architecture

At runtime, the system consists of two cooperating Docker containers:

1. **Open WebUI**
   - Provides the UI and backend orchestration
   - Hosts multiple agents within a single instance
   - Remains stateless with respect to memory
   - Uses a Docker volume for chats, users, and settings

2. **Memory Service (Sidecar)**
   - A FastAPI application
   - Uses a persistent vector store (Chroma)
   - Generates embeddings via SentenceTransformers
   - Exposes `/store` and `/recall` endpoints
   - Persists memory across restarts

Open WebUI decides **when** memory is relevant.  
The memory service decides **what** is stored and retrieved.

---

## Why External Memory (Not Prompt Memory)

Prompt‑only memory was explicitly rejected because it:

- Is lost on refresh
- Is bounded by token limits
- Pollutes context over time
- Is hard to reason about or debug
- Does not scale across agents

Instead, memory is **externalized**, durable, and accessed on demand. This allows:

- Clear failure modes
- Graceful degradation
- Independent evolution of memory storage
- Explicit boundaries between reasoning and persistence

---

## Agent Model: Single UI, Multiple Agents

The system uses **one Open WebUI instance** hosting **multiple agents**, rather than multiple UIs.

Agents are differentiated by:
- model/persona name
- backend behavior gating
- memory policy
- tools (future)

Archie is the **flagship agent**, responsible for:
- systems architecture
- backend reasoning
- long‑term project continuity

Other agents remain stateless unless explicitly extended.

This avoids duplicated infrastructure, fragmented UX, and operational overhead.

---

## Backend Integration Strategy

Memory integration is implemented **entirely in the backend**, not the UI.

The integration hook lives in:

```
open_webui/apps/webui/routers/chat.py
```

Specifically inside:

```
generate_chat_completion()
```

The hook executes:
- after `form_data["messages"]` is finalized
- before the LLM provider is called
- before streaming begins

At this point:
- the selected model is known
- the system prompt exists
- no tokens have been sent

This is the **last safe mutation point**.

---

## Memory Recall Policy (Hybrid)

The system uses a **hybrid recall policy**:

- **Cheap recall** on every Archie request
  - Small `k`
  - Query = latest user message
  - Low latency
  - Surfaces obvious preferences or decisions

- **Heavier recall** only when signaled
  - New conversations
  - Explicit references to past decisions
  - “remember”, “as we discussed”, etc.

This balances:
- relevance
- cost
- prompt focus
- behavioral consistency

---

## Prompt Injection Strategy

Recalled memory is injected into the **system prompt**, not user content.

Injection rules:
- Additive, not destructive
- Clearly labeled as memory context
- High‑confidence, distilled statements
- Instructions not to parrot verbatim

This keeps memory:
- authoritative
- latent
- non‑intrusive

---

## Memory Write‑Back Policy (Explicit)

The system intentionally avoids automatic memory creation.

Memory write‑back follows this policy:

1. Archie may **propose** a memory
2. The user must **explicitly confirm**
3. Only then is `/store` called

This avoids:
- memory pollution
- incorrect assumptions
- loss of trust
- opaque behavior

Memory types include:
- preferences
- architectural decisions
- project summaries

Not included:
- raw chat transcripts
- transient tasks
- speculative ideas

---

## Docker Strategy

Building Open WebUI from source triggers heavy frontend builds (Node, Vite, Pyodide) that frequently fail due to memory limits.

To avoid this, the system uses **backend‑only image layering**:

```
FROM ghcr.io/open-webui/open-webui:main
COPY backend/open_webui /app/open_webui
```

This:
- reuses the official frontend
- overrides only backend Python code
- speeds up builds dramatically
- reduces upgrade risk

Two Docker volumes are used:
- `open-webui` → chats, users, settings
- `archie-memory` → vector store persistence

---

## Operational Characteristics

- Memory service is **best‑effort**
- If memory is down, Archie behaves statelessly
- No chat failures due to memory outages
- Rollback is immediate by running a prior image
- Logging can be added to validate recall

The system prioritizes **graceful degradation**.

---

## Documentation as First‑Class Output

A major outcome of this project is **comprehensive documentation**, including:

- README (narrative overview)
- Architecture document
- Memory service deep dive
- Open WebUI integration details
- Runbook (operations & recovery)
- ADRs capturing design decisions
- Security considerations
- Code map linking files to responsibilities

Documentation is treated as **canonical knowledge**, complementary to memory.

---

## Testing and Validation Philosophy

Memory correctness is tested behaviorally, not by inspecting the DB.

Validation methods include:
- default concision vs explicit “explain” prompts
- cross‑chat consistency
- negative controls using non‑Archie agents
- deterministic backend hooks (e.g., “banana” test rules)

Preference memory influences **tendencies**, not scripts.

---

## Preferences Stored in Memory

Examples of preferences intentionally stored:

- Ross prefers concise answers by default
- Long‑form documentation should be delivered as a single fenced code block
- Archie is the agent for systems and architecture work

These are **style and workflow preferences**, not hard rules.

---

## What This System Is *Not*

- Not a fully autonomous agent
- Not automatic long‑term chat logging
- Not a replacement for human judgment
- Not dependent on Open WebUI’s built‑in RAG

It is intentionally **explicit, controlled, and understandable**.

---

## Key Takeaway

The most important lesson from this project:

> **You don’t want to remember the conversation.  
> You want to remember what the conversation meant.**

This system captures meaning through:
- distilled memory
- explicit decisions
- documented architecture
- deliberate write‑back

That combination creates continuity **without fragility**.

---

## Summary Statement

Archie is a memory‑aware agent built inside Open WebUI using a sidecar semantic memory service. The system emphasizes explicit write‑back, agent‑scoped behavior, backend integration, and durable infrastructure over prompt‑level hacks. It was built both to function and to teach how memory‑aware systems actually work.

This document exists to ensure that understanding persists across future conversations.
