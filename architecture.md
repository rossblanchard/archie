# Architecture: Archie and Memory‑Aware Agents in Open WebUI

This document describes the **system‑level architecture** used to build *Archie*, a memory‑aware agent running inside Open WebUI.

It explains:
- the major components
- how they interact
- why boundaries exist where they do
- and how this design scales to additional agents

This is an architectural document, not a setup guide.

---

## Architectural Goals

The architecture is driven by the following goals:

1. **Durable memory**
   - Memory must survive UI refreshes and container restarts.
   - Memory must not depend on prompt length or chat history.

2. **Agent‑specific behavior**
   - Memory applies to Archie only.
   - Other agents remain stateless unless explicitly configured.

3. **Clear system boundaries**
   - Open WebUI orchestrates.
   - Memory is an external service.
   - Models remain stateless compute.

4. **Upgrade tolerance**
   - Open WebUI upgrades should require minimal rework.
   - Frontend rebuilds should be avoided.

5. **Future extensibility**
   - Additional agents should reuse the same patterns.
   - Memory, tools, and file outputs should be independently configurable.

---

## High‑Level System Overview

At runtime, the system consists of three cooperating components:

```
Open WebUI (Docker)
├── UI (browser)
├── Backend (Python)
│   ├── Agent resolution
│   ├── Prompt construction
│   └── LLM routing
│
├── Agents
│   ├── Archie (memory‑aware)
│   └── Others (stateless)
│
└── External APIs
    └── OpenAI (LLM inference)

Sidecar Memory Service (Docker)
├── FastAPI
├── Embedding model
└── Vector store (persistent)
```

The key design choice is that **memory is externalized** and accessed only when needed.

---

## Request Flow (End‑to‑End)

A single chat message follows this path:

1. **User submits a message via the Open WebUI UI**
2. Open WebUI backend receives the request
3. Backend resolves:
   - user
   - session
   - selected model/persona
4. **Agent resolution occurs**
   - If the model corresponds to Archie, memory rules apply
   - Otherwise, the request proceeds unchanged
5. **Memory recall (Archie only)**
   - Latest user message is extracted
   - A semantic recall request is sent to the memory service
6. **Prompt augmentation**
   - Recalled memories are injected into the system prompt
   - Injection is additive and non‑destructive
7. **LLM call**
   - OpenAI (or compatible backend) receives the final prompt
8. **Response streams back to the UI**

At no point does the LLM itself store state.

---

## Agent Resolution

Agents are resolved **by persona / model name**, using a simple and explicit rule:

- If the selected model name contains `"archie"`:
  - Archie’s configuration is applied
- Otherwise:
  - No agent‑specific behavior is triggered

This keeps agent behavior:
- deterministic
- inspectable
- easy to extend later into a config‑driven registry

---

## Memory Architecture

### Why Memory Is External

Prompt‑only memory was rejected because it is:

- ephemeral
- token‑bounded
- opaque
- difficult to reason about
- impossible to share safely across agents

Instead, memory is treated as **infrastructure**.

---

### Memory Service Responsibilities

The sidecar memory service is responsible for:

- storing distilled memory entries
- generating embeddings
- persisting data to disk
- returning semantically relevant memories

It is **not** responsible for:
- deciding *when* to recall
- modifying prompts
- interpreting model output

Those responsibilities remain in Open WebUI.

---

### Memory Recall Semantics

Memory recall is **best‑effort**:

- If recall succeeds → memories are injected
- If recall fails → chat proceeds normally

This ensures:
- no hard dependency on memory availability
- no chat outages due to memory failures

---

## Prompt Injection Strategy

Recalled memory is injected into the **system prompt**, not as user content.

The injected block follows these principles:

- Explicitly labeled as memory context
- High‑confidence, distilled statements
- Instructions to the model *not* to parrot verbatim
- Appended to the existing system message

This keeps memory:
- latent
- authoritative
- non‑intrusive

---

## Backend Integration Point

The integration hook lives in:

```
open_webui/apps/webui/routers/chat.py
```

Specifically:
- inside `generate_chat_completion()`
- immediately before the LLM provider call
- after `form_data["messages"]` is finalized

This hook point was chosen because:

- the model is known
- the prompt is complete
- no tokens have been sent yet
- streaming has not started

This is the last safe modification point.

---

## Docker Architecture

### Container Roles

Two long‑running containers are used:

1. **Open WebUI**
   - UI + backend
   - Stateless with respect to memory
   - Uses a named volume for chats and settings

2. **Memory Service**
   - Stateless compute + persistent storage
   - Uses a mounted volume for vector data

---

### Image Layering Strategy

To avoid rebuilding the Open WebUI frontend, images are layered as follows:

```
ghcr.io/open-webui/open-webui:main
        ↓
local open-webui-v2
  (backend Python patched)
```

Only backend code is replaced:

```
COPY backend/open_webui /app/open_webui
```

Benefits:
- avoids Node / Vite / Pyodide builds
- faster iteration
- safer upgrades
- smaller diff surface

---

## Failure Modes and Guarantees

### Guaranteed

- Chats persist across restarts
- Memory persistence is durable
- Non‑Archie agents are unaffected
- Memory failure does not break chat

### Not Guaranteed

- Perfect recall
- Global consistency across agents
- Automatic memory write‑back

These are intentional tradeoffs.

---

## Scaling the Architecture

This architecture naturally supports:

- additional agents with different memory policies
- agent‑specific tools and file outputs
- multiple memory backends
- future config‑driven agent registries

All within a **single Open WebUI instance**.

---

## Architectural Principle

The guiding principle is simple:

> **Models are stateless. Systems carry memory.**

By enforcing this boundary, the system remains:
- debuggable
- evolvable
- understandable months later

Archie is the first application of this pattern, not the last.
