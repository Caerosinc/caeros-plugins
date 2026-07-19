---
name: svelte-testing
description: Testing Svelte 5 with Vitest: component tests via vitest-browser-svelte or Testing Library + jsdom, testing runes logic with flushSync, and SvelteKit-specific mocking. Use when the user writes or fixes tests for Svelte code.
---

# Testing Svelte

Two supported lanes, both on Vitest:

1. Browser mode (current recommendation): `vitest-browser-svelte` renders in a
   real browser via Playwright.
2. jsdom: `@testing-library/svelte` with the jsdom environment, lighter and
   fine for most component logic.

`npx sv create` can scaffold Vitest; `sv add vitest` retrofits it.

## Browser mode setup

```js
// vite.config.js
import { defineConfig } from "vitest/config"
import { svelte } from "@sveltejs/vite-plugin-svelte"
export default defineConfig({
  plugins: [svelte()],
  test: {
    browser: { enabled: true, provider: "playwright", instances: [{ browser: "chromium" }] },
  },
})
```

```js
import { render } from "vitest-browser-svelte"
import { expect, test } from "vitest"
import Counter from "./Counter.svelte"

test("increments", async () => {
  const screen = render(Counter, { start: 1 })
  await screen.getByRole("button", { name: "+" }).click()
  await expect.element(screen.getByText("2")).toBeInTheDocument()
})
```

Locators auto-retry: always `await expect.element(...)`, never assert on
snapshots of stale DOM.

## Testing Library + jsdom

```js
// vitest config: test: { environment: "jsdom" }
import { render, screen } from "@testing-library/svelte"
import userEvent from "@testing-library/user-event"

test("increments", async () => {
  const user = userEvent.setup()
  render(Counter, { props: { start: 1 } })
  await user.click(screen.getByRole("button", { name: "+" }))
  expect(screen.getByText("2")).toBeInTheDocument()
})
```

- Svelte 5 needs current `@testing-library/svelte` (v5+); it wires the
  svelte5 entry automatically.
- Pass snippets/children by wrapping in a tiny test host component; snippets
  cannot be constructed in plain JS.
- If components resolve to server-side code under jsdom, ensure the plugin
  sets browser resolve conditions (current @sveltejs/vite-plugin-svelte plus
  vitest handles this; `resolve.conditions: ["browser"]` is the manual fix).

## Testing runes logic

Rune files (`.svelte.js`/`.svelte.ts`) are testable if the TEST FILE is also
named `*.svelte.test.js` so the compiler processes runes in it:

```js
// counter.svelte.test.js
import { flushSync } from "svelte"
import { createCounter } from "./counter.svelte.js"

test("derived updates", () => {
  const c = createCounter()
  c.increment()
  flushSync()                  // effects/derived settle synchronously
  expect(c.doubled).toBe(2)
})
```

- Wrap code that creates `$effect` in `$effect.root(() => {...})` inside tests
  and call the returned cleanup.
- `flushSync()` from `"svelte"` forces pending updates; use it instead of
  awaiting ticks in unit tests.

## SvelteKit specifics

- Mock `$app/state`, `$app/navigation`, `$env/*` with `vi.mock("$app/state", ...)`;
  aliases resolve in tests when the config comes from `vitest/config` and the
  project uses the SvelteKit Vite plugin.
- Test load functions and actions as plain functions: call them with a stubbed
  event (`{ params, fetch: vi.fn(), cookies, locals }`) and assert on the
  return or thrown redirect/error.
- End-to-end belongs in Playwright proper (`npx sv add playwright`); keep
  Vitest for units and components.
- Common failure: "lifecycle_outside_component" means a component API was
  called outside `render`; "cannot use $state in .test.js" means the test file
  lacks the `.svelte.test.js` naming.
