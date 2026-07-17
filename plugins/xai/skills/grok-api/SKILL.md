---
name: grok-api
description: Build with xAI's Grok API — current models with real context windows and pricing, the api.x.ai surface, structured outputs, and tool calling. Use when the user is writing or reviewing code that calls xAI.
---

# Grok API

Base URL `https://api.x.ai/v1`. Auth: Bearer key via `XAI_API_KEY` (create at
console.x.ai). Primary surface is the Responses API (`/responses`); xAI also
supports OpenAI-compatible clients — point the OpenAI SDK at the base URL:

```python
from openai import OpenAI
client = OpenAI(base_url="https://api.x.ai/v1", api_key=os.environ["XAI_API_KEY"])
resp = client.chat.completions.create(
    model="grok-4.5",
    messages=[{"role": "user", "content": "..."}],
)
```

## Current models (verified against console.x.ai pricing)

| Model | ID | Context | Notes |
|---|---|---|---|
| Grok 4.5 | `grok-4.5` (alias `grok-4.5-latest`) | 500K | Coding/agentic flagship: text+image input, native parallel tool calling, structured outputs. $2/M in, $0.50/M cached, $6/M out at ≤200K prompt tokens — every rate DOUBLES above 200K. |
| Grok Build | `grok-build-0.1` (alias `grok-code-fast-1`) | 256K | Fast, cheap coding tier: $1/M in, $0.20/M cached, $2/M out. |

- **Reasoning is always on for Grok 4.5** and controlled with
  `reasoning_effort`: `low` / `medium` / `high` (default `high`). It cannot
  be disabled — `"none"` returns a 400.
- `grok-4.3` is retired upstream; migrate to `grok-4.5` (note the price
  change: $2/$6 vs 4.3's $1.25/$2.50).
- Max output tokens: 32768 across the lineup.

## Features

- **Function calling**: OpenAI-style `tools` definitions; Grok 4.5 issues
  parallel calls natively — return all results before continuing.
- **Structured outputs**: JSON-schema constrained responses; supported on
  both models.
- **Vision**: image input on both models.
- **Server-side search**: X and web search run on xAI's side — see the
  `grok-server-tools` skill.

## Practices

- Watch the 200K prompt-token pricing cliff on Grok 4.5: for long-context
  work, measure prompt sizes; crossing 200K doubles every rate.
- Cached input is 4x cheaper — keep stable prefixes stable.
- Exact request schemas for the Responses API and search parameters:
  https://docs.x.ai — fetch current docs when precision matters; parameter
  shapes change faster than this skill.
