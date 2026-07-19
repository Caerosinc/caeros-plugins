---
name: redis-caching
description: Build correct cache layers on Redis, cache-aside, invalidation strategies, stampede protection, hot keys, and eviction policies. Use when the user is adding or debugging caching.
---

# Redis Caching

## Cache-aside (the default pattern)

```python
def get_user(uid):
    key = f"user:{uid}"
    cached = r.get(key)
    if cached is not None:
        return json.loads(cached)
    user = db.query_user(uid)                       # source of truth
    r.set(key, json.dumps(user), ex=3600 + random.randint(0, 300))
    return user
```

Rules that keep it correct:
- The database stays the source of truth; Redis is disposable. Code must work
  (slower) with Redis down: wrap reads in try/except and fall through to the DB.
- Always set a TTL. A cache without TTLs is a second database with no
  consistency story.
- Add TTL jitter so a deploy-time warmup does not create a synchronized
  expiry wave.
- Cache negative results too (`SET key "__miss__" EX 60`) if callers probe
  for missing entities, otherwise every miss hits the DB.

## Invalidation

Ranked by preference:

1. **Delete on write**: after a successful DB update, `DEL user:42`
   (or `UNLINK`). Delete, do not update-in-place: two racing writers can
   leave a stale value, but a delete at worst causes one extra miss.
2. **Short TTLs** as the backstop for everything you forgot to invalidate.
3. **Versioned keys** for list/aggregate caches you cannot enumerate:
   `INCR ver:products`, and build keys as `products:v{ver}:page:1`. Old
   versions expire naturally.
4. **Pub/Sub or keyspace notifications** to invalidate in-process caches on
   other nodes (`CONFIG SET notify-keyspace-events Ex` for expiry events).

The classic race: read-miss loads old DB row, a write + DEL lands, then the
reader SETs the stale row back. Mitigations: short TTL on cache fills, or a
brief "dirty" tombstone (`SET dirty:user:42 1 EX 5` checked before filling).

## Stampede protection

When a hot key expires, hundreds of workers hit the DB at once. Options:

- **Mutex rebuild**: only the lock winner recomputes.

```python
if r.set(f"lock:{key}", token, nx=True, ex=10):
    value = rebuild(); r.set(key, value, ex=ttl)
    # release with a Lua compare-and-delete on token, never plain DEL
else:
    time.sleep(0.05); retry_get(key)      # or serve slightly-stale copy
```

- **Serve-stale**: store `{value, soft_expiry}` with a hard TTL much longer;
  after soft expiry, one caller refreshes (via the lock), others keep serving
  the stale value.
- **Probabilistic early refresh**: each hit refreshes with probability rising
  as TTL approaches zero, spreading rebuilds out.

## Hot keys and big values

- One extremely hot key pins one shard/CPU. Split read load with N replicas
  of the key (`conf:copy:{0..9}`, read a random one) or cache it in-process
  with a 1-5s TTL.
- Keep values under ~100KB; megabyte values stall the single-threaded event
  loop and the network. Compress or split.

## Eviction policy

For a dedicated cache instance:

```
CONFIG SET maxmemory 2gb
CONFIG SET maxmemory-policy allkeys-lfu    # or allkeys-lru
```

- `allkeys-lfu`/`allkeys-lru`: pure cache, anything can go.
- `volatile-*`: only keys with TTLs are evictable; use when the same instance
  also holds must-keep data (better: separate instances).
- Default `noeviction` returns write errors at maxmemory, which is almost
  never what a cache wants.

Watch `INFO stats`: `keyspace_hits` / `keyspace_misses` gives the hit rate;
`evicted_keys` climbing means maxmemory is undersized or TTLs are too long.
