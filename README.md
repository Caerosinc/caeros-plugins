# Caeros Plugins

The curated plugin marketplace for [Caeros](https://caeros.com). This repository
hosts the official, first-party plugin collection: 41 plugins that add skills,
slash commands, MCP server connections, apps, and assets to your Caeros client.

- [Quick start](#quick-start)
- [Plugin catalog](#plugin-catalog)
- [Anatomy of a plugin](#anatomy-of-a-plugin)
- [Authoring your own plugin](#authoring-your-own-plugin)
- [Contributing](#contributing)
- [Distribution and updates](#distribution-and-updates)

## Quick start

Install plugins from the in-app **Plugins store** (hosted registry), or add this
repository as a git marketplace directly:

```sh
# Register this repo as a marketplace named "caeros-official"
cae plugin add caeros-official https://github.com/Caerosinc/caeros-plugins.git

# Install any plugin from the catalog below
cae plugin install caeros-official <name>

# Later: pull new versions of everything you installed
cae plugin update
```

Updates are delivered as a shallow git re-fetch with content-digest comparison —
the Update button (or `cae plugin update`) only touches plugins whose contents
actually changed.

## Plugin catalog

Every plugin lives in its own directory under [`plugins/`](plugins/) with a
`.caeros-plugin/plugin.json` manifest. Capabilities are abbreviated as
**S** = skills, **M** = bundled MCP server(s), **C** = slash commands,
**A** = apps/connectors.

### AI & Media

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`elevenlabs`](plugins/elevenlabs) | ElevenLabs | Text to speech, transcription, voice agents, dubbing, and sound effects. | S, M |
| [`higgsfield`](plugins/higgsfield) | Higgsfield | Image and video generation via Higgsfield's hosted MCP server and Cloud API. | S, M |
| [`replicate`](plugins/replicate) | Replicate | Discover and run thousands of open AI models; deploy your own with Cog. | S, M |

### Auth

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`auth0`](plugins/auth0) | Auth0 | App quickstarts, Actions, custom claims, and tenant management via the official Auth0 MCP server. | S, M |
| [`clerk`](plugins/clerk) | Clerk | Next.js App Router integration, backend SDK, JWT verification, and webhooks. | S, M |
| [`workos`](plugins/workos) | WorkOS | Enterprise auth: AuthKit, SSO, Directory Sync, RBAC, and Audit Logs. | S, M |

### Automations

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`automations`](plugins/automations) | Caeros Automations | Prebuilt slash-command automations: `/fix-ci`, `/daily-digest`, `/vuln-scan`, `/triage-issues`, `/gen-docs`, `/add-tests`, `/autofix-review-comments`, `/flag-cleanup`. | S, C |

### Databases

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`firebase`](plugins/firebase) | Firebase | Firestore, Auth, Hosting, and Functions through the official Firebase MCP server. | S, M |
| [`mongodb`](plugins/mongodb) | MongoDB | Query, model, and manage MongoDB and Atlas via the official MongoDB MCP server. | S, M |
| [`prisma`](plugins/prisma) | Prisma | Model, migrate, and query databases with Prisma ORM and Prisma Postgres. | S, M |
| [`redis`](plugins/redis) | Redis | Redis data structures, caching, and search via the official Redis MCP server. | S, M |

### Developer Tools

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`anthropic`](plugins/anthropic) | Anthropic Developers | Build AI apps and agents on the Claude API with Anthropic best practices. | S, A |
| [`plugin-dev`](plugins/plugin-dev) | Caeros Plugin Dev | Toolkit for building Caeros plugins: scaffolding (`/new-plugin`), manifest schema reference, and publishing guides. | S, C |
| [`xai`](plugins/xai) | xAI Developers | Build with xAI's Grok API — models, server-side search, and best practices. | S |

### Finance

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`investment-banking`](plugins/investment-banking) | Investment Banking | M&A, capital markets, and LevFin workflows on Caeros' private-markets vault. | S, A |
| [`public-equity-investing`](plugins/public-equity-investing) | Public Equity Investing | Public equity PM research: tearsheets, earnings reviews, and idea screening. | S, A |

### Frontend & Design

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`gsap`](plugins/gsap) | GSAP | Web animation: tweens, timelines, ScrollTrigger, and React integration. | S |
| [`shadcn`](plugins/shadcn) | shadcn/ui | CLI workflows, theming, variants, and registries, plus the official shadcn MCP server. | S, M |
| [`svelte`](plugins/svelte) | Svelte | Svelte 5 and SvelteKit: runes, snippets, routing, form actions, and testing. | S, M |
| [`tldraw`](plugins/tldraw) | tldraw | Infinite canvas apps with the tldraw SDK: embedding, persistence, custom shapes, multiplayer sync. | S |

### Infrastructure

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`aws`](plugins/aws) | AWS | CDK v2 and CloudFormation, serverless on Lambda, databases, ECS/Fargate containers, plus AWS's hosted Knowledge MCP server. | S, M |
| [`gcp`](plugins/gcp) | Google Cloud | gcloud, Cloud Run, GKE, BigQuery, and IAM. | S |

### LLM Ops

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`braintrust`](plugins/braintrust) | Braintrust | Evals, scorers, datasets, and production tracing via Braintrust's hosted MCP server. | S, M |
| [`langfuse`](plugins/langfuse) | Langfuse | Instrument, evaluate, and manage LLM applications with the open-source Langfuse platform. | S, M |
| [`mem0`](plugins/mem0) | Mem0 | Persistent memory for AI apps: add, search, and manage memories across users, agents, and sessions. | S, M |

### Observability

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`grafana`](plugins/grafana) | Grafana | Dashboards, metrics and log queries, and alert management. | S, M |
| [`pagerduty`](plugins/pagerduty) | PagerDuty | Incident triage, services and on-call management, and workflow automation. | S, M |

### Payments

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`chargebee`](plugins/chargebee) | Chargebee | Subscription billing: customers, subscriptions, invoices, hosted pages, and webhook idempotency. | S |
| [`ramp`](plugins/ramp) | Ramp | Corporate spend platform: developer API, spend analysis, and the official Ramp MCP server. | S, M |
| [`revenuecat`](plugins/revenuecat) | RevenueCat | In-app subscriptions: SDKs, webhooks, REST API, and the official RevenueCat MCP server. | S, M |

### Productivity

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`fibery`](plugins/fibery) | Fibery | Query, build, and automate a Fibery workspace through Fibery's official MCP server. | S, M |
| [`gmail`](plugins/gmail) | Gmail | Read, search, draft, and send Gmail through your connected Google account. | S, A |
| [`spreadsheets`](plugins/spreadsheets) | Spreadsheets | Create, edit, analyze, verify, and export XLSX/CSV with formulas, formatting, and charts. | S |

### Sales

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`plusvibe`](plugins/plusvibe) | PlusVibe | Cold email outreach: campaigns, sequences, lead lists, reply triage, and inbox placement. | S, A |

### Security

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`onepassword`](plugins/onepassword) | 1Password Developer | Keep secrets out of code: `op` CLI, secret references, and Environments. | S, M |
| [`semgrep`](plugins/semgrep) | Semgrep | Static analysis: scan code for vulnerabilities and write custom rules. | S, M |
| [`snyk`](plugins/snyk) | Snyk | Find and fix vulnerabilities in code, dependencies, containers, and IaC. | S, M |

### Social

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`linkedin`](plugins/linkedin) | LinkedIn | Read your profile, share posts, and manage profile sections through the LinkedIn API. | S, M |
| [`x`](plugins/x) | X | Search, read, and organize X through X's official MCP server. | S, M |

### Web Data

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`exa`](plugins/exa) | Exa | Neural, meaning-based web search with clean content, highlights, and research modes. | S, M |
| [`firecrawl`](plugins/firecrawl) | Firecrawl | Scrape, crawl, map, and extract structured data from any website. | S, M |
| [`tavily`](plugins/tavily) | Tavily | Real-time web search, extraction, and crawling built for AI agents. | S, M |

### Examples

| Plugin | Name | What it does | Capabilities |
| --- | --- | --- | --- |
| [`hello-caeros`](plugins/hello-caeros) | Hello Caeros | A minimal reference plugin demonstrating a bundled skill. | S |

## Anatomy of a plugin

A plugin is one directory under `plugins/` containing a manifest plus any number
of capability folders:

```
plugins/<name>/
├── .caeros-plugin/
│   └── plugin.json      # the manifest (canonical location)
├── skills/              # SKILL.md packs the agent loads on demand
├── commands/            # slash-command prompt templates
├── apps/                # .app.json app connectors (+ optional instructions)
├── assets/              # logo.svg, screenshots, other static files
└── .mcp.json            # optional bundled MCP server definitions
```

### The manifest (`plugin.json`)

The loader reads `.caeros-plugin/plugin.json`, resolves every declared path
safely under the plugin root, then fills unset capabilities from layout
conventions. A complete manifest looks like:

```json
{
  "name": "my-plugin",
  "version": "0.1.0",
  "description": "One sentence about what the plugin does.",
  "keywords": ["search", "terms"],
  "capabilities": {
    "skills": "./skills",
    "mcpServers": "./.mcp.json",
    "apps": "./apps",
    "commands": "./commands",
    "assets": "./assets"
  },
  "permissions": {
    "network": ["api.example.com"],
    "exec": false
  },
  "runtime": "local",
  "interface": {
    "displayName": "My Plugin",
    "shortDescription": "Storefront one-liner.",
    "longDescription": "Full storefront description.",
    "developerName": "Your Name",
    "category": "Developer Tools",
    "capabilities": ["Read", "Write", "Interactive"],
    "websiteUrl": "https://example.com",
    "privacyPolicyUrl": "https://example.com/privacy",
    "termsOfServiceUrl": "https://example.com/terms",
    "defaultPrompt": ["try my plugin on this repo"],
    "brandColor": "#1b365d",
    "logo": "./assets/logo.svg",
    "composerIcon": "./assets/logo.svg"
  }
}
```

Key rules:

- **capabilities** — each entry is a `./`-relative path to a directory or file
  for that surface. Supported surfaces: `skills`, `mcpServers`, `apps`,
  `agents`, `workflows`, `tools`, `commands`, `views`, `hooks`, `assets`.
  Paths that escape the plugin root (`..`, absolute paths) are silently
  rejected.
- **permissions** — scopes the user grants at install and the runtime enforces:
  `fs` (path scopes), `network` (allowed hosts), `secrets` (readable secret
  keys), `tools` (tool-name globs), `exec` (shell access). Request the
  narrowest set that makes the plugin work.
- **runtime** — `"local"` (default) or `"cloud"` (executed server-side via the
  Caeros gateway).
- **interface** — the storefront listing. The hosted registry renders
  `displayName`, descriptions, `category`, `logo`, `screenshots`, and
  `defaultPrompt` directly, so complete it carefully before submitting.
- **Convention defaults** — unset capabilities fall back to `skills/`,
  `commands/`, `agents/`, `.mcp.json`, and `hooks/hooks.json` when they exist.
  Declare paths explicitly anyway: it documents intent and survives layout
  changes.

### Skills

Each skill is `skills/<skill-name>/SKILL.md` with YAML frontmatter:

```markdown
---
name: my-skill
description: What the skill covers and when the agent should load it. The
  "use when" part is the trigger — make it concrete.
---

The body is reference material the agent reads when the skill triggers.
```

### Slash commands

Each command is a markdown file in `commands/`:

```markdown
---
name: my-command
description: Shown in the command palette.
aliases: mc, my-cmd
---

The body is the prompt template the agent runs when the user types /my-command.
```

Command names are sanitized to lowercase `a-z`, `0-9`, `-`, `_`; built-in
Caeros commands always win on collision, and empty bodies are rejected.

### Bundled MCP servers (`.mcp.json`)

```json
{
  "mcpServers": {
    "my-api": { "transport": "url", "url": "https://mcp.example.com/mcp" },
    "my-local-tool": { "transport": "stdio", "command": "npx", "args": ["-y", "my-mcp-server"] }
  }
}
```

A `url` transport requires the host in `permissions.network`; a `stdio`
transport spawns a process and requires `"exec": true`.

The authoritative reference for all of the above is the
[`plugin-dev`](plugins/plugin-dev) plugin's `create-caeros-plugin` skill.

## Authoring your own plugin

1. **Scaffold** — install [`plugin-dev`](plugins/plugin-dev) and run
   `/new-plugin`, or copy [`hello-caeros`](plugins/hello-caeros) as a starting
   point.
2. **Write the manifest** — complete every `interface` field; it is your
   storefront listing.
3. **Validate**:

   ```sh
   cae plugin validate <dir>
   python3 -m json.tool plugins/<name>/.caeros-plugin/plugin.json   # parses cleanly
   ```

4. **Test locally**:

   ```sh
   cae plugin install-local <dir>
   ```

5. **Pre-ship checklist**:
   - Every declared capability path exists and stays inside the plugin root.
   - Each `SKILL.md` has `name` and `description` frontmatter.
   - Each command file has a non-empty body.
   - Permissions are the minimum the plugin needs.
   - The logo is original artwork (no third-party trademarks).

## Contributing

Contributions are welcome — both new plugins and improvements to existing ones.

1. One plugin per directory under `plugins/`, named in lowercase kebab-case.
2. Follow the authoring checklist above; CI and reviewers will run
   `cae plugin validate` on your plugin directory.
3. Review criteria for first-party submissions: manifest validity, minimal
   permissions, an original logo, accurate storefront copy, and skills/commands
   that follow the conventions in the `create-caeros-plugin` skill.
4. Open a PR against `main` with a clear description of what the plugin does
   and how you tested it (`install-local` output helps).

## Distribution and updates

- **Git marketplace** (this repo): clients add the repo URL once, then install
  and update plugins from it. Updates are shallow re-fetches compared by
  content digest, so unchanged plugins are never re-downloaded.
- **Hosted registry**: powers the in-app Plugins store. It indexes published
  plugins for search and renders the storefront from each manifest's
  `interface` block.
- **Versioning**: the manifest `version` field is what users see; tag releases
  if you want users pinned to stable snapshots.
- **Compatibility**: Caeros also loads plugin bundles authored for other
  harnesses (`.codex-plugin/`, `.cursor-plugin/`, `.claude-plugin/` manifests),
  so skills and MCP configs written elsewhere generally work unchanged.
