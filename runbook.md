# Runbook: Operating and Recovering the Archie System

This runbook documents **operational procedures**, **failure modes**, and **recovery steps** for the Archie memory‑aware agent system.

It is intended for:
- operators
- maintainers
- future you at 2am

This document assumes familiarity with Docker and Open WebUI.

---

## System Components (Operational View)

At runtime, the system consists of:

- **Open WebUI container**
  - UI and backend orchestration
  - Stateless with respect to memory
  - Uses a named Docker volume for chats/settings

- **Archie memory service container**
  - FastAPI service
  - Persistent vector store
  - Persona‑scoped semantic recall

Both containers must be running for full functionality.

---

## Normal Startup Procedure

### Start the memory service

```bash
docker run -d \
  --name archie-memory \
  -p 8001:8000 \
  -v archie-memory:/memory \
  archie-memory
```

### Start Open WebUI

```bash
docker run -d \
  --name open-webui \
  -p 3000:8080 \
  -v open-webui:/app/backend/data \
  open-webui-v2
```

---

## Health Checks

### Check containers are running

```bash
docker ps
```

Expected:
- `open-webui` → Up
- `archie-memory` → Up

---

### Check Open WebUI logs

```bash
docker logs open-webui
```

Look for:
- Python tracebacks
- Import errors
- Startup failures

---

### Check memory service logs

```bash
docker logs archie-memory
```

Look for:
- FastAPI startup messages
- Chroma initialization errors
- Permission issues on `/memory`

---

## Common Failure Modes

---

### Open WebUI fails to start

**Symptoms**
- Container exits immediately
- UI unavailable on port 3000

**Likely causes**
- Backend code copied to wrong path
- Import errors from custom helpers

**Recovery**
1. Inspect logs:
   ```bash
   docker logs open-webui
   ```
2. Verify backend path:
   - Custom code must live under `/app/open_webui`
3. Rebuild image:
   ```bash
   docker build -t open-webui-v2 .
   ```
4. Restart container

---

### Memory service unavailable

**Symptoms**
- Archie behaves like a stateless agent
- No crashes or UI errors

**Explanation**
- Memory recall is best‑effort by design

**Recovery**
1. Restart memory container:
   ```bash
   docker restart archie-memory
   ```
2. Verify persistence directory exists:
   ```bash
   docker volume inspect archie-memory
   ```

No Open WebUI restart is required.

---

### Docker build fails with Node / heap errors

**Symptoms**
- Errors referencing `npm`, `vite`, `pyodide`
- JavaScript heap out of memory

**Explanation**
- Frontend rebuild triggered accidentally

**Corrective action**
Ensure Dockerfile uses backend‑only layering:

```dockerfile
FROM ghcr.io/open-webui/open-webui:main
COPY backend/open_webui /app/open_webui
```

Never rebuild the frontend unless explicitly required.

---

## Rollback Procedure

If the system must be reverted immediately:

### Stop current container

```bash
docker stop open-webui
docker rm open-webui
```

### Run last known good image

```bash
docker run -d \
  --name open-webui \
  -p 3000:8080 \
  -v open-webui:/app/backend/data \
  open-webui-pre-archie
```

Chats and settings remain intact.

---

## Validation Checklist (Post‑Recovery)

After recovery:

- [ ] Open WebUI loads in browser
- [ ] Existing chats are visible
- [ ] Archie responds normally
- [ ] Other agents unaffected
- [ ] Memory recall works when memory service is running
- [ ] System degrades gracefully if memory is stopped

---

## Operational Principle

> **Memory enhances behavior, but must never block it.**

This runbook enforces that principle by prioritizing graceful degradation and fast recovery.
