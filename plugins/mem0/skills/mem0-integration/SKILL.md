---
name: mem0-integration
description: Integrate the Mem0 memory layer into apps with the Python and JS SDKs, including add and search APIs, user/agent/session scopes, filters, and the retrieve-then-store loop. Use when adding persistent memory to a chatbot or agent.
---

# Mem0 Integration

Mem0 extracts durable facts from conversations and retrieves them by semantic
search. Install `pip install mem0ai` / `npm install mem0ai`; platform auth via
`MEM0_API_KEY` (see `mem0-setup`).

## Client

```python
from mem0 import MemoryClient
client = MemoryClient()          # reads MEM0_API_KEY
```

```javascript
import MemoryClient from "mem0ai";
const client = new MemoryClient({ apiKey: process.env.MEM0_API_KEY });
```

## Add memories

Pass conversation messages; Mem0 extracts and deduplicates facts (it may
store zero, one, or several memories per call):

```python
messages = [
    {"role": "user", "content": "I'm vegetarian and allergic to nuts."},
    {"role": "assistant", "content": "Noted, I'll remember that."},
]
client.add(messages, user_id="user123", metadata={"source": "onboarding"})
```

Adds run asynchronously on the platform; use the returned event id to check
status when you need confirmation.

## Search memories

```python
results = client.search(
    "What are my dietary restrictions?",
    filters={"user_id": "user123"},
)
context = "\n".join(m["memory"] for m in results.get("results", results))
```

Also available: `get_all` (list with pagination and filters), `get` (by
memory id), `update`, `delete`, `delete_all`. Filters support the scope ids
plus `metadata` and categories; check current docs for the exact filter DSL
of the API version you target (v2 search uses structured filters).

## Scopes: who owns a memory

- `user_id`: long-lived facts about a person (preferences, profile). The
  default scope for personalization.
- `agent_id`: facts an agent learns about its own operation (persona,
  procedures). Combine with `user_id` for per-user-per-agent memory.
- `run_id`: session or task scope; short-lived working memory isolated from
  the durable profile.

Choose scopes at `add` time and mirror them in `search` filters. Multi-agent
apps: shared user profile under `user_id` only, agent-private knowledge under
`user_id` + `agent_id`, ephemeral task state under `run_id`.

## The loop

Per turn:
1. `search` with the incoming user message, scoped to the user.
2. Inject top memories into the system prompt ("Known about this user: ...").
3. Generate the reply.
4. `add` the new turn (user + assistant messages) so facts keep accruing.
   Fire-and-forget; do not block the response on it.

## Practices

- Namespace `user_id` values per environment (e.g. `prod:usr_42`) to keep
  test data out of production memory.
- Store provenance in `metadata` so memories can be audited or bulk-deleted.
- Deleting a user: `delete_all` with their `user_id` covers GDPR-style
  erasure.
- The bundled hosted MCP server exposes the same operations as tools
  (`add_memory`, `search_memories`, `get_memories`, `update_memory`,
  `delete_memory`, entity and event tools), useful for agent workflows
  without SDK code.
