---
name: tavily-search
description: Use the Tavily API - search with depth and topic controls, extract clean content from known URLs, and crawl sites with natural language instructions. Use when the user needs live web results, page content, or site traversal for agents.
---

# Tavily API

Base `https://api.tavily.com`, auth header
`Authorization: Bearer tvly-...` (see the `tavily-api-key` skill). Through
this plugin's MCP server (OAuth sign-in) the core capabilities appear as
the Tavily search and extract tools.

## Search: POST /search

```json
{
  "query": "EU AI act enforcement timeline",
  "search_depth": "basic",
  "topic": "general",
  "max_results": 5
}
```

Key fields:

- `search_depth`: `ultra-fast` (lowest latency) | `fast` | `basic`
  (default, 1 credit) | `advanced` (2 credits, best relevance, enables
  content chunking).
- `topic`: `general` (default) | `news` | `finance`. `news` unlocks
  publish-date-aware results; `country` boost works on `general` only.
- Freshness: `time_range` (`day`/`week`/`month`/`year`) or explicit
  `start_date`/`end_date` (YYYY-MM-DD).
- Scope: `include_domains` (max 300) / `exclude_domains` (max 150),
  `exact_match` for quoted phrases.
- Content: `include_answer` (`true`/"basic"/"advanced") for an LLM answer,
  `include_raw_content` (`true`/"markdown"/"text") for full page content,
  `chunks_per_source` (1 to 3, advanced only), `include_images` and
  `include_image_descriptions`.
- `auto_parameters: true` lets Tavily pick depth/topic from the query.
- `max_results`: 0 to 20, default 5.

Response:

```json
{
  "query": "...",
  "answer": "... (if requested)",
  "results": [
    {"title": "...", "url": "https://...", "content": "snippet",
     "score": 0.95, "favicon": "...", "raw_content": "... (if requested)"}
  ],
  "response_time": 1.2,
  "usage": {"credits": 1},
  "request_id": "..."
}
```

`score` is the relevance signal; sort and cut on it.

## Extract: POST /extract

For URLs you already have: `urls` (string or array, up to 20),
`extract_depth` `basic` (default) | `advanced`, `format` `markdown`
(default) | `text`, optional `query` + `chunks_per_source` (1 to 5) to
rerank and return only the relevant chunks, `include_images`, `timeout`
(1 to 60 s). Response: `results` with `url` + `raw_content` (chunks joined
by `[...]` when `query` is set) and `failed_results` with per-URL errors.
Cost: 1 credit per 5 basic extractions, 2 per 5 advanced.

## Crawl: POST /crawl

Graph-based traversal from a root `url`:

- `instructions`: natural language steering ("only API reference pages");
  raises cost from 1 to 2 credits per 10 pages.
- Bounds: `limit` (default 50), `max_depth` (default 1, max 5),
  `max_breadth` (default 20, max 500).
- Scope: `select_paths` / `exclude_paths` (regex),
  `allow_external` (default true; set false to stay on-domain).
- `extract_depth` controls per-page extraction quality/cost.

Response mirrors extract: `base_url`, `results` (`url`, `raw_content`),
`usage`, `request_id`. Always set `limit` and `allow_external: false`
unless you want the open web.

## Gotchas

- `advanced` depth is the only way to get `chunks_per_source`; `basic` +
  `include_raw_content` returns whole pages instead (token-heavy).
- `include_answer` costs quality-wise: it is a convenience summary, not a
  substitute for reading `results` (see `tavily-agent-patterns`).
- 401: bad key or missing Bearer prefix. 429: rate limit, back off.
- Keep `request_id` from responses when reporting API issues to Tavily.
