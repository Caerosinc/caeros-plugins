---
name: gsap-core
description: GSAP fundamentals: tweens, timelines, easing, control methods, responsive matchMedia, and performance rules. Use when the user is writing or debugging GSAP animations.
---

# GSAP core

`npm install gsap` (3.x line, currently 3.15). GSAP is fully free, including
formerly premium plugins, since 3.13 (Webflow acquisition). It is free to use
but not open source (no forking into competing products).

```js
import { gsap } from "gsap"
```

## Tweens

```js
gsap.to(".box", { x: 200, rotation: 360, duration: 1, ease: "power2.out" })
gsap.from(".card", { y: 40, autoAlpha: 0, stagger: 0.1 })
gsap.fromTo(el, { scale: 0.8 }, { scale: 1, duration: 0.4 })
gsap.set(el, { transformOrigin: "50% 100%" })   // instant, no animation
```

- Targets: selector strings, elements, refs, arrays. Selector strings tween
  every match with automatic stagger support (`stagger: 0.1` or
  `{ each: 0.1, from: "center" }`).
- Useful vars: `delay`, `repeat` (-1 = infinite), `yoyo`, `repeatDelay`,
  `paused`, `overwrite: "auto"`, callbacks (`onComplete`, `onUpdate`).
- `autoAlpha` = opacity + visibility, better than `opacity` for show/hide.
- Transform shorthands: `x`, `y`, `scale`, `rotation`, `skewX`, `xPercent`.
  Always prefer these over `left/top` (transforms skip layout).

## Timelines

```js
const tl = gsap.timeline({ defaults: { duration: 0.6, ease: "power3.out" } })
tl.to(".hero", { autoAlpha: 1 })
  .from(".title", { y: 30 }, "-=0.3")   // overlap previous by 0.3s
  .from(".cta", { scale: 0.9 }, "<")     // "<" = with previous start
  .addLabel("ready")
  .to(".badge", { rotation: 10 }, "ready+=0.2")
```

Position parameter cheat sheet: absolute time `1.5`, `"<"` start of previous,
`">"` end of previous, `"+=0.5"` gap, `"-=0.5"` overlap, labels. Timelines
nest: build section timelines in functions that return them, then
`master.add(intro()).add(scroll())`.

## Control and easing

- Any tween/timeline: `.play() .pause() .reverse() .restart() .seek(1.2)
  .progress(0.5) .timeScale(2) .kill()`.
- Eases as strings: `"power1"` to `"power4"` (+ `.in/.out/.inOut`), `"back"`,
  `"elastic"`, `"bounce"`, `"expo"`, `"circ"`, `"sine"`, `"steps(5)"`,
  `"none"` (linear, use for scrub). Configurable: `"back.out(1.7)"`,
  `"elastic.out(1, 0.3)"`. Default is `"power1.out"`.
- `gsap.utils`: `mapRange`, `clamp`, `wrap`, `random`, `interpolate`, `toArray`.

## Responsive and accessible

```js
const mm = gsap.matchMedia()
mm.add({
  isMobile: "(max-width: 767px)",
  reduce: "(prefers-reduced-motion: reduce)",
}, (ctx) => {
  const { isMobile, reduce } = ctx.conditions
  if (reduce) return            // skip or simplify animation
  gsap.to(".hero", { x: isMobile ? 50 : 200 })
  return () => {}               // optional extra cleanup
})
```

Everything created inside `mm.add` reverts automatically when the condition
stops matching. Always ship a `prefers-reduced-motion` branch.

## Performance rules

- Animate transforms and opacity; avoid `width/height/top/left/margin`
  (layout thrash). For layout-affecting changes, use the Flip plugin.
- High-frequency updates (cursor followers): use `gsap.quickTo(el, "x")` or
  `gsap.quickSetter` instead of creating tweens per event.
- Batch DOM reads before writes; GSAP already syncs updates on its own ticker
  (`gsap.ticker.add(fn)` for custom loops, not rAF + new tweens).
- `will-change` is handled by GSAP (`force3D: "auto"` promotes during
  animation); do not sprinkle it manually.
- Kill or revert animations you no longer need; in component frameworks use
  `gsap.context()` / `useGSAP` for scoped cleanup (see gsap-react).
