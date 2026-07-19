---
name: sveltekit
description: SvelteKit essentials: file-based routing, load functions, form actions with progressive enhancement, page options, hooks, and adapters. Use when the user builds routes, data loading, or deployment for a SvelteKit app.
---

# SvelteKit

SvelteKit 2 on Svelte 5, Vite-powered. Scaffold: `npx sv create my-app`
(the `sv` CLI also handles `sv add` integrations and `sv migrate`).

## Routing (src/routes)

- `+page.svelte` UI, `+page.js` universal load, `+page.server.js` server-only
  load + actions, `+layout.svelte` / `+layout.server.js` nested wrappers,
  `+server.js` API endpoints (`export function GET/POST`), `+error.svelte`.
- Dynamic: `[slug]`, rest `[...path]`, optional `[[lang]]`, matchers
  `[id=integer]` (defined in `src/params`). Group dirs `(marketing)` organize
  without affecting URLs.
- Navigation state: `import { page, navigating } from "$app/state"` (runes
  based, replaces the older `$app/stores` in current SvelteKit; read as plain
  `page.url`, `page.data`, no `$` prefix).

## Load functions

```js
// +page.server.js  (server only: DB, secrets)
export async function load({ params, fetch, cookies, locals, depends }) {
  return { post: await db.getPost(params.slug) }
}
```

- Universal load (`+page.js`) runs on server for SSR then in browser; server
  load runs only server-side and its return must be serializable (devalue:
  Dates, Maps, BigInt ok; class instances not).
- Data merges down layouts; access in the page as
  `let { data } = $props()`.
- Use the provided `fetch` (cookie forwarding, relative URLs, SSR inlining).
- Errors and redirects: `error(404, "Not found")` and `redirect(303, "/login")`
  from `@sveltejs/kit` are called directly in SvelteKit 2 (no `throw` needed,
  they throw internally).
- Rerun control: `depends("app:posts")` + `invalidate("app:posts")`, or
  `invalidateAll()`. Streaming: return a promise nested in an object and
  `{#await}` it in the page.

## Form actions

```js
// +page.server.js
import { fail } from "@sveltejs/kit"
export const actions = {
  default: async ({ request, cookies }) => {
    const data = await request.formData()
    const email = data.get("email")
    if (!email) return fail(400, { email, missing: true })
    return { success: true }
  },
}
```

```svelte
<script>
  import { enhance } from "$app/forms"
  let { form } = $props()   // last action result
</script>
<form method="POST" use:enhance>
  {#if form?.missing}<p>Email required</p>{/if}
  <input name="email" value={form?.email ?? ""} />
</form>
```

Named actions (`?/register`) let one page host several forms. `use:enhance`
progressively enhances (works without JS); customize with a submit callback
returning `({ result, update }) => ...` and `applyAction`.

## Page options and hooks

- Per page/layout exports: `export const prerender = true`, `ssr = false`,
  `csr = false`, `trailingSlash`. Static site = `prerender` on the root
  layout + adapter-static.
- `src/hooks.server.js`: `handle({ event, resolve })` for auth/sessions
  (populate `event.locals`), `handleFetch`, `handleError`. Compose multiple
  with `sequence()`.
- Env: `$env/static/private`, `$env/static/public` (PUBLIC_ prefix),
  dynamic variants for runtime values. Private modules cannot be imported
  into client code (build-time enforced).

## Adapters

`@sveltejs/adapter-auto` (default, detects platform), `adapter-node`
(standalone Node server), `adapter-static` (full prerender/SPA),
`adapter-vercel`, `adapter-netlify`, `adapter-cloudflare`. Set in
`svelte.config.js` under `kit.adapter`. Deploy failures on platform APIs
usually mean code assumed Node while the adapter targets edge runtimes; gate
with `$app/environment` (`browser`, `building`) and platform-specific
`event.platform`.
