---
name: shadcn-customization
description: Customize shadcn/ui components: cva variants, the cn utility, dark mode, and composition patterns (asChild and render). Use when the user is extending, restyling, or composing shadcn/ui components.
---

# shadcn/ui customization

You own the component source, so customization means editing files in
`components/ui`, not fighting a library API.

## Variants with cva

Components define variants with `class-variance-authority`:

```tsx
import { cva, type VariantProps } from "class-variance-authority"

const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 rounded-md text-sm font-medium transition-colors disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-white hover:bg-destructive/90",
        outline: "border bg-background hover:bg-accent hover:text-accent-foreground",
        ghost: "hover:bg-accent hover:text-accent-foreground",
      },
      size: {
        default: "h-9 px-4 py-2",
        sm: "h-8 rounded-md px-3",
        lg: "h-10 rounded-md px-6",
        icon: "size-9",
      },
    },
    defaultVariants: { variant: "default", size: "default" },
  }
)
```

To add a variant: add a key to the map, done. Type it via
`VariantProps<typeof buttonVariants>`. Export `buttonVariants` so other
components can render link-shaped buttons:
`<Link className={buttonVariants({ variant: "outline" })} />`.

## The cn utility

`cn` (in `lib/utils.ts`) is `twMerge(clsx(...))`: it merges conditional classes
and resolves Tailwind conflicts so caller classes win. Always route
`className` props through it:

```tsx
<div className={cn("flex items-center gap-2", className)} />
```

Without `twMerge`, `"p-2"` from a caller would not override a baked-in `"p-4"`.

## Dark mode

Theme tokens live in CSS variables, so dark mode is only a `.dark` class on
`<html>` overriding `:root` values. In Next.js use next-themes:

```tsx
<ThemeProvider attribute="class" defaultTheme="system" enableSystem>
```

Rules: style with semantic utilities (`bg-card`, `text-muted-foreground`),
reserve `dark:` prefixes for rare structural tweaks, and never pair a raw
palette color with a semantic one (breaks in one of the two modes).

## Composition patterns

- Radix-based components support `asChild`: the component merges its props
  onto your child instead of rendering its own element.
  `<Button asChild><Link href="/login">Login</Link></Button>`
- Base UI based components (the default for new projects) use a `render`
  prop instead: `<Button render={<Link href="/login" />}>Login</Button>`.
  Check the component source to see which primitive it wraps.
- Compound parts stay separate exports (`Card`, `CardHeader`, `CardTitle`,
  `CardContent`): compose in place rather than adding boolean props.
- Current components stamp `data-slot="card-header"` style attributes on each
  part; target them from a parent for contextual styling:
  `[&_[data-slot=card-title]]:text-lg`.

## Practical recipes

- Wrap, do not fork, for app-level defaults: create `components/app-button.tsx`
  that renders `<Button>` with your presets; keep `components/ui` close to
  upstream so `diff` stays useful.
- Global look changes (radius, shadows, fonts) belong in the CSS variables
  (`--radius`, `--font-sans`), not in per-component classes.
- Animations: components use `tw-animate-css` utilities plus Radix/Base state
  attributes (`data-[state=open]:animate-in`, `data-[state=closed]:fade-out-0`).
- For a new primitive not in the registry, copy the closest existing component
  and follow its conventions (forwarded `className`, cva, data-slot, cn).
