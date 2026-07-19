---
name: mongodb-schema-design
description: Model documents in MongoDB, embedding vs referencing, schema patterns, avoiding unbounded arrays, and JSON Schema validation. Use when the user is designing or refactoring MongoDB collections.
---

# MongoDB Schema Design

The prime rule: **data that is accessed together should be stored together.**
Design for your query patterns, not for third normal form.

## Embed vs reference

Embed when:
- The child data is always read with the parent (order + line items).
- The child list is bounded and small (a user's few addresses).
- The child has no meaning outside the parent.

Reference when:
- The relation is high-cardinality or unbounded (user -> events).
- The child is queried independently or shared across parents (products
  referenced by many orders).
- The child is large and rarely needed with the parent.

Hard limits that force referencing: 16MB max document size, and large arrays
degrade update performance long before that. Treat any array that grows
without bound (comments, logs, followers) as a design bug.

```js
// Embedded (read together, bounded)
{ _id, name, addresses: [{ label: "home", city: "Casablanca" }] }

// Referenced (unbounded), child points at parent
{ _id, orderId, userId: ObjectId("..."), total: 42 }   // orders collection
```

Put the foreign key on the many side; join with `$lookup` only when a screen
truly needs both sides.

## Patterns worth naming

- **Extended reference**: copy the 2-3 parent fields you always display into
  the child (`order.customer: { _id, name }`). Accept the duplication; update
  it on the rare parent rename.
- **Subset**: keep the 10 most recent reviews embedded in the product, full
  history in a `reviews` collection.
- **Bucket**: group time-series points into one doc per device per hour
  (`{ deviceId, hour, points: [...] }`). For serious loads use native
  time-series collections (`db.createCollection("m", { timeseries: {...} })`).
- **Computed**: store precomputed aggregates (`ratingAvg`, `ratingCount`) and
  update them on write, instead of aggregating on every read.
- **Schema versioning**: put `schemaVersion: 2` in docs and migrate lazily on
  read/write instead of a big-bang migration.
- **Polymorphic**: heterogeneous docs in one collection with a `type` field
  work well; index `{ type: 1, ... }`.

Anti-patterns: unbounded arrays, one collection per tenant/day (kills the
working set with thousands of collections + indexes), massive `$lookup` fan-out
that reimplements a relational join per request.

## Validation

Enforce shape at the collection with JSON Schema:

```js
db.createCollection("users", {
  validator: { $jsonSchema: {
    bsonType: "object",
    required: ["email", "createdAt"],
    properties: {
      email: { bsonType: "string", pattern: "^.+@.+$" },
      plan: { enum: ["free", "pro", "enterprise"] },
      createdAt: { bsonType: "date" }
    }
  } },
  validationLevel: "moderate",     // "strict" checks all writes; moderate skips existing bad docs
  validationAction: "error"        // or "warn" to log instead of reject
})
```

Add to an existing collection with `db.runCommand({ collMod: "users",
validator: ... })`. Start with `validationAction: "warn"` on live systems,
watch the logs, then flip to `error`.

## Checklist before shipping a schema

1. List the top 5 queries and writes; every one should hit an index and
   ideally a single document.
2. Any array field: what bounds its growth? If nothing, reference instead.
3. Duplicated fields: who updates them, and how stale is acceptable?
4. `_id` choice: default ObjectId is fine; natural keys only if truly
   immutable.
5. Write a validator for anything another team or service writes to.
