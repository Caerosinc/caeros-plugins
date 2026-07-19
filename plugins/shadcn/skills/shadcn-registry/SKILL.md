---
name: shadcn-registry
description: Build and publish shadcn registries: registry.json, registry item schema, shadcn build, private registries with auth, and namespaced installs. Use when the user wants to distribute their own components through the shadcn CLI or MCP.
---

# shadcn registries

A registry is just static JSON served over HTTP: one index (`registry.json`)
plus one JSON file per item. Anything the CLI can fetch, it can install, which
also makes your design system installable by agents through the shadcn MCP.

## registry.json

```json
{
  "$schema": "https://ui.shadcn.com/schema/registry.json",
  "name": "acme",
  "homepage": "https://registry.acme.com",
  "items": [
    {
      "name": "hero",
      "type": "registry:block",
      "title": "Hero",
      "description": "Marketing hero with CTA.",
      "registryDependencies": ["button", "@acme/badge"],
      "dependencies": ["motion"],
      "files": [
        { "path": "registry/new-york/hero/hero.tsx", "type": "registry:component" },
        { "path": "registry/new-york/hero/use-hero.ts", "type": "registry:hook" }
      ],
      "cssVars": {
        "light": { "brand": "oklch(0.65 0.2 25)" },
        "dark": { "brand": "oklch(0.75 0.18 25)" }
      }
    }
  ]
}
```

Item types: `registry:block`, `registry:component`, `registry:ui`,
`registry:lib`, `registry:hook`, `registry:page` (needs `target`),
`registry:file` (arbitrary file, needs `target`), `registry:style`,
`registry:theme`, `registry:item`. `registryDependencies` reference other
registry items (bare name = shadcn registry, `@ns/name` = namespaced,
or a full URL); `dependencies` are npm packages.

## Build and serve

```bash
npx shadcn@latest registry validate   # check schema before building
npx shadcn@latest build               # emits public/r/<name>.json per item
```

Serve the output from any static host or a route handler. Consumers install
with `npx shadcn@latest add https://registry.acme.com/r/hero.json` or, better,
configure a namespace.

## Namespaced registries and auth

Consumers declare namespaces in `components.json`:

```json
{
  "registries": {
    "@acme": {
      "url": "https://registry.acme.com/r/{name}.json",
      "headers": { "Authorization": "Bearer ${REGISTRY_TOKEN}" }
    }
  }
}
```

- `{name}` is replaced per item; `${VAR}` reads from the consumer's env
  (put `REGISTRY_TOKEN` in `.env.local`, never commit it).
- Then `npx shadcn@latest add @acme/hero` just works, as do `view`, `search`,
  and MCP-driven installs.
- For private registries, any auth your HTTP layer understands works: bearer
  headers, API-key query params, or a route handler that checks the session.

## Authoring rules

- Registry file paths follow `registry/[style]/[name]/...`; imports inside
  files must use the standard aliases (`@/components/ui/button`,
  `@/lib/utils`) so the CLI can rewrite them for the consumer's project.
- Ship theme tokens through `cssVars` (and extra CSS through `css`) instead of
  telling users to paste snippets; the CLI merges them into globals.css.
- Universal registry items (every file has an explicit `target`) can install
  into any project, no components.json or React required: useful for
  distributing configs, rules, or non-React assets.
- Version by URL if you need stability guarantees (`/r/v2/{name}.json`);
  the protocol itself has no lockfile.
- Test the loop end to end: `validate`, `build`, serve locally, then
  `npx shadcn@latest add http://localhost:3000/r/hero.json` into a scratch app.
