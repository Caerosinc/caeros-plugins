---
name: email-verification
description: Verify email addresses and clean lead lists with the Caeros enrichment engine (MillionVerifier / BounceBan). Use when asked to verify emails, check deliverability, clean or dedupe an email list, or prepare a send list for a campaign.
---

# Email Verification

The Caeros gateway runs a server-side enrichment engine: it splits, dedupes,
and verifies email lists of any size (one address to a million), caches every
verdict for 30 days, and exports CSV. You drive it with the native
`enrich_*` tools; runs keep going even if the session disconnects.

## The flow (always in this order)

1. **`enrich_providers`** — confirm a provider is configured (free).
2. **`enrich_email_preview`** — verify a REPRESENTATIVE sample of up to 10
   addresses synchronously. Show the user the verdicts and the total list
   size. This is the review gate: never launch a full run before the user
   has seen a sample forecast and agreed.
3. **`enrich_run_start`** — launch the full run with `confirm_spend: true`
   only after the user agrees. Pass the addresses as `emails` (array) or
   `text` (raw CSV/lines; the gateway extracts and dedupes).
4. **`enrich_run_status`** — poll the counters. Do NOT busy-poll: the tool
   directs you to stop after a few still-running snapshots; small runs
   finish in seconds, large ones continue server-side regardless.
5. **`enrich_run_results`** — page per-address results, or filter with
   `status: "deliverable"` to get just the send-safe rows.
6. **`enrich_run_cancel`** — stop a running run; verified rows keep their
   results (and stay cached).

## Verdicts (normalized across providers)

| status | meaning | typical reasons |
|---|---|---|
| `deliverable` | mailbox confirmed, safe to send | |
| `risky` | exists but delivery unsure | `catch_all` (domain accepts everything), `disposable` |
| `undeliverable` | do not send | `no_mailbox`, `dns_error`, `syntax` |
| `unknown` | provider could not decide | retry later or treat as risky |

- `cached: true` rows cost zero credits (verdicts cached 30 days; a preview
  warms the cache, so previewed addresses are free again in the full run).
- Syntactically invalid addresses are rejected by the local checker
  (`provider: "local"`, reason `syntax`) at zero cost and zero latency.
- Cost: up to 1 provider credit per never-seen address, nothing else.

## Big lists: the workflow kind

For a list produced by another task (file extraction, a scrape, an upstream
agent), use a `workflow_run` task with `kind: "enrich"` — one task is the
whole run, no per-row fan-out (never `foreach` over emails):

```json
{"name":"verify-leads","tasks":[
  {"id":"extract","prompt":"Extract every email address from ~/leads.csv and output them one per line."},
  {"id":"verify","kind":"enrich","prompt":"${task:extract}","deps":["extract"]}
]}
```

The enrich task's prompt is the address source text; its output is
`{run_id, status, total_items, counters}`. Put the preview forecast in the
run description so the launch approval card shows what will be spent.

## Providers

`millionverifier` is preferred when both are configured; `bounceban` is the
alternative. Pin one per run with the `provider` argument, or omit it for
the gateway default. If no provider is configured, `enrich_run_start` fails
with `provider_unavailable` — load the `lead-enrichment-setup` skill.

## CSV export

The full results CSV (all columns: status, reason, provider fields,
cached, error class) streams from the gateway API:

```
GET {gateway}/v1/enrich/runs/{run_id}/results?format=csv
Authorization: Bearer <session JWT>
```

Optional `status=<verdict>` filters the export. For in-chat summaries
prefer the counters from `enrich_run_status` over paging every row.
