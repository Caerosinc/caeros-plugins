---
name: bigquery
description: Query and manage BigQuery with the bq CLI, SQL patterns (partitioning, clustering), external tables, and cost controls for on-demand and editions pricing. Use when the user is writing BigQuery SQL or managing datasets and query spend.
---

# BigQuery

## bq CLI essentials

```bash
bq ls                                   # datasets in current project
bq mk --dataset --location=EU proj:analytics
bq show --schema --format=prettyjson proj:analytics.events
bq query --use_legacy_sql=false 'SELECT ... FROM `proj.analytics.events`'
bq load --source_format=NEWLINE_DELIMITED_JSON \
  analytics.events gs://bucket/events/*.json ./schema.json
bq extract analytics.events gs://bucket/export/*.parquet \
  --destination_format=PARQUET
bq head -n 20 analytics.events           # peek without a billed query
```

Always `--use_legacy_sql=false` (or set `[query] use_legacy_sql=false` in
`~/.bigqueryrc`). Dataset location is immutable and queries cannot join
across locations; pick EU/US/region deliberately on day one.

## Table design that controls cost

- **Partition** big tables by date: `PARTITION BY DATE(created_at)` (or
  ingestion time `_PARTITIONDATE`). Queries filtering the partition column
  scan only matching partitions.
- **Cluster** within partitions by high-cardinality filter columns:
  `CLUSTER BY user_id, event_type` (up to 4 columns, order matters).
- Require partition filters on huge tables:
  `--require_partition_filter=true` at table creation.
- Never `SELECT *`: BigQuery bills by columns read (columnar storage);
  select only needed fields. `LIMIT` does NOT reduce bytes scanned.
- Useful SQL: `QUALIFY ROW_NUMBER() OVER (...) = 1` for dedupe,
  `APPROX_COUNT_DISTINCT` over exact when close enough, `EXCEPT(col)` in
  star selects, `MERGE` for upserts, `TABLESAMPLE SYSTEM (1 PERCENT)` for
  cheap exploration.

## Cost controls (do these, in order)

1. **Dry run everything**: `bq query --dry_run 'SELECT ...'` prints bytes
   that would be billed, free.
2. **Hard cap per query**: `bq query --maximum_bytes_billed=1000000000 ...`
   fails the query instead of overspending. Set it in `~/.bigqueryrc` so
   every ad hoc query is capped by default.
3. **Custom quotas**: project-level and per-user daily query byte quotas
   (IAM & admin > Quotas, `QueryUsagePerDay`).
4. **Watch actuals**: `INFORMATION_SCHEMA.JOBS_BY_PROJECT` has
   `total_bytes_billed` and `total_slot_ms` per job; build the top-10
   expensive queries report from it.

Pricing model (verify live at cloud.google.com/bigquery/pricing):

- **On-demand**: $6.25/TiB scanned (US multi-region), first 1 TiB/month
  free, 10 MB minimum per query.
- **Editions (slot-hours)**: Standard $0.04, Enterprise $0.06 per
  slot-hour pay-as-you-go; 1yr/3yr commitments cut Enterprise ~20%/40%.
  Break-even versus on-demand typically lands in the hundreds of TiB
  scanned per month; measure real `total_slot_ms` from
  INFORMATION_SCHEMA before switching, the two models do not map 1:1.
- Storage: active vs long-term (untouched 90 days, roughly half price)
  is automatic; consider physical (compressed) billing per dataset for
  highly compressible data.

## External and federated data

- External tables over GCS (Parquet/CSV/JSON): query in place, no load
  step; slower than native and no partition pruning unless you use hive
  partitioning layouts (`gs://bucket/dt=2026-07-19/...`).
- BigLake tables add fine-grained security and better performance over
  data lakes, including Iceberg support.
- Federated queries reach Cloud SQL/Spanner (`EXTERNAL_QUERY`), and Google
  Sheets can back a table (classic silent-failure source: sheet schema
  drift).
- Load (free, batch) versus streaming/Storage Write API (fast, billed per
  GiB): default to batch loads unless you need seconds-level freshness.

## Gotchas

- Query results cache is free and automatic (24h) but disabled by
  non-deterministic functions like `CURRENT_TIMESTAMP()` in some shapes.
- Scheduled queries run as a service account or user; failures are silent
  unless you configure email/Pub/Sub notifications.
- `bq cp analytics.events analytics.events_backup` is a free metadata copy
  within a location: use it before destructive `MERGE`/`DELETE` work, and
  time travel (`FOR SYSTEM_TIME AS OF`) covers the last 7 days.
