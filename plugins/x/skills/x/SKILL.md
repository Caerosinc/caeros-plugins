---
name: x
description: Search, read, and monitor X (Twitter) through X's official MCP server — post search, timelines, threads, trends, news, and Articles. Use when the user wants to know what X is saying about anything, follow accounts or threads, or scan trends.
---

# X

This skill drives X's official MCP server (`https://api.x.com/mcp`), connected
with the user's own X account. It assumes the X plugin's MCP server is
connected so the X tools are available.

## Step 0: Connect the X account (if tools are unavailable)

If no X MCP tools appear, pause and ask the user to connect X:

1. Open Settings → MCP. The plugin's X server is listed there after install.
2. Click Connect for X and finish the OAuth sign-in in the browser.
3. Retry once the server shows Connected.

After connecting, continue with the request.

## What the X MCP can do

Discover the exact tool names from the connected server's tool list — do not
guess slugs. The server provides tools in these families:

- **Search**: full-archive post search, user search, and news search.
  Supports X search query syntax (`from:user`, `to:user`, `"exact phrase"`,
  `min_faves:`, `lang:`, `since:`/`until:`, `-filter:retweets`, ...).
- **Posts**: fetch specific posts by id (text, author, metrics, media refs).
- **Timelines**: a user's recent posts and replies.
- **Trends**: current trending topics.
- **Articles**: long-form X Articles.
- **Bookmarks**: list, add, remove, and organize into folders (see the
  `x-bookmarks` skill for workflows).

**Read-and-bookmark by design.** The connection deliberately excludes posting
permissions: you cannot post, reply, like, repost, or DM. If asked to post,
say the X connection is read-only and offer to draft the post text instead.

## Working practices

- **Recent first, archive second.** Start with a tight window (last 1-7 days)
  and only widen to the full archive when the user asks for history. Archive
  queries are the expensive path.
- **Cite every claim.** Link posts as `https://x.com/i/status/<post_id>`.
  Never invent engagement numbers; report metrics only when a tool returned
  them.
- **Dedupe.** Collapse retweets/quotes of the same origin post; prefer
  `-filter:retweets` in search queries when hunting for original content.
- **Threads.** To unroll a thread: fetch the head post, then search
  `conversation_id:<id> from:<author>` and order by time.
- **Paginate deliberately.** Fetch one page, check whether it answers the
  question, and only then continue. Say when results were truncated.

## Workflows

### Topic brief ("what is X saying about ...")
1. Search recent posts for the topic (7-day window, `-filter:retweets`,
   `min_faves:` threshold to cut noise).
2. Cluster into themes; pull the 3-5 highest-signal posts per theme.
3. Deliver: themes with 1-2 sentence summaries, linked example posts, and
   notable voices. Note where sentiment splits.

### Account digest ("catch me up on @user")
1. Fetch the user's timeline (last 20-50 posts).
2. Split original posts from replies; summarize main threads with links.

### News scan
1. Run news search for the topic plus trends for context.
2. Cross-check headline claims against the underlying posts before repeating
   them.

### Thread unroll
1. Fetch the head post by id or URL.
2. Collect the author's replies in the conversation, order chronologically,
   and summarize with the full-thread link.

### Save for later
Bookmark high-value finds (ask first unless the user already told you to),
using folders for the topic — details in `x-bookmarks`.
