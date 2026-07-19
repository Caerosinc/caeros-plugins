---
name: svelte-components
description: Svelte 5 component authoring: snippets and render tags, event attributes, callback props, bindings, keyed each, class/style directives, and error boundaries. Use when the user writes or reviews Svelte component markup and APIs.
---

# Svelte 5 components

## Snippets replace slots

```svelte
<!-- Card.svelte -->
<script>
  let { header, children } = $props()
</script>
<article>
  {#if header}{@render header()}{/if}
  {@render children?.()}
</article>

<!-- usage -->
<Card>
  {#snippet header()}<h2>Title</h2>{/snippet}
  <p>Body content becomes the children snippet.</p>
</Card>
```

- Content not wrapped in an explicit `{#snippet}` becomes `children`.
- Snippets take parameters: `{#snippet row(item, i)}...{/snippet}` passed as a
  prop and rendered with `{@render row(item, 0)}`; this replaces `let:` slot
  props and is the idiom for render-prop style lists and tables.
- Snippets are values: define once, pass to several components, or declare
  top-level in a file and `export` from `<script module>`.
- Type as `Snippet` / `Snippet<[Item, number]>` from `"svelte"`.

## Events are attributes, callbacks are props

```svelte
<button onclick={handle} onkeydown={(e) => e.key === "Enter" && go()}>Go</button>
```

- `on:click` is legacy; runes-mode components use plain `onclick={fn}`
  attributes (case sensitive, no modifiers: call
  `e.preventDefault()` yourself or wrap the handler).
- `createEventDispatcher` is deprecated. Components expose callback props:

```svelte
<script>
  let { onselect } = $props()
</script>
<li onclick={() => onselect?.(item)}>...</li>
```

- Spread `...rest` from `$props()` onto the root element so consumers can add
  `onclick`, `class`, `aria-*` without wiring each one.

## Bindings

- Form: `bind:value`, `bind:checked`, `bind:group`, `bind:files`,
  `<select bind:value>`; numeric inputs coerce to numbers.
- Element/instance: `bind:this={el}` for DOM nodes or component instances;
  size observers `bind:clientWidth`, media `bind:currentTime`.
- Component props are bindable only if declared with `$bindable` in the child;
  otherwise `bind:` errors. Keep two-way binding rare and deliberate.
- Function bindings (Svelte 5.9+): `bind:value={get, set}` for validated or
  transformed two-way flow.

## Template syntax worth using

- Keyed each: `{#each items as item (item.id)}` (always key dynamic lists).
- `{#if}/{:else if}`, `{#await promise}...{:then v}...{:catch e}`,
  `{#key expr}` to force re-creation, `{@const total = a + b}`, `{@html str}`
  (sanitize first).
- Class: `class={["btn", { active, danger: kind === "danger" }]}` (Svelte 5.16
  accepts objects/arrays, clsx style) or `class:active` directive.
- Style: `style:color={c}`, `style:--track-color={c}` to feed CSS custom
  properties; component styles are scoped by default, escape with
  `:global(...)`.
- `<svelte:boundary onerror={(e, reset) => ...}>` with a `{#snippet failed(error, reset)}`
  block catches rendering/effect errors in its subtree (Svelte 5.3+).
- `<svelte:window>`, `<svelte:document>`, `<svelte:head>` for
  listeners/metadata; `<svelte:element this={tag}>` for dynamic tags.

## Lifecycle and imperative mounting

- Prefer `$effect` over `onMount` in runes mode; `onMount`/`onDestroy` still
  work. `tick()` awaits pending DOM updates.
- Components are no longer classes: imperative use is
  `mount(App, { target, props })` and `unmount(app)` from `"svelte"`
  (`new App(...)` is Svelte 4 only; `createRoot` does not exist).
- Transitions remain: `transition:fade`, `in:fly={{ y: 20 }}`, `animate:flip`
  on keyed each items.
