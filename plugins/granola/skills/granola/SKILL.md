---
name: granola
description: Core guide for the Granola meeting-notes MCP connection — auth/connect flow, the six official tools, free-vs-paid gating, and which tool to reach for first. Use whenever the user asks about their meetings, meeting notes, transcripts, action items, decisions, or anything Granola, and start here before the specialized granola-* skills.
---

# Granola

This skill drives Granola's official MCP server at `https://mcp.granola.ai/mcp`
with the user's own Granola account. Auth is browser OAuth; the token lives in
the `GRANOLA_MCP_TOKEN` secret and refreshes automatically. The connection is
read-only: nothing here can modify or delete notes.

## Step 0: Connect (if Granola tools are unavailable)

If no Granola tools appear, pause and ask the user to connect:

1. Open Settings → MCP. Find the Granola tile under "Connect an account".
2. Click Connect and finish signing in from the browser window.
3. Retry once the tile shows Connected.

Do not improvise through other integrations (Composio, web search) when the
native server is merely disconnected — ask to connect instead.

## The six tools

Discover exact names from the connected server's tool list — do not guess
beyond these:

- `query_granola_meetings` — "chat with Granola": ask a natural-language
  question across meeting content and get a synthesized answer. Best first
  call for open-ended questions ("what did we decide about pricing?").
- `list_meetings` — list meeting notes by date, title, or attendees;
  filterable by folder on paid plans. Best first call for enumeration
  ("my last meeting", "this week's meetings").
- `get_meetings` — read specific meetings: id, title, date, attendees,
  private notes, and summarized notes (action items live in the notes).
- `get_meeting_transcript` — raw transcript for one meeting id. Paid plans
  only.
- `list_meeting_folders` — folders with id, title, description, note count.
  Paid plans only.
- `get_account_info` — email + active workspace of the connected account.
  Use to disambiguate when results look like the wrong workspace.

## Picking the first call

- "Last meeting" / "meetings this week" / "meetings with X" →
  `list_meetings`, then `get_meetings` on the ids you need. See
  [granola-meetings].
- Open-ended question or cross-meeting insight → `query_granola_meetings`.
- Exact quotes or word-for-word review → `get_meeting_transcript` via
  [granola-transcripts].
- Wrong-looking results → `get_account_info` (MCP follows the active
  workspace selected in the Granola app; switching workspaces there changes
  what you see here).

## Plan gating — degrade gracefully

- Free plan: personal notes from the last 30 days only; transcripts and
  folders are unavailable.
- Business: personal + public workspace notes. Enterprise: admin-scoped.
- If a transcript or folder call is rejected, say the feature needs a paid
  Granola plan and fall back to `get_meetings` summarized notes — do not
  retry the gated tool.
- Rate limit is roughly 100 requests/minute; batch reads with one
  `get_meetings` call over several ids instead of looping.
