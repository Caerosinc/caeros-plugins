---
name: linkedin
description: Read the user's LinkedIn profile, share posts, and manage profile sections through the LinkedIn plugin's MCP server. Use when the user wants to post to LinkedIn, check their LinkedIn profile, or work with their LinkedIn presence.
---

# LinkedIn

This skill drives the LinkedIn plugin's local MCP server
(`@pegasusheavy/linkedin-mcp`), signed in with the user's own LinkedIn account
via OAuth.

## Step 0: Connect the LinkedIn account (if tools are unavailable)

If no LinkedIn MCP tools appear or tools fail with an authentication error:

1. Open Settings, then MCP. LinkedIn is listed under "Connect an account".
2. Click Connect for LinkedIn and finish the OAuth sign-in in the browser.
3. Retry once the server shows Connected.

## What works, and what is gated by LinkedIn

Discover the exact tool names from the connected server's tool list; do not
guess slugs. The server exposes tools in these families:

- **Profile**: fetch the signed-in member's profile (name, headline, picture,
  email). Works with any standard LinkedIn app.
- **Posting**: share a post to the member's feed. Works with any standard
  LinkedIn app (`w_member_social` scope).
- **Connections, people search, reading past posts**: these call
  partner-gated LinkedIn APIs. They work only when the connected LinkedIn
  developer app has the corresponding LinkedIn products approved.
- **Profile editing** (skills, positions, education, certifications,
  publications, languages): also partner-gated.

**When a tool returns a 403 or "access denied": do not retry and do not
loop.** That tool needs LinkedIn partner API access the connected app does
not have. Say so plainly, and offer what does work (profile fetch, posting)
or a manual path instead.

## Working practices

- **Posting is public and permanent-ish.** Always show the exact final post
  text and get explicit confirmation before calling the share tool, unless
  the user already approved that exact text. Never invent hashtags, mentions,
  or claims the user did not ask for.
- **One post per confirmation.** Never batch-post or re-post on failure
  without checking with the user; a timeout may still have published.
- **Profile data is personal data.** Do not include it in anything shared or
  published without explicit confirmation.
- **Drafting beats posting.** When intent is ambiguous ("write a LinkedIn
  post about X"), deliver a draft first and post only on approval.

## Specialized skills

- `linkedin-posting` — drafting and sharing posts: voice, structure, the
  confirm-before-share workflow, and post-failure handling.
- `linkedin-profile` — reading the profile and (where the connected app has
  partner access) editing profile sections.
- `linkedin-network` — connections and people search (partner-gated), and
  syncing LinkedIn contacts into the CRM.
