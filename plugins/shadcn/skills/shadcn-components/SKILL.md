---
name: shadcn-components
description: Add and manage shadcn/ui components with the CLI, configure registries, and theme with CSS variables. Use when the user is installing shadcn/ui components, wiring up components.json, or adjusting a shadcn theme.
---

# shadcn/ui components

shadcn/ui is not an npm component library: the CLI copies component source into
your repo (usually `components/ui`) and you own the code from then on. The CLI
is currently v4; new projects default to Base UI primitives (Radix remains
supported and maintained, existing Radix projects keep working).

## Core CLI workflow

```bash
npx shadcn@latest init            # creates components.json, sets up Tailwind + CSS vars
npx shadcn@latest add button card dialog form
npx shadcn@latest add @acme/hero  # namespaced item from a configured registry
npx shadcn@latest view button     # inspect an item before installing
npx shadcn@latest search sidebar  # search configured registries
npx shadcn@latest diff            # check installed items against upstream
```

- `init` asks for style, base color, and CSS variables; answers land in
  `components.json`. Re-run `add` any time, it resolves dependencies
  (e.g. `form` pulls `label`, `button`, react-hook-form, zod).
- Components are plain files: edit them directly, that is the intended model.
- The official MCP server (`npx shadcn@latest mcp`) exposes browse, search,
  view, and install over your configured registries; prefer it for discovery.

## components.json essentials

```json
{
  "style": "new-york",
  "tailwind": { "css": "app/globals.css", "cssVariables": true },
  "aliases": { "components": "@/components", "utils": "@/lib/utils", "ui": "@/components/ui" },
  "registries": {
    "@acme": { "url": "https://registry.acme.com/{name}.json" }
  }
}
```

Aliases must match your `tsconfig.json` paths. The `registries` map enables
`@acme/item` installs, including private registries with auth headers (see the
shadcn-registry skill).

## Theming with CSS variables

Colors are semantic pairs of variables consumed as Tailwind utilities:

```css
:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);
  --radius: 0.625rem;
}
.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
}
```

With Tailwind v4 the variables are mapped in `@theme inline` (no
tailwind.config needed): `--color-background: var(--background);` then use
`bg-background text-foreground`. Convention: every `--x` has an
`--x-foreground` for content rendered on top of it. Other pairs: `card`,
`popover`, `secondary`, `muted`, `accent`, `destructive`, plus `border`,
`input`, `ring`, `chart-1..5`, and the `sidebar-*` group. Dark mode is just the
`.dark` class overriding variables (pair with next-themes in Next.js).

## Component highlights worth knowing

- Layout and structure: `sidebar` (composable, collapsible), `resizable`,
  `scroll-area`, `breadcrumb`.
- Data: `chart` (Recharts wrappers themed via `chart-1..5`), `data-table`
  pattern built on TanStack Table, `calendar`, `carousel`.
- Forms: `field`, `input-group`, `button-group`, `combobox`, `input-otp`,
  `date-picker` (composed from `calendar` + `popover`).
- Feedback and misc: `sonner` (toasts), `spinner`, `empty`, `kbd`, `item`,
  `skeleton`, `drawer` (Vaul), `command` (cmdk palette).

## Gotchas

- Never hardcode colors in components; use semantic utilities so themes and
  dark mode keep working.
- `add` overwrites on confirmation: commit before re-adding an edited
  component, then review the diff.
- Monorepos: run the CLI from the app workspace so aliases resolve; the CLI
  understands workspace setups but paths in `components.json` are per-app.
- The docs and registry evolve fast; when exact props matter, `view` the item
  or check https://ui.shadcn.com/docs rather than trusting memory.
