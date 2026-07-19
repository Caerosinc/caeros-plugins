---
name: redis-connection-setup
description: Connect Caeros and apps to Redis, the official Redis MCP server (redis-mcp-server via uvx), environment variables, TLS, ACL users, and connection triage. Use when setting up or debugging a Redis connection.
---

# Redis Connection Setup

## The MCP server this plugin runs

Official server from the redis/mcp-redis project, published on PyPI as
`redis-mcp-server`, launched with uv's runner:

```
uvx --from redis-mcp-server@latest redis-mcp-server
```

Requirement: `uv` must be installed (`brew install uv`, or
`curl -LsSf https://astral.sh/uv/install.sh | sh`). No Redis client install
needed; uvx resolves the package on first run.

## Configure via environment (never inline credentials)

The server reads standard variables at startup:

```bash
export REDIS_HOST=127.0.0.1        # default 127.0.0.1
export REDIS_PORT=6379             # default 6379
export REDIS_DB=0
export REDIS_USERNAME=default
export REDIS_PWD='...'             # note: PWD, not PASSWORD
export REDIS_SSL=True              # plus, when using TLS:
export REDIS_SSL_CA_PATH=/path/ca.pem
export REDIS_CLUSTER_MODE=False
```

Alternatively a single URL argument (args in `.mcp.json`):
`--url redis://user:pass@host:6379/0` or, TLS,
`rediss://user:pass@host:port?ssl_cert_reqs=required&ssl_ca_certs=/path/ca.pem`.
Prefer env vars: CLI args show up in process lists. Restart the server after
changing them.

For managed Redis Cloud subscriptions there is a separate official server
(mcp-redis-cloud) for account-level management; this plugin targets the data
plane of an instance you already have.

## Local Redis in one command

```bash
docker run -d --name redis -p 6379:6379 redis:8       # includes query engine + JSON
redis-cli ping        # PONG
```

## Least-privilege ACL users

Do not hand agents the `default` user. Create a scoped one:

```
ACL SETUSER caeros_ro on >S3cret ~app:* +@read +@connection -@dangerous
ACL SETUSER caeros_rw on >S3cret ~app:* +@read +@write +@keyspace -@dangerous -@admin
ACL LIST
ACL WHOAMI
```

- `~app:*` limits key access by pattern; `%R~pat` / `%W~pat` split read/write
  patterns.
- Category shorthand: `+@read`, `+@write`, `-@dangerous` (blocks FLUSHALL,
  KEYS, SHUTDOWN, CONFIG and friends), `+@search` for FT.* commands,
  `+@json` for JSON.*.
- Persist ACLs: `CONFIG SET aclfile /etc/redis/users.acl` + `ACL SAVE`, or
  define users in redis.conf. In Redis Cloud, manage ACLs in the console.

## Triage ladder

1. `redis-cli -h HOST -p PORT ping`: reachability. Timeout = network,
   firewall, or wrong port; managed Redis is usually TLS on a nonstandard
   port (then add `--tls`).
2. `NOAUTH Authentication required` = password not sent;
   `WRONGPASS` = bad user/password pair (remember `REDIS_PWD`).
3. `NOPERM` = ACL blocks the command or key pattern; check `ACL GETUSER u`.
4. TLS handshake errors: `rediss://` vs `redis://` mismatch, or missing CA
   file (`ssl_cert_reqs=none` only as a diagnostic, never in production).
5. `MOVED 1234 ...` errors = cluster mode target; set
   `REDIS_CLUSTER_MODE=True`.
6. Slow but connected: `redis-cli --latency`, then `SLOWLOG GET 10`.

## Hygiene

- One Redis logical DB per app is a weak boundary; prefer separate instances
  (or at least ACL key patterns) per trust domain.
- Never expose Redis directly to the internet without TLS + ACLs; default
  Redis trusts its network.
- Rotate any password that has ever been pasted into a chat, log, or shell
  history file.
