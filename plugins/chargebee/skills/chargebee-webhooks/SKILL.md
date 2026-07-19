---
name: chargebee-webhooks
description: Handle Chargebee webhook events reliably, endpoint auth, deduplication, key event types, dunning flows, and reconciliation patterns. Use when building the server that consumes Chargebee events.
---

# Chargebee Webhooks

Configure under Settings > Configure Chargebee > API Keys and Webhooks:
HTTPS endpoint URL, optional Basic Auth username/password (set them, verify
on every request), API version for payloads, and optional event filtering.
Test and live sites have separate webhook configs; wire both.

## Delivery semantics (design for these)

- Chargebee POSTs a JSON body containing an `event` with `id`,
  `event_type`, `occurred_at`, `api_version`, and a `content` object holding
  the affected resources (`subscription`, `customer`, `invoice`,
  `transaction`, ...).
- Respond 2xx fast; queue real work. Failed deliveries are retried, so
  duplicates and out-of-order arrivals happen.
- **Deduplicate by `event.id`** in a processed-events table.
- **Do not trust payload freshness**: on receipt, re-fetch the resource
  (`GET /subscriptions/{id}` etc.) and act on current state. This makes
  reordering harmless.
- Verify the endpoint's Basic Auth header; optionally also allowlist by
  fetching the event back (`GET /events/{event_id}`) as authenticity proof.

## Event types to handle first

Subscription lifecycle:
- `subscription_created`, `subscription_activated` (trial converted),
  `subscription_changed`, `subscription_renewed`,
  `subscription_cancelled`, `subscription_reactivated`,
  `subscription_paused`, `subscription_resumed`.

Money and dunning:
- `payment_succeeded`, `payment_failed`, `payment_refunded`
- `invoice_generated`, `invoice_updated`
- `subscription_cancellation_scheduled` (cancel at term end pending)
- `card_expiry_reminder`, `payment_source_added`, `payment_source_updated`

Minimal state machine: grant on `subscription_created`/`activated`, adjust
on `subscription_changed`, flag on `payment_failed`, revoke on
`subscription_cancelled`, clear flags on `payment_succeeded` or
`subscription_reactivated`.

## Dunning (failed-payment recovery)

Chargebee retries failed renewal payments per the site's dunning settings
(retry schedule, then final action: cancel, mark unpaid, or leave). Your
job as the integrator:

- On `payment_failed`: mark the account at-risk, surface an in-app banner,
  email a payment-update link (hosted page `manage_payment_sources` or a
  portal session). Do not cut access on first failure.
- On `payment_succeeded` after failures: clear the at-risk state.
- On the terminal event (`subscription_cancelled` or invoice left unpaid):
  downgrade or revoke, and keep the invoice reference for support.
- Track `invoice.dunning_status` (`in_progress`, `stopped`, `exhausted`)
  when reconciling.

## Idempotent side effects

Webhook processing must be replay-safe end to end:

- Guard external side effects (emails, provisioning) with the event ID.
- When your handler calls Chargebee mutation APIs, send
  `chargebee-idempotency-key` headers so your own retries cannot
  double-apply (see `chargebee-billing-integration` for the exact rules).

## Reconciliation backstop

Webhooks can be missed (downtime past the retry window). Run a periodic
sync using `GET /events?occurred_at[after]=<checkpoint>` or list
subscriptions filtered by `updated_at[after]` to heal drift. Persist the
checkpoint only after successful processing.
