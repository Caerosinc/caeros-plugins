---
name: firecrawl-scrape
description: Use the Firecrawl v2 API to scrape pages into markdown, map sites, crawl multi-page, search the web, and extract schema-shaped JSON. Use when the user needs page content, site inventories, or structured data from websites.
---

# Firecrawl v2 API

Base URL `https://api.firecrawl.dev/v2`, auth header `Authorization: Bearer fc-...`
(see the `firecrawl-api-key` skill). Through this plugin's MCP server the same
capabilities appear as tools: `firecrawl_scrape`, `firecrawl_map`,
`firecrawl_search`, `firecrawl_crawl` (+ `firecrawl_check_crawl_status`),
`firecrawl_extract`, `firecrawl_parse`, `firecrawl_agent`, `firecrawl_interact`.

## Scrape one page: POST /v2/scrape

```json
{
  "url": "https://example.com/pricing",
  "formats": ["markdown"],
  "onlyMainContent": true,
  "maxAge": 172800000
}
```

- `formats`: `markdown` (default), `html`, `rawHtml`, `links`, `images`,
  `summary`, `screenshot`, `json`, `changeTracking`, plus specialized
  `product`, `branding`, `menu`.
- `onlyMainContent` (default true) strips navs/footers; `onlyCleanContent`
  adds LLM boilerplate removal.
- `maxAge` (ms, default 2 days): serve a cached copy if fresh enough. Huge
  speedup and cost saver; set `0` to force a live fetch.
- `actions`: browser steps before capture: `click`, `write`, `scroll`,
  `wait`, `screenshot`, `executeJavascript`, `pdf`. Needed for JS-gated
  content.
- Response: `{"success": true, "data": {"markdown": ..., "metadata":
  {"title", "sourceURL", "statusCode", ...}}}`. Check `data.metadata.statusCode`
  and the top-level `warning`.

## Structured extraction

Two routes:

1. **`json` format on /scrape** (single page, synchronous): add
   `{"type": "json", "schema": {...}, "prompt": "..."}` to `formats`.
   Result lands in `data.json`, guaranteed to match the schema.
2. **POST /v2/extract** (multi-page, async): body takes `urls` (glob
   patterns like `https://docs.site.com/*` allowed), `prompt`, `schema`,
   optional `enableWebSearch` to fill gaps from the wider web. Returns a job
   `id`; poll `GET /v2/extract/{id}` until `status` is `completed`.
   `ignoreInvalidURLs: true` (default) keeps the job alive when some URLs 404.

Always pass a JSON Schema when the output feeds code; prompt-only extraction
drifts.

## Map a site: POST /v2/map

`{"url": "https://example.com", "search": "blog", "limit": 5000}` returns
`links: [{url, title, description}]` fast, without scraping. `search` ranks
matching URLs first; `sitemap`: `include` (default) | `only` | `skip`;
`includeSubdomains` default true. Map first, then scrape the URLs you
actually need.

## Crawl multi-page: POST /v2/crawl

Async job: returns `{"success": true, "id": ...}`; poll
`GET /v2/crawl/{id}` (status, `completed` page count, page data).

```json
{
  "url": "https://docs.example.com",
  "limit": 100,
  "maxDiscoveryDepth": 2,
  "includePaths": ["^/docs/.*"],
  "scrapeOptions": {"formats": ["markdown"], "onlyMainContent": true}
}
```

- `includePaths` / `excludePaths` are regex against pathnames.
- `crawlEntireDomain: true` follows sibling/parent links (default only
  crawls deeper); `allowSubdomains`, `allowExternalLinks` widen scope.
- `prompt`: natural language that Firecrawl compiles into crawler options.
- MCP flow: `firecrawl_crawl` starts the job, `firecrawl_check_crawl_status`
  polls it.

## Search the web: POST /v2/search

`{"query": "...", "limit": 10, "sources": ["web"]}` with optional `sources`
`news` / `images`, `tbs` time filters (`qdr:d`, `qdr:w`), `location`,
`categories` (`github`, `research`, `pdf`), `includeDomains` /
`excludeDomains`. Add `scrapeOptions` to get full markdown for each hit in
one call instead of search-then-scrape loops. Response groups results per
source under `data.web`, `data.news`, `data.images` and reports
`creditsUsed`.

## Gotchas

- v2 is current; v1 paths (`/v1/scrape` etc.) still exist but new fields
  (summary format, `maxAge`, `prompt` on crawl) are v2. Do not mix versions.
- 402 means out of credits, 429 means rate limited: back off, do not retry
  hot.
- Screenshots and `actions` cost more time; keep `timeout` (default 60000
  ms, max 300000) realistic for heavy pages.
- For local files (PDF, DOCX, XLSX, HTML) use the MCP `firecrawl_parse`
  tool rather than the scrape endpoint.
