---
name: elevenlabs-api-key
description: Create and configure an ElevenLabs API key for the MCP server, SDKs, and REST calls. Use when ElevenLabs tools fail with auth errors or the user needs to set up access.
---

# ElevenLabs API key setup

## Get a key

1. Create an account at https://elevenlabs.io. The free tier includes 10k
   credits per month, enough to test every feature; every tool call spends
   credits.
2. In the dashboard, open your profile menu and create an API key. Prefer a
   scoped key (limit it to the products you actually use) over a full-access
   key.
3. Copy it once and store it in a secrets manager or shell profile; the
   dashboard will not show it again.

## Wire it up

The official MCP server and SDKs all read `ELEVENLABS_API_KEY` from the
environment:

```bash
export ELEVENLABS_API_KEY="..."
```

- MCP server: this plugin launches `uvx elevenlabs-mcp`, which requires
  `ELEVENLABS_API_KEY` in the environment Caeros runs in. Requires `uv`
  installed (`curl -LsSf https://astral.sh/uv/install.sh | sh`).
- Direct REST: send the key as the `xi-api-key` header on
  `https://api.elevenlabs.io` requests.
- SDKs: `ElevenLabs()` (Python) and the JS client pick up the env var
  automatically; pass `api_key=` only in tests.

Optional MCP environment knobs:

- `ELEVENLABS_MCP_BASE_PATH`: base directory for relative file paths
  (default `~/Desktop`).
- `ELEVENLABS_MCP_OUTPUT_MODE`: `files` (default, saves audio to disk),
  `resources` (base64 MCP resources), or `both`.
- `ELEVENLABS_API_RESIDENCY`: data residency region, enterprise only,
  default `us`.

## Troubleshooting

- 401 from the API: wrong or revoked key, or the key's scope excludes the
  endpoint. Regenerate with the right scope.
- MCP server fails to start with a spawn error for `uvx`: `uv` is not on
  PATH for GUI-launched processes; use the absolute path from `which uvx`
  in the server config.
- Quota errors mid-task: credits exhausted for the month; upgrade the plan
  or wait for reset. Check usage in the dashboard before big batch jobs.

## Hygiene

Never commit the key, never paste it into chat or logs, rotate it if it
leaks, and keep separate keys per environment (dev vs production) so a
revocation does not take everything down.
