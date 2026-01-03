# ADR‑002: Docker Image Layering Strategy

## Status

Accepted

---

## Context

Open WebUI is distributed as a prebuilt Docker image that includes:

- a compiled frontend (Vite, Pyodide, Node)
- a Python backend

When modifying backend behavior (e.g., adding memory integration), two approaches were possible:

1. Rebuild Open WebUI from source
2. Layer backend changes on top of the official image

A decision was required.

---

## Decision

We chose to **layer backend‑only changes on top of the official Open WebUI image**, instead of rebuilding the entire stack from source.

---

## Considered Options

### Option A — Build Open WebUI from Source

Pros:
- Full control over build
- Single image artifact

Cons:
- Frontend build is memory‑intensive
- Frequent Node heap failures
- Slower iteration
- Fragile across upstream changes
- Unnecessary complexity for backend‑only changes

---

### Option B — Backend‑Only Image Layering (Chosen)

Pros:
- Reuses official frontend build
- Avoids Node / Vite / Pyodide entirely
- Faster, safer builds
- Smaller diff surface
- Easier upgrades
- Clear separation of concerns

Cons:
- Requires understanding Open WebUI’s package layout
- Slightly more Dockerfile precision required

---

## Implementation

The Dockerfile used:

```dockerfile
FROM ghcr.io/open-webui/open-webui:main
COPY backend/open_webui /app/open_webui
```

Key detail:
- `/app/open_webui` is the Python package actually imported at runtime
- Copying only this directory safely overrides backend logic

---

## Consequences

### Positive

- Dramatically faster builds
- No frontend tooling required
- Reliable local iteration
- Reduced operational risk

### Negative

- Must re‑apply patch after major Open WebUI refactors
- Requires awareness of backend package structure

These tradeoffs are acceptable given the project’s goals.

---

## Operational Guidance

- Never rebuild frontend unless required
- Treat frontend as opaque
- Keep backend patches minimal and explicit
- Validate after Open WebUI upgrades

---

## Principle

> **Do not rebuild what you did not change.**

This principle reduces risk and increases maintainability.
