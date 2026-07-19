---
name: gsap-react
description: GSAP in React with the official useGSAP hook: scoping, cleanup, contextSafe, dependencies, ScrollTrigger integration, and SSR. Use when the user animates React (or Next.js) components with GSAP.
---

# GSAP + React

```bash
npm install gsap @gsap/react
```

`useGSAP` (from `@gsap/react`, currently 2.x) is the official hook: a drop-in
replacement for `useLayoutEffect`/`useEffect` that collects every GSAP object
created inside it (tweens, timelines, ScrollTriggers, Draggables, SplitText)
and reverts them automatically on unmount. It is React 18/19 Strict Mode safe:
no double-playing animations in dev.

```tsx
"use client"                       // required in Next.js App Router files
import { useRef } from "react"
import { gsap } from "gsap"
import { useGSAP } from "@gsap/react"

gsap.registerPlugin(useGSAP)

export function Hero() {
  const container = useRef<HTMLDivElement>(null)

  useGSAP(() => {
    // selector text is scoped to container: only matches inside it
    gsap.from(".card", { y: 40, autoAlpha: 0, stagger: 0.1 })
  }, { scope: container })

  return <div ref={container}>{/* .card elements */}</div>
}
```

## Config options

```tsx
useGSAP(() => { ... }, { scope: container, dependencies: [endX], revertOnUpdate: true })
```

- `scope`: ref; all selector text inside resolves within it. Prefer this over
  a ref per animated element.
- `dependencies`: like an effect dep array (default `[]`, runs once on mount).
- `revertOnUpdate: true`: revert and rebuild when deps change (default only
  reverts on unmount).

## Event handlers: contextSafe

Animations created outside the hook body (clicks, timers) are not collected
unless wrapped:

```tsx
const { contextSafe } = useGSAP({ scope: container })
const onClick = contextSafe(() => {
  gsap.to(".flair", { rotation: "+=360" })
})
```

Without `contextSafe`, handler-created tweens leak past unmount and, with
ScrollTrigger, leave ghost triggers.

## ScrollTrigger and plugins

```tsx
import { ScrollTrigger } from "gsap/ScrollTrigger"
gsap.registerPlugin(ScrollTrigger, useGSAP)

useGSAP(() => {
  gsap.timeline({
    scrollTrigger: { trigger: container.current, start: "top top", end: "+=1500", scrub: 1, pin: true },
  }).to(".panel", { xPercent: -200, ease: "none" })
}, { scope: container })
```

Triggers made inside `useGSAP` are reverted on unmount, which is exactly what
route changes need. After async content changes heights, call
`ScrollTrigger.refresh()`.

## SSR and Next.js

- The hook is isomorphic (falls back to `useEffect` server-side), so importing
  it is SSR-safe, but any component calling it needs `"use client"` in the
  App Router.
- Avoid hydration flashes for from-style intros: hide with CSS
  (`opacity: 0` class) and animate `to` visible, or set initial styles inline;
  do not rely on JS running before paint.
- Register plugins once in a shared module, not per component.

## Patterns and gotchas

- Never create tweens during render; only inside `useGSAP` or contextSafe
  callbacks.
- Communicate between components by passing timelines through props/context
  (build the timeline in the parent `useGSAP`, `tl.add()` child pieces), or
  keep it simple with scoped selectors.
- Reusable interactions: wrap in custom hooks that call `useGSAP` internally.
- Lists that reorder: rebuild with `revertOnUpdate` plus a dependency on the
  list, or use the Flip plugin for smooth reflow.
- If an animation runs twice in dev, you bypassed the hook (raw `useEffect`
  without cleanup); move it into `useGSAP`.
