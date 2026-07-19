---
name: mongodb-atlas
description: Operate MongoDB Atlas, the Atlas CLI, cluster management, local dev deployments, Atlas Search, and vector search. Use when the user works with Atlas clusters or wants semantic/vector queries on MongoDB.
---

# MongoDB Atlas

Atlas is MongoDB's managed service: clusters, backups, search, and vector
search in one control plane. The MongoDB MCP server can run many of these
operations when given Atlas service-account credentials (see
`mongodb-connection-setup`).

## Atlas CLI

```bash
brew install mongodb-atlas-cli        # or download from mongodb.com/try/download/atlascli
atlas auth login                      # browser OAuth
atlas projects list
atlas clusters list --projectId <id>
atlas clusters create dev01 --provider AWS --region US_EAST_1 --tier M10
atlas clusters describe dev01
atlas dbusers create --username app --role readWrite@appdb --password '...'
atlas accessLists create --currentIp   # allow your IP through the firewall
```

- Free tier: `--tier M0` (one per project, shared, no backups). M10+ is
  dedicated and supports metrics-driven autoscaling.
- Two blockers cause almost every "cannot connect" report: the IP access list
  and the database user. Check both before debugging drivers.

## Local dev without a cloud cluster

```bash
atlas deployments setup --type local   # runs a local Atlas-like deployment in a container
atlas deployments list
atlas deployments connect
```

This gives you Atlas Search and Vector Search locally, which plain
`mongod` does not have.

## Atlas Search (full text)

Create a search index, then query with `$search`:

```js
db.products.createSearchIndex("default", { mappings: { dynamic: true } })
db.products.aggregate([
  { $search: { index: "default", text: { query: "wireless headphones", path: ["name", "description"] } } },
  { $limit: 10 },
  { $project: { name: 1, score: { $meta: "searchScore" } } }
])
```

## Vector search

1. Create a vector index (mongosh, driver, CLI, or UI):

```js
db.docs.createSearchIndex("vec_idx", "vectorSearch", {
  fields: [
    { type: "vector", path: "embedding", numDimensions: 1536, similarity: "cosine" },
    { type: "filter", path: "tenantId" }
  ]
})
```

2. Query with `$vectorSearch` as the FIRST pipeline stage:

```js
db.docs.aggregate([
  { $vectorSearch: {
      index: "vec_idx", path: "embedding",
      queryVector: embedding,            // array of floats from your embedding model
      numCandidates: 200,                // ANN beam width; ~10-20x limit is a good start
      limit: 10,
      filter: { tenantId: "t1" }         // only fields indexed as type "filter"
  } },
  { $project: { text: 1, score: { $meta: "vectorSearchScore" } } }
])
```

Gotchas:
- `numDimensions` must exactly match your embedding model's output size.
- Filters must be declared in the index as `type: "filter"`; an ordinary
  `$match` after `$vectorSearch` filters AFTER the top-k, silently dropping
  results.
- Similarity options: `cosine`, `dotProduct`, `euclidean`. Use what the
  embedding model was trained for (cosine is the safe default).
- Index builds are asynchronous; poll `db.docs.getSearchIndexes()` until
  `status: "READY"` before querying.

## Operations worth automating

- `atlas backups snapshots list/create` for on-demand snapshots (M10+).
- `atlas metrics processes` for CPU/connections when a cluster feels slow.
- `atlas clusters pause/start` to stop paying for idle dev clusters.
- Programmatic control uses service accounts (client ID/secret) via the Atlas
  Admin API; scope them to a single project with the least role that works.
