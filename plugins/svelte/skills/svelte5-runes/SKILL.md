---
name: svelte5-runes
description: Svelte 5 runes reactivity: $state, $derived, $effect, $props, $bindable, shared state in .svelte.js modules, and migration from stores and $:. Use when the user writes Svelte 5 reactivity or migrates Svelte 4 code.
---

# Svelte 5 runes

Runes are compiler keywords (no import) and the reactivity model for Svelte 5.
They work in `.svelte` files and in `.svelte.js` / `.svelte.ts` modules.

## $state

```svelte
<script>
  let count = $state(0)
  let user = $state({ name: "Ada", tags: ["dev"] })  // deep reactive proxy
  let big = $state.raw(largeObject)  // no proxy: reassign to update
</script>
<button onclick={() => count++}>{count}</button>
```

- Objects/arrays become deep proxies: `user.tags.push("x")` is reactive.
- `$state.raw` for large or immutable data (only reassignment triggers).
- `$state.snapshot(user)` gives a plain non-proxy copy (for structuredClone,
  console logging, sending to APIs).
- Class fields can be `$state`, giving reactive class instances.

## $derived and $effect

```js
let doubled = $derived(count * 2)
let total = $derived.by(() => items.reduce((a, b) => a + b.price, 0))

$effect(() => {
  console.log("count is", count)     // dependencies tracked automatically
  return () => clearInterval(id)     // teardown before rerun and on destroy
})
```

- `$derived` is lazy, memoized, and must be side-effect free. Prefer it over
  `$effect` + assignment (a classic anti-pattern the compiler cannot save you
  from).
- Only what a derived/effect reads synchronously is a dependency; reads inside
  `await`/`setTimeout` are not tracked. `untrack(fn)` opts a read out.
- `$effect.pre` runs before DOM updates (scroll position work);
  `$effect.root` creates a manually controlled scope outside components.
- Effects do not run during SSR.

## $props and $bindable

```svelte
<script>
  let { title, count = 0, children, ...rest } = $props()
  let { value = $bindable("") } = $props()   // parent can bind:value
</script>
```

Props are reactive when destructured this way. `$bindable` opts a prop into
two-way binding; everything else is one-way. `$props.id()` gives a stable
unique id (label/input pairing). `$inspect(count)` is a dev-only reactive
console.log; `$host()` exposes the custom-element host.

## Shared state in modules

```js
// counter.svelte.js
export const counter = $state({ value: 0 })
export function increment() { counter.value++ }
```

Export the object, mutate its properties: reassigning an exported `$state`
binding is not allowed across modules. This pattern replaces most writable
stores. Caution in SSR: module-level state is shared across requests, keep
per-user state in components or context (`setContext`/`getContext`).

## Migration from Svelte 4

| Svelte 4 | Svelte 5 |
|---|---|
| `let x = 0` (reactive) | `let x = $state(0)` |
| `$: y = x * 2` | `let y = $derived(x * 2)` |
| `$: { sideEffect(x) }` | `$effect(() => sideEffect(x))` |
| `export let prop` | `let { prop } = $props()` |
| `writable()` store for local/shared state | `$state` (module or component) |

- Stores still work (`$store` autosubscription included) and remain right for
  async/custom subscription sources; new code should default to runes.
- `svelte-migrate` automates most of it: `npx sv migrate svelte-5`.
- Mixing: a component uses either runes or legacy reactive statements, not
  both semantics for the same variable; the compiler infers mode per file.
- The official Svelte MCP server autofixer flags outdated patterns
  (`on:click`, `$:` in runes mode, store overuse): run generated code
  through it.
