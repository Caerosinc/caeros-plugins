---
name: fibery
description: Read, search, and navigate a Fibery workspace through Fibery's official MCP server - spaces, databases, entities, documents, views, and links. Use whenever the user mentions Fibery, asks what is in their workspace, or wants to pull, summarize, or act on Fibery data. Routes to the specialist fibery-* skills for querying, writing, schema design, views, reports, and admin.
---

# Fibery

This skill drives Fibery's official MCP server (`https://mcp.fibery.io/mcp`),
connected with the user's own Fibery account. It is the entry point for every
Fibery request: orient here first, then hand off to the specialist skill.

## Step 0: check the connection

If no `fibery` MCP tools are available, the plugin's server is not authorized
yet. Say exactly that, and ask the user to connect the Fibery server for this
plugin.

Then stop and wait. Do not improvise a substitute: do not fall back to web
search, do not guess CLI commands, and do not invent tool names. An
unauthenticated server serves zero tools, and guessing burns the turn.

If the tools are present, call `get_me` once to confirm identity and role
(`role/admin` matters for schema and access work), then continue.

## The mental model

Fibery is not a task tracker with a database bolted on. It is a connected
relational graph that you shape yourself:

- **Workspace** is the whole account, at `https://{account}.fibery.io`.
- **Spaces** are top-level containers, roughly a team or a domain
  (`Product`, `CRM`, `GitHub`).
- **Databases** live inside Spaces and define a type of thing
  (`Product/Feature`, `CRM/Account`). This is Fibery's core distinction from
  ClickUp or Jira, where everything is a Task with properties.
- **Entities** are the records inside a Database: one specific Feature.
- **Fields** define what an Entity holds: text, number, date, select,
  document, file, or a link to another Database.
- **Relations** connect Databases and are what make the workspace a graph.
  A Feature has many Tasks; pulling on a Feature shows everything related.
- **Views** are saved ways to look at data (grid, board, timeline, calendar,
  gantt, map, feed, gallery, form, report, document). The same Database
  usually has several.
- **Automations** (Rules and Buttons), **Formulas**, and **Access templates**
  sit above the data.

## The naming law

Almost every argument is a fully-qualified `Space/Name` string, and this is the
single most common source of errors.

- Databases: `"Product Management/Feature"`, never `"Feature"`.
- Fields: `"Product Management/Priority"`, and the space prefix must match the
  database's space.
- Built-ins keep their own prefixes: `fibery/id`, `fibery/public-id`,
  `fibery/creation-date`, `fibery/modification-date`, `fibery/created-by`,
  `workflow/state`, `assignments/assignees`, `comments/comments`,
  `Collaboration~Documents/Document`.
- Never invent a name. Read it from `schema` or `schema_detailed` first.

## Discovery ladder

Climb it in order. Skipping a rung is what produces "field not found" loops.

1. **`schema`** gives the whole workspace as a tree of spaces and databases,
   with each database's access level. Cheap. Call it first, every session.
2. **`schema_detailed`** takes specific database names and returns YAML fields
   with types and metadata comments (`collection`, `UI title`, `read-only`,
   `formula`, `default`, `dependency`) plus enum `values` with their ids.
   Pass `includeRelatedDatabases: true` to walk relations faster on a small
   schema. You need this before any query filter, any write, and any view.
3. **`query`** or **`search`** to get actual data.

## Two identifiers, two jobs

- `fibery/id` is a UUID. Every write tool takes this.
- `fibery/public-id` is the short human number (`42`) that appears in the URL.
  Users quote this. `get_entity_links` and `search_history` take it.

Select both when you plan to report results back with links.

## Reading data: pick the right door

| Situation | Tool |
|---|---|
| Known database, precise filters, aggregation, pagination | `query` |
| Fuzzy words, unsure where it lives | `search` (BM-25 over titles, docs, comments) |
| User pasted a Fibery URL | `fetch_by_url` |
| User names a view ("the Roadmap board") | `search` with `viewType`, then `fetch_view_data` |
| Need a view's configuration, not its rows | `query_views` |
| "How does Fibery do X?" (product question) | `search_guide` |

`search_guide` reads Fibery's public User Guide and never sees workspace data.
`search` and `query` read workspace data and never see the guide. Choosing
wrong is a common miss: "how do automations work" is `search_guide`, "what
automations do we have" is not answerable by either (automations are not
exposed as MCP tools; say so and point at Space configuration in the UI).

## Documents are a two-step read

Document fields (`Collaboration~Documents/Document`, such as Description) hold
their content behind a secret. You cannot select the field directly.

1. `query` selecting `["Space/Description", "Collaboration~Documents/secret"]`.
2. `get_documents_content` with those secrets. Pass `reducePrompt` to steer
   how long documents get summarized, for example
   `"Extract the customer pain points only"`.

Standalone documents (not attached to an entity) are views: find them with
`search` using `viewType: "document"` and use the returned `documentSecret`.

## Files

`get_files_meta` lists attachments on entities and returns a `secret` per file.
`download_file` turns a secret into a signed URL valid about 60 minutes, and
inlines the content for images up to 5 MB and text up to 256 KB so you can read
them directly. Larger files and PDFs come back as a URL only.

## Linking back

Always give the user something clickable. `get_entity_links` takes a database
and a list of **public** ids and returns web links. Select `fibery/public-id`
in your query so you can do this without a second round trip.

## Routing to the specialist skills

| The user wants | Skill |
|---|---|
| A precise filtered or aggregated data pull | `fibery-query` |
| To create, update, delete, comment, or attach | `fibery-entities` |
| New databases, fields, relations, workflows, formulas | `fibery-schema` |
| A board, timeline, grid, calendar, gantt, map, or form | `fibery-views` |
| Charts, tables of metrics, time-in-state analysis | `fibery-reports` |
| Import or sync data, audit history, check access | `fibery-admin` |

The MCP also ships its own exhaustive references. Call
`get_fibery_skill` with `"query"`, `"views"`, or `"reports"` when you need the
full operator list, config shape, or expression catalog. The skills here give
you workflow and traps; `get_fibery_skill` gives you the complete grammar.

## Working practices

- **Read before you write.** `schema_detailed` on the target database is not
  optional. Field types decide which write tool is legal.
- **Paginate deliberately.** `q/limit` defaults to 100 and caps at 1000. Fetch
  one page, decide whether it answers the question, then continue with
  `q/offset`. Say so when you truncate.
- **Never invent data.** Report only values a tool returned. If a field was
  empty, say it was empty.
- **Confirm before destruction.** `delete_entities`, `delete_databases`,
  `delete_space`, and `delete_fields` destroy real work. State exactly what
  will be deleted and how many records, then wait for a yes. Deleted spaces
  and views can be restored by the user from the Activity Log; deleted
  entities are not something you should count on recovering.
- **Respect access.** The user's role limits what they can see. A query
  returning nothing may mean no access rather than no data;
  `display_schema_capabilities` tells you which.
