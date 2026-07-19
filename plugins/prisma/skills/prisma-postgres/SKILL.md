---
name: prisma-postgres
description: Use Prisma Postgres, the managed PostgreSQL service, creating databases, connecting from Prisma ORM and other clients, built-in pooling, caching, and backups. Use when the user hosts a database on Prisma Postgres or asks where to put a Postgres for a Prisma app.
---

# Prisma Postgres

Prisma's managed PostgreSQL (PostgreSQL 17, unikernel-based): built-in
connection pooling (a dedicated PgBouncer instance per database), query
caching, daily backups with point-in-time recovery, and serverless/edge
support. Usage-based pricing with spend controls in Prisma Console
(console.prisma.io).

## Create a database

- Fastest, disposable: `npx create-db@latest` provisions a temporary Prisma
  Postgres database in one command (claim it in the Console to keep it).
- Standard: create it in Prisma Console (or during `prisma init`), pick a
  region near your compute, then copy the connection string into `.env` as
  `DATABASE_URL`. Never commit `.env`.
- The Prisma MCP server in this plugin can act on your Prisma Postgres
  account: check auth status (Prisma-Postgres-account-status), and the remote
  variant (`https://mcp.prisma.io/mcp` via `npx -y mcp-remote`) adds
  backup/restore and connection-string management tools.

## Connect

From Prisma ORM: it is just Postgres, so the normal flow applies:

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

```bash
npx prisma migrate dev --name init
```

Other paths into the same database:
- Any Postgres client works: psql, GUI tools, node-postgres, other ORMs
  (Drizzle, Kysely, TypeORM), via the direct TCP connection string.
- Serverless/edge runtimes without TCP: the `@prisma/ppg` serverless driver
  speaks HTTP/WebSocket to Prisma Postgres.

Pooling is built in, so you normally do NOT need your own PgBouncer, and the
pool-exhaustion rituals from bare Postgres (tuning `connection_limit` per
lambda) mostly disappear. Long-lived migrations and `CREATE INDEX
CONCURRENTLY` style operations should still use the direct (non-pooled)
string if the Console provides one for your plan.

## Query caching

Prisma Postgres integrates result caching (the capability formerly sold
separately as Accelerate). Per query:

```ts
const posts = await prisma.post.findMany({
  where: { published: true },
  cacheStrategy: { ttl: 60, swr: 120 },   // serve cached 60s, stale-while-revalidate 120s
})
```

- `ttl`: seconds a result is served without touching the DB.
- `swr`: after ttl, serve stale immediately while refreshing in background.
- Cache only read-heavy, staleness-tolerant queries (feeds, counts,
  catalogs); skip it for anything read-after-write sensitive.
- Requires the client extension wiring shown in the Console snippet for your
  project (`@prisma/extension-accelerate`); verify against current docs since
  this surface has been consolidating into Prisma Postgres.

## Operations

- Backups: daily automated snapshots plus point-in-time recovery; restores
  create a new database (also exposed as MCP tools: CreateBackupTool,
  CreateRecoveryTool on the remote server).
- Connection strings can be created and revoked individually
  (CreateConnectionStringTool / DeleteConnectionStringTool): issue one per
  environment so a leaked staging string never touches prod.
- Watch usage and set spend limits in Console before pointing load tests at
  it; usage-based billing rewards that habit.

## When NOT to choose it

- You need Postgres extensions it does not offer (verify your must-have
  extensions in current docs before committing).
- Data-residency rules require a region it does not serve.
- You are all-in on another cloud's IAM/VPC peering story; Prisma Postgres
  connects over public TLS endpoints.
