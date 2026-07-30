---
name: crm
description: Core guide for the CRM (Twenty) MCP connection — auth, data model, tool naming conventions, search-first workflow, and filter syntax. Use whenever the user asks to read or change anything in the CRM (contacts, companies, deals, notes, tasks, views, workflows, dashboards, email) and start here before the specialized crm-* skills.
---

# CRM (Twenty)

This skill drives a Twenty CRM workspace through its MCP server. The default
connection is `https://crm.caeros.ai/mcp`; self-hosted workspaces use their own
URL. Auth is a bearer API key stored in the `CRM_MCP_TOKEN` secret.

## Step 0: Connect (if CRM tools are unavailable)

If no CRM tools appear, pause and ask the user to connect:

1. Open Settings → MCP. Find the CRM tile under "Connect an account".
2. Enter the CRM URL (keep the default for crm.caeros.ai, or the self-hosted
   instance URL ending in `/mcp`) and the workspace API key
   (CRM → Settings → API & Webhooks → Generate API key).
3. Connect. The key is saved to the `CRM_MCP_TOKEN` secret and reused on
   every restart.

Retry once the server shows Connected.

## Tool naming conventions

Twenty exposes one tool family per object, in singular and plural forms.
Discover exact names from the connected server's tool list — do not guess:

- **Find <object>** — fetch one record by id (or unique filter).
- **Search <objects>** — full-text + filtered search. Start here.
- **Group <objects>** — aggregate/group-by (counts, sums per field value).
- **Create / Update / Delete <object>(s)** — singular for one record,
  plural for batches.
- **Upsert <objects>** — create-or-update by a matching key. Prefer this for
  imports and enrichment to avoid duplicates.

Cross-cutting tools: `Get Object Metadata`, `Search Help Center`,
`HTTP Request`, `Code Interpreter`, `Extract Json Paths`, `Navigate App`
(deep-link the user's CRM UI to a record or view), `Search Output`.

## Data model essentials

- **Standard objects**: people, companies, opportunities, notes, tasks,
  attachments, calendar events, messages, timeline activities, workspace
  members, lists, blocklists.
- **Custom objects vary per workspace** (this one has deals, family offices,
  campaigns, call recordings, granola connections). Run `Get Object Metadata`
  first when unsure which objects and fields exist; never assume a field.
- **Composite fields** use nested sub-fields:
  - name: `{ firstName, lastName }` (people); companies use a plain `name`.
  - emails: `{ primaryEmail, additionalEmails }`
  - phones: `{ primaryPhoneNumber, primaryPhoneCallingCode, ... }`
  - links (domainName, linkedinLink, xLink): `{ primaryLinkUrl, primaryLinkLabel }`
  - currency (e.g. deal/opportunity amount): `{ amountMicros, currencyCode }`.
    `amountMicros` is the value multiplied by 1,000,000 ($50k = 50000000000).
- **Relations** are set through id fields: `companyId` on a person,
  `pointOfContactId` on an opportunity. Link notes/tasks to records through
  the join objects (note targets, task targets), not directly.
- Select/enum fields (deal stage, task status) accept only their defined
  option values — read the tool schema or object metadata for the exact set.

## Filter and query syntax

Search/Group tools accept Twenty's filter DSL as a **string, not JSON**:
`field[op]:value`, comma-joined for AND, `or(...)` for OR, `not(...)` to
negate. Ops: `eq, neq, in, is (NULL/NOT_NULL), gt, gte, lt, lte, startsWith,
like, ilike`. Nested composite fields use dots: `name.firstName[eq]:Jane`,
`amount.amountMicros[gte]:10000000000`. Dates are ISO 8601. Combine with
`orderBy`, `limit`, and depth parameters where the schema offers them.

## Working rules

1. **Search before create.** Always Search for an existing person/company/deal
   before creating one; use Upsert when syncing external data.
2. **Ids are UUIDs.** Get them from Search/Find results; never fabricate.
3. **Batch with plural tools.** Creating 20 contacts = one `Create people`
   call, not 20 singular calls.
4. **Destructive ops need care.** Delete tools are permanent; confirm with the
   user before deleting records, and prefer archiving-style field updates when
   the workspace has them.
5. **Link work to records.** After creating notes/tasks, immediately create
   the matching targets so they appear on the record timeline.
6. **Show, don't tell.** After significant changes, offer a `Navigate App`
   deep link so the user can open the record or view in the CRM.

## Specialized skills

- `crm-records` — people, companies, family offices, lists, attachments,
  dedupe and enrichment.
- `crm-pipeline` — deals and opportunities: stages, forecasts, reports.
- `crm-productivity` — notes, tasks, targets, timeline activities, calendar.
- `crm-comms` — email drafting/sending, message threads, campaigns,
  call recordings.
- `crm-admin` — object/field metadata, views, navigation, webhooks,
  workspace members, blocklists.
- `crm-workflows` — CRM-native workflow automation (versions, steps,
  triggers, runs).
- `crm-dashboards` — dashboards, tabs, and widgets.
