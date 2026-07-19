---
name: tldraw-sync
description: Multiplayer for tldraw with tldraw sync: useSync and useSyncDemo, self-hosting on Cloudflare Durable Objects or Node, asset handling, and custom sockets. Use when the user adds real-time collaboration to a tldraw app.
---

# tldraw sync (multiplayer)

tldraw sync is the first-party multiplayer module, the same stack behind
tldraw.com, and is included with an SDK license at no extra charge. It is
purpose-built for the tldraw canvas (do not reach for generic CRDT layers
first).

## Quick start: demo server

```tsx
import { Tldraw } from "tldraw"
import { useSyncDemo } from "@tldraw/sync"

function App() {
  const store = useSyncDemo({ roomId: "my-unique-room-id" })
  return <Tldraw store={store} />
}
```

`useSyncDemo` connects to tldraw's hosted demo backend: rooms are public to
anyone with the id and wiped regularly. Prototyping only, never production.

## Production: useSync against your backend

```tsx
import { useSync } from "@tldraw/sync"

const store = useSync({
  uri: `https://sync.example.com/connect/${roomId}`,   // your WebSocket endpoint
  assets: myAssetStore,     // { upload(asset, file), resolve(asset, ctx) }
})
return <Tldraw store={store} />
```

- `useSync` returns a store descriptor with connection status; pass straight
  to `<Tldraw store={...} />` and it renders loading/error states.
- You must provide an asset store: sync moves record data only, images/videos
  go to your storage (S3, R2), `upload` returns the URL, `resolve` can serve
  size-appropriate variants.
- Add `userInfo` for presence (id, name, color); cursors, selections, and
  viewports come for free.
- Custom shapes must be registered on BOTH client and server schema, or sync
  rejects the records.

## Self-hosting the backend

Two official starting points:

1. Cloudflare Durable Objects template (recommended, the tldraw.com
   architecture): repo `tldraw/tldraw-sync-cloudflare`, or
   `npm create tldraw@latest` and pick the multiplayer starter kit. One
   Durable Object per room runs `TLSocketRoom`, persists snapshots to R2, and
   handles asset uploads and bookmark unfurling endpoints.
2. Node: `@tldraw/sync-core` exposes `TLSocketRoom`; the simple-server-example
   repo shows an Express/ws server. You wire: a WebSocket route per room,
   `room.handleSocketConnect`, periodic `room.getCurrentSnapshot()` persistence
   (v4.3+ added a SQLite storage option), and an asset upload route.

Server responsibilities checklist: room lifecycle (create/load/snapshot/
dispose), auth at the WebSocket upgrade (sign the uri or send a token, check
before `handleSocketConnect`), readonly enforcement, asset storage, and the
bookmark metadata (unfurl) endpoint if you use bookmark shapes.

## Custom transports

Since SDK 4.2 `useSync` accepts your own `connect()` implementation, so you
can run sync over socket.io or any transport that delivers ordered messages.
Default is plain WebSockets; stick with that unless infrastructure dictates
otherwise.

## Alternatives and scope

- One doc, no live cursors needed? Plain `persistenceKey` (local, cross-tab)
  or snapshot save/load may be enough; sync adds a server you must operate.
- Community adapters exist for Yjs and other backends, but presence and
  conflict semantics are best-tested on first-party sync.
- Scaling: one room = one socket group; shard rooms across DO instances
  naturally, keep rooms modest (tldraw.com-scale rooms work, but huge
  documents inflate join time since new clients fetch the full snapshot).
- Recent versions keep multi-tab sync at full speed (previously idle rooms
  throttled to one network flush per second); if you see laggy cross-tab
  updates, upgrade the SDK.
