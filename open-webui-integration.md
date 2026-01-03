# Open WebUI Integration

This document explains **how and where** the memory service is integrated into Open WebUI.

It is intended for:
- backend maintainers
- future upgrades
- anyone modifying agent behavior

---

## Integration Philosophy

Open WebUI is treated as an **orchestrator**, not a memory store.

The integration follows these rules:

- no UI changes
- no prompt templates
- no global side effects
- no modification of streaming behavior
- agent‑specific gating

The goal is minimal, durable change.

---

## Integration Point

The integration hook lives in:

```
open_webui/apps/webui/routers/chat.py
```

Specifically inside:

```
async def generate_chat_completion(...)
```

This function is responsible for:
- receiving the chat request
- resolving the model/persona
- preparing the payload
- routing to the LLM backend

---

## Why This Hook Point?

The hook is placed:

- **after** `form_data["messages"]` is finalized
- **before** the LLM provider is called
- **before** streaming begins

At this point:
- the selected model is known
- the system prompt exists
- no tokens have been sent

This is the last safe modification point.

---

## Agent Resolution

Agent behavior is gated by model name.

Example rule:

- If model name contains `"archie"`:
  - apply Archie configuration
- Otherwise:
  - proceed unchanged

This ensures:
- no accidental global behavior
- deterministic activation
- easy future extension

---

## Memory Recall Flow

For Archie requests:

1. Extract the latest user message
2. Call the memory service `/recall`
3. Receive a list of relevant memories
4. Inject memory into the system prompt
5. Continue normal LLM execution

Other agents bypass this flow entirely.

---

## Prompt Injection Strategy

Memory is injected into the **system message**, not as user content.

Injection is:
- additive
- clearly labeled
- non‑destructive

The model is instructed:
- to use memory when relevant
- not to parrot memory verbatim

This keeps memory latent and authoritative.

---

## Failure Behavior

If memory recall fails:

- no exception is raised
- the chat proceeds normally
- the user experience degrades gracefully

Memory is an enhancement, not a dependency.

---

## Upgrade Considerations

This integration is resilient to Open WebUI upgrades because:

- it modifies a single backend function
- it avoids frontend changes
- it relies on stable invariants:
  - model selection
  - message construction
  - provider call

After upgrades, validation consists of:
- locating `generate_chat_completion()`
- re‑applying the hook if needed

---

## Extending the Integration

This pattern supports:

- additional agents
- different memory policies per agent
- tool routing per agent
- file output per agent

All within a single Open WebUI instance.

---

## Design Principle

> **Open WebUI decides *when* memory matters.  
> The memory service decides *what* is remembered.**

Keeping this boundary explicit preserves clarity and control.
