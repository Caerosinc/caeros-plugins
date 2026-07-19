---
name: exa-search
description: Use the Exa search API - meaning-based web search with type tiers (instant to deep-reasoning), category and domain filters, and content options (text, highlights, summary). Use when the user needs web search results or clean page content for agents.
---

# Exa Search API

`POST https://api.exa.ai/search`, auth via `x-api-key: <key>` header (or
`Authorization: Bearer <key>`; see the `exa-api-key` skill). Through this
plugin's MCP server the same engine appears as `web_search_exa` and
`web_fetch_exa` (plus `web_search_advanced_exa` and agent tools when
enabled via `?tools=` on the server URL).

## Minimal request

```json
{
  "query": "startups building solid state batteries",
  "type": "auto",
  "numResults": 10,
  "contents": {"text": {"maxCharacters": 2000}, "highlights": true}
}
```

## Search types (latency vs depth)

- `instant`: lowest latency, quick lookups.
- `fast`: reduced latency, still high quality; good default for agent loops.
- `auto`: balanced, the recommended default when unsure.
- `deep-lite`: a few seconds, adds synthesis.
- `deep`: comprehensive multi-source research.
- `deep-reasoning`: heaviest, for complex analytical questions.

Under the hood Exa blends neural (embedding) retrieval with keyword
retrieval per tier. Phrase queries as natural language descriptions of the
answer ("companies working on X"), not keyword soup; that is what the
neural side rewards.

## Filters

- `category`: `company`, `research paper`, `news`, `publication`,
  `personal site`, `financial report`, `people`. Dramatically improves
  precision for entity-shaped queries.
- `includeDomains` / `excludeDomains`: arrays of hostnames.
- `startPublishedDate` / `endPublishedDate`: ISO 8601; use for freshness.
- `numResults`: 1 to 100 (default 10).
- `moderation`: boolean content filter.

## Contents options (fetch page content in the same call)

- `text`: `true` or `{"maxCharacters": ..., "includeHtmlTags": false}`.
  Full cleaned page text.
- `highlights`: relevance-scored snippets; ideal for citations because each
  highlight ties to its source URL.
- `summary`: LLM-written overview per result, optionally steered with a
  custom query.
- `maxAgeHours`: cache freshness control; lower forces fresher content at
  higher latency.
- `subpages`: also crawl N related pages per result.

Requesting contents inline is one round trip; do it instead of
search-then-fetch loops. For content of URLs you already know, use the MCP
`web_fetch_exa` tool.

## Response shape

```json
{
  "results": [
    {
      "title": "...", "url": "https://...",
      "publishedDate": "2026-05-01T00:00:00.000Z", "author": "...",
      "text": "...", "highlights": ["..."], "summary": "..."
    }
  ],
  "costDollars": {"total": 0.01}
}
```

Check `costDollars` when tuning: contents and deep types cost more than
bare result lists.

## Websets (standing research lists)

Separate product at `https://api.exa.ai/websets/v0/...`: define a search in
natural language, Exa finds and verifies candidates against your criteria,
`enrichments` extract typed fields per result, `monitors` re-run on a
schedule, webhooks push updates. Use Websets when the user wants a living
list ("all Series A fintechs in Europe") rather than a one-shot answer.

## Gotchas

- An empty `results` array usually means over-constrained filters; drop
  `category` or widen dates before concluding nothing exists.
- `text: true` on 100 results is a token flood; cap `maxCharacters` or use
  `highlights` when feeding a model.
- Deep types return synthesized output and take much longer; do not put
  them inside tight tool loops (see `exa-research-patterns`).
