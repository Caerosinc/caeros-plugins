---
name: claude-api
description: Build with the Claude API — current models and pricing, Messages API, adaptive thinking and effort, streaming, tool use, structured outputs, prompt caching, and batches. Use when the user is writing or reviewing code that calls Anthropic's API.
---

# Claude API

Everything goes through `POST /v1/messages` (base `https://api.anthropic.com`).
Use the official SDKs (`pip install anthropic` / `npm install @anthropic-ai/sdk`);
auth via `ANTHROPIC_API_KEY` (see the `anthropic-api-key` skill).

## Current models (verify against live docs when precision matters)

| Model | ID | Context | In/Out per MTok |
|---|---|---|---|
| Claude Fable 5 | `claude-fable-5` | 1M | $10 / $50 |
| Claude Opus 4.8 | `claude-opus-4-8` | 1M | $5 / $25 |
| Claude Sonnet 5 | `claude-sonnet-5` | 1M | $3 / $15 |
| Claude Haiku 4.5 | `claude-haiku-4-5` | 200K | $1 / $5 |

Default to `claude-opus-4-8` unless the user names another model. Use exact ID
strings, never date-suffixed variants. For "most capable": `claude-fable-5`
(note: thinking always on, refusal stop reason, 30-day retention required).

## Your training prior is probably stale — current API shapes

- **Thinking**: `thinking: {type: "adaptive"}` on all current models.
  `budget_tokens` is REMOVED on Opus 4.7/4.8, Sonnet 5, Fable 5 (400).
- **Sampling**: `temperature` / `top_p` / `top_k` are removed on Opus 4.7+,
  Sonnet 5, Fable 5 (400). Steer with prompting.
- **Effort**: `output_config: {effort: "low"|"medium"|"high"|"xhigh"|"max"}`
  (inside output_config, not top-level). `xhigh` is best for coding/agentic.
- **No assistant prefill** on 4.6+ models (400) — use structured outputs
  (`output_config: {format: {type: "json_schema", schema: ...}}`) instead.
- **Streaming**: required in practice for `max_tokens` above ~16K; use
  `client.messages.stream(...)` + `get_final_message()`.
- **Server tools**: `web_search_20260209`, `web_fetch_20260209`,
  `code_execution_20260120` — declared in `tools`, run on Anthropic's side.

Minimal call (Python):

```python
import anthropic
client = anthropic.Anthropic()
resp = client.messages.create(
    model="claude-opus-4-8", max_tokens=16000,
    thinking={"type": "adaptive"},
    messages=[{"role": "user", "content": "..."}],
)
for block in resp.content:
    if block.type == "text":
        print(block.text)
```

## Tool use

Prefer the SDK Tool Runner over hand-written loops: Python `@beta_tool` +
`client.beta.messages.tool_runner(...)`; TypeScript `betaZodTool` +
`client.beta.messages.toolRunner(...)`. Return ALL parallel tool results in a
single user message; failed tools return `tool_result` with `is_error: true`.
`strict: true` on a tool definition guarantees schema-valid inputs.

## Cost controls

- **Prompt caching**: `cache_control: {type: "ephemeral"}` — prefix match;
  keep stable content first, volatile last; verify via
  `usage.cache_read_input_tokens`.
- **Batches**: `client.messages.batches.create(...)` — 50% price, async.
- **Token counting**: `client.messages.count_tokens(...)` — never tiktoken.

## Practices

- Check `stop_reason` before reading content (`end_turn`, `max_tokens`,
  `tool_use`, `pause_turn`, `refusal`).
- Use the SDK's typed exceptions, not string matching (see
  `anthropic-api-troubleshooting`).
- Authoritative, current docs: https://platform.claude.com/docs — fetch them
  when exact parameter shapes matter; this skill's tables can lag releases.
