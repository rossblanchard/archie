# Security Considerations

This document describes security assumptions and practices for the Archie system.

It is not a compliance document.

---

## API Keys

### OpenAI API Key

- Stored as an environment variable
- Never committed to source control
- Used only by Open WebUI backend

Recommended:
- `.env` file
- Docker secrets (if applicable)

---

## Memory Service Exposure

- Memory service runs on a local Docker network
- No public exposure by default
- No authentication required for local use

Assumption:
- Local, trusted environment

If exposed externally:
- Add authentication
- Restrict network access
- Consider TLS

---

## Data Stored in Memory

Memory entries may include:
- preferences
- architectural decisions
- workflow habits

They should **not** include:
- secrets
- credentials
- tokens
- personal sensitive data

Memory is semantic, not secure storage.

---

## Agent Isolation

- Agent behavior is explicitly gated
- No shared global memory
- No implicit cross‑agent access

Failure to gate agents correctly is a security risk.

---

## Docker Isolation

- Containers are isolated by default
- Volumes persist data
- No container runs with elevated privileges

Avoid:
- mounting sensitive host directories
- running as root unnecessarily

---

## Logging

- Avoid logging raw memory content in production
- Avoid logging prompts containing sensitive data
- Logs should aid debugging, not leak data

---

## Principle

> **Treat memory as knowledge, not secrets.**

Security is maintained by explicit boundaries and conservative assumptions.
