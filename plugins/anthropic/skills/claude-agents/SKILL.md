---
name: claude-agents
description: Choose and build the right Anthropic agent architecture — manual loop, SDK Tool Runner, Managed Agents, or the Claude Agent SDK. Use when the user wants to build an agent on Claude or asks which Anthropic agent surface to use.
---

# Building Agents on Claude

Two questions separate the four approaches: who supplies the **harness**
(agent loop + context management) and who supplies the **deployment**.

| # | Approach | You write | Harness / deployment | Use when |
|---|---|---|---|---|
| 1 | Manual loop (`while stop_reason == "tool_use"`) | The loop | Yours / yours | Full control, no beta dependency |
| 2 | **Tool Runner** (`client.beta.messages.tool_runner`) | Just tool functions | SDK / yours | Custom-tool agent without loop code — the default choice |
| 3 | **Managed Agents** (REST, beta `managed-agents-2026-04-01`) | Agent config + tool results | Anthropic / Anthropic (per-session sandbox) | Hosted loop + sandbox, versioned configs, schedules |
| 4 | **Claude Agent SDK** (`claude-agent-sdk`) | A prompt + options | Claude Code harness / yours | Batteries-included coding agent on your infra |

Don't conflate 2 and 4: the Tool Runner loops over tools YOU define; the
Agent SDK ships Claude Code's built-in tools (read/write/bash/grep/web).

## Managed Agents in one screen

Mandatory flow: **Agent (create ONCE, store the id) → Session (every run)**.
`model` / `system` / `tools` live on the agent, never the session.

```python
agent = client.beta.agents.create(
    name="Reviewer", model="claude-opus-4-8",
    tools=[{"type": "agent_toolset_20260401"}],
)
session = client.beta.sessions.create(agent=agent.id, environment_id=env.id)
client.beta.sessions.events.send(session_id=session.id, events=[
    {"type": "user.message", "content": [{"type": "text", "text": "..."}]},
])
```

- Open the event stream BEFORE sending the kickoff (no replay on SSE).
- Break on `session.status_terminated`, or `session.status_idle` whose
  `stop_reason.type` is not `requires_action`.
- MCP auth goes in vaults (`vault_ids`), never on the agent definition.
- Cron shapes use deployments (`schedule` + `initial_events`).
- Recommended control plane: version-controlled YAML applied with the `ant`
  CLI (`ant beta:agents create < agent.yaml`); code owns sessions only.

## Design guidance

- Only build an agent when the task is genuinely open-ended; single calls and
  code-orchestrated workflows cover most cases.
- Start with a bash-style broad tool, promote actions to dedicated tools when
  you need gating, rendering, auditing, or parallel scheduling.
- Long runs: context editing (prune), compaction (summarize, beta), memory
  (cross-session). Cache-friendly: frozen system prompt, deterministic tools.

Current docs: https://platform.claude.com/docs/en/managed-agents/overview and
https://code.claude.com/docs/en/agent-sdk — fetch for exact bindings.
