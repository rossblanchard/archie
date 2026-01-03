# Code Map: Archie System

This document maps **actual code files** to their responsibilities.

It exists to answer:
- “Where does this logic live?”
- “What did we actually change?”
- “What breaks if I touch this?”

---

## Open WebUI Backend (Modified)

### `open_webui/apps/webui/routers/chat.py`

**Responsibility**
- Main chat request orchestration
- Model selection
- Payload preparation
- LLM routing

**Modification**
- Added Archie memory injection hook inside:
  ```
  generate_chat_completion()
  ```
- Hook executes:
  - after `form_data["messages"]` is finalized
  - before LLM provider call

---

### `open_webui/utils/agents.py`

**Responsibility**
- Resolve agent configuration based on model/persona
- Gate agent‑specific behavior

---

### `open_webui/utils/memory.py`

**Responsibility**
- Call external memory service
- Handle timeouts and failures gracefully
- Never raise exceptions to chat pipeline

---

### `open_webui/utils/prompt_injection.py`

**Responsibility**
- Inject recalled memory into system prompt
- Preserve existing system content
- Prevent parroting

---

## Memory Service (Sidecar)

### `app.py` (memory service)

**Responsibility**
- FastAPI application
- `/store` endpoint
- `/recall` endpoint

---

### Vector Store (Chroma)

**Responsibility**
- Persistent embedding storage
- Semantic similarity search

---

## Docker Artifacts

### `Dockerfile` (Open WebUI)

```dockerfile
FROM ghcr.io/open-webui/open-webui:main
COPY backend/open_webui /app/open_webui
```

**Responsibility**
- Layer backend changes only
- Avoid frontend rebuild

---

### Docker Volumes

- `open-webui`
  - Chats, users, settings
- `archie-memory`
  - Vector store persistence

---

## Terminal Commands Used

### Build Open WebUI image

```bash
docker build -t open-webui-v2 .
```

### Run Open WebUI

```bash
docker run -d \
  --name open-webui \
  -p 3000:8080 \
  -v open-webui:/app/backend/data \
  open-webui-v2
```

### Run Memory Service

```bash
docker run -d \
  --name archie-memory \
  -p 8001:8000 \
  -v archie-memory:/memory \
  archie-memory
```

---

## Principle

> **Know exactly where behavior lives.**

This code map makes implicit behavior explicit.
