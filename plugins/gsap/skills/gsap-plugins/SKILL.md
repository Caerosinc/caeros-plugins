---
name: gsap-plugins
description: The GSAP plugin set, all free since 3.13: SplitText, Draggable, MorphSVG, DrawSVG, Flip, ScrollSmoother, MotionPath and friends, with current import patterns. Use when the user needs text splitting, dragging, SVG morphing, or other plugin effects.
---

# GSAP plugins (all free)

Since GSAP 3.13, every formerly paid Club plugin ships free in the main `gsap`
npm package and on CDNs. No memberships, no separate tarballs. Import from the
package root path and register once:

```js
import { gsap } from "gsap"
import { SplitText } from "gsap/SplitText"
import { Draggable } from "gsap/Draggable"
import { MorphSVGPlugin } from "gsap/MorphSVGPlugin"
gsap.registerPlugin(SplitText, Draggable, MorphSVGPlugin)
```

CDN builds live at `https://cdn.jsdelivr.net/npm/gsap@3/dist/<Plugin>.min.js`.

## SplitText (rewritten in 3.13)

```js
const split = SplitText.create(".headline", {
  type: "chars,words,lines",
  autoSplit: true,          // re-splits on font load / resize
  onSplit(self) {           // animate inside so autoSplit re-runs it
    return gsap.from(self.chars, { y: 20, autoAlpha: 0, stagger: 0.02 })
  },
})
split.revert()               // restore original markup
```

The rewrite cut file size roughly in half, added masking (`mask: "lines"`),
better emoji and non-Latin support, and accessibility defaults (`aria: "auto"`
keeps a screen-reader-friendly label). Split after fonts load or use
`autoSplit`.

## Draggable + Inertia

```js
Draggable.create(".knob", {
  type: "x,y",               // or "rotation", "x", "scroll"
  bounds: ".container",
  inertia: true,             // needs InertiaPlugin registered
  snap: { x: v => Math.round(v / 100) * 100 },
  onDragEnd() { console.log(this.endX) },
})
```

InertiaPlugin (free now) gives momentum flicks and powers `snap` landing.

## SVG plugins

- MorphSVGPlugin: `gsap.to("#circle", { morphSVG: "#hippo" })`; convert
  primitives with `MorphSVGPlugin.convertToPath("circle, rect")`; tune with
  `shapeIndex` and `type: "rotational"`.
- DrawSVGPlugin: stroke reveal, `gsap.from(".line", { drawSVG: 0 })` or
  `drawSVG: "20% 80%"` segments. Requires actual strokes with stroke-width.
- MotionPathPlugin (always was free): `motionPath: { path: "#route",
  align: "#route", autoRotate: true }`.

## Text, scramble, effects

- TextPlugin (free core): `gsap.to(el, { text: "New copy" })`.
- ScrambleTextPlugin: `scrambleText: { text: "DECODED", chars: "01" }`.
- CustomEase / CustomBounce / CustomWiggle: `CustomEase.create("hop",
  "M0,0 C0.2,0 ...")` then `ease: "hop"`; SVG-path-defined curves.
- Physics2DPlugin / PhysicsPropsPlugin: particle bursts
  (`physics2D: { velocity: 300, angle: -60, gravity: 400 }`).

## Layout and scroll helpers

- Flip: FLIP-technique layout animation. `const state = Flip.getState(items)`,
  reparent/reorder/toggle classes in the DOM, then
  `Flip.from(state, { duration: 0.5, absolute: true, stagger: 0.02 })`.
  This is the right tool for animating layout changes core GSAP should not
  tween directly.
- ScrollSmoother: `ScrollSmoother.create({ smooth: 1, effects: true })` with
  the required `#smooth-wrapper > #smooth-content` structure;
  `data-speed` / `data-lag` attributes give per-element parallax.
- Observer: normalized wheel/touch/pointer input without scrolling
  (`Observer.create({ onUp, onDown, tolerance: 10 })`), great for
  fullscreen section paging.
- ScrollToPlugin (free core): `gsap.to(window, { scrollTo: "#section3" })`.

## Gotchas

- Registering a plugin twice is harmless; forgetting to register fails
  silently in production builds (tree shaking drops it), so register in a
  central setup module.
- SSR: plugins touch the DOM at import-register time only via register; still
  guard usage so it runs client-side (see gsap-react).
- Older tutorials mention "Club GreenSock", bonus zips, or trial builds:
  obsolete since 3.13, everything installs from public npm.
