# Archie: A Memory‑Aware Agent in Open WebUI

This repository documents how to build **Archie**, a custom agent running inside **Open WebUI** with **durable, semantic memory** backed by a sidecar vector store.

The goal is not to create “just another chatbot,” but a **long‑lived, architecture‑aware assistant** that:

- Remembers preferences and decisions across sessions
- Behaves differently from other agents
- Scales cleanly as you add more agents later

---

## Overview

At a high level, this system consists of:

- **Open WebUI**, running locally in Docker
- A custom agent persona (“Archie”)
- A **sidecar memory service** providing persistent semantic recall
- A small **backend integration hook** inside Open WebUI
- A Docker layering strategy that avoids rebuilding the frontend

The result is a single Open WebUI instance (one URL) hosting multiple agents, where only Archie has durable memory.

---

## Design Goals

1. **Durable memory**  
   Memory must survive container restarts and UI reloads.

2. **Agent‑specific behavior**  
   Memory should apply only to Archie, not globally.

3. **Backend‑level integration**  
   No prompt‑only hacks or UI glue.

4. **Upgrade tolerance**  
   Changes should survive Open WebUI upgrades with minimal rework.

5. **Future extensibility**  
   Easy to add more agents with different tools, memory, or file access.

---

## High‑Level Architecture

```
Open WebUI (single instance)
├── Archie (agent)
│   └── Memory recall enabled
├── Other agents
│   └── No persistent memory
└── Backend hook
    └── Calls memory service (Archie only)

Sidecar Memory Service
├── FastAPI
├── ChromaDB (persistent)
└── SentenceTransformers
```

**Key idea:** memory is a service, not a prompt trick.

---

## Prerequisites

- macOS or Linux
- Docker Desktop
- Git
- An OpenAI API key
- Comfort editing Python files

---

## Step 1: Running Open WebUI Locally with Docker

```bash
docker run -d \
  --name open-webui \
  -p 3000:8080 \
  -v open-webui:/app/backend/data \
  ghcr.io/open-webui/open-webui:main
```

The named volume preserves chats, users, and settings.

---

## Step 2: Creating a Custom Agent (“Archie”)

Inside Open WebUI:

- Create a new model/persona named **Archie**
- Point it at the OpenAI API
- Give it a system prompt emphasizing:
  - architectural reasoning
  - explicit tradeoffs
  - peer‑level tone

At this stage, Archie has **no memory**.

---

## Step 3: Why Prompt Memory Isn’t Enough

Prompt‑only memory:

- Is lost on refresh
- Is token‑bounded
- Is hard to reason about
- Does not scale to multiple agents

This design uses:

- Semantic embeddings
- Explicit persistence
- Controlled recall

Memory becomes infrastructure, not hidden state.

---

## Step 4: Building the Sidecar Memory Service

A separate Docker container runs a FastAPI app with:

- **ChromaDB** for disk‑backed storage
- **SentenceTransformers** for embeddings
- JSON endpoints:
  - `/store`
  - `/recall`

Properties:

- Persistent across restarts
- Persona‑scoped
- Best‑effort (never breaks chat)

---

## Step 5: Integrating Memory into Open WebUI

Instead of modifying the UI, we patch the backend:

- File: `open_webui/apps/webui/routers/chat.py`
- Function: `generate_chat_completion()`

The hook:

1. Detects the selected model
2. Checks if it matches “Archie”
3. Extracts the latest user message
4. Calls the memory service
5. Injects recalled memories into the **system prompt**
6. Proceeds normally to the LLM call

Other agents are untouched.

---

## Step 6: Docker Image Strategy

Avoid rebuilding the frontend.

Use a layered Dockerfile:

```dockerfile
FROM ghcr.io/open-webui/open-webui:main
COPY backend/open_webui /app/open_webui
```

Benefits:

- No Node / Vite / Pyodide build
- Fast, reliable builds
- Upgrade‑friendly
- Backend‑only customization

---

## Step 7: Testing and Validation

1. Restart Open WebUI
2. Open existing chats (confirm persistence)
3. Chat with Archie:
   - Ask preference‑based questions
   - Look for consistent behavior
4. Chat with other agents:
   - Confirm unchanged behavior

If memory is unavailable, Archie still responds normally.

---

## Failure Modes and Tradeoffs

Accepted tradeoffs:

- Slight backend complexity
- One additional service

Avoided problems:

- Silent context loss
- Prompt bloat
- Global memory contamination
- Debugging ambiguity

Failures degrade gracefully.

---

## Extending This Pattern

This architecture supports:

- Additional agents
- Agent‑specific memory policies
- File outputs per agent
- Tool routing per agent
- Long‑term knowledge bases

All within **one Open WebUI instance**.

---

## Closing Thoughts

**Models don’t remember. Systems do.**

By externalizing memory, you gain:

- Durability
- Clarity
- Control
- Extensibility

Archie is just the flagship.
