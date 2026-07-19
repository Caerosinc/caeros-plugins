---
name: gsap-scrolltrigger
description: Scroll-driven animation with GSAP ScrollTrigger: triggers, scrub, pinning, snapping, batching, and refresh gotchas. Use when the user wants scroll animations, pinned sections, or parallax.
---

# ScrollTrigger

```js
import { gsap } from "gsap"
import { ScrollTrigger } from "gsap/ScrollTrigger"
gsap.registerPlugin(ScrollTrigger)
```

## Basic trigger

```js
gsap.to(".panel", {
  x: 300,
  scrollTrigger: {
    trigger: ".panel",
    start: "top 80%",     // trigger-top hits viewport 80%
    end: "bottom 20%",
    toggleActions: "play none none reverse",
    markers: true,        // dev only, remove before ship
  },
})
```

- `start`/`end`: `"<triggerPos> <viewportPos>"`, keywords or px or `"+=500"`
  (relative distance). Functions allowed for dynamic values.
- `toggleActions`: four words for onEnter, onLeave, onEnterBack, onLeaveBack
  (`play pause resume reset restart complete reverse none`).
- One-off reveal: `once: true`. Class toggling: `toggleClass: "active"`.
- Standalone (no tween): `ScrollTrigger.create({ trigger, onEnter, onUpdate })`
  with `self.progress` available in callbacks.

## Scrub and pinning

```js
gsap.timeline({
  scrollTrigger: {
    trigger: ".stage",
    start: "top top",
    end: "+=2000",        // 2000px of scroll distance
    scrub: 1,             // true = hard-linked, number = smoothing seconds
    pin: true,
    anticipatePin: 1,     // reduces jump on fast scroll
  },
})
.to(".layer1", { yPercent: -50, ease: "none" })
.to(".layer2", { yPercent: -100, ease: "none" }, 0)
```

- Use `ease: "none"` on scrubbed tweens; scroll position is the easing.
- Pinning gotchas: never apply transforms to a pinned element's ancestors,
  do not pin an element with a CSS `transform` parent, and let ScrollTrigger
  add pin spacing (`pinSpacing: false` only for overlap effects).
  With `pinType: "transform"` for smooth-scroll containers.
- Horizontal fake-scroll pattern: pin a container and tween
  `xPercent: -100 * (sections - 1)` with scrub; use
  `containerAnimation` for triggers inside that moving strip.

## Snap, batch, events

- `snap: 1 / (sections - 1)` or
  `snap: { snapTo: "labels", duration: 0.3, ease: "power1.inOut" }`.
- Many similar reveals: `ScrollTrigger.batch(".card", { onEnter: b =>
  gsap.from(b, { y: 40, autoAlpha: 0, stagger: 0.1 }) })` (one observer,
  interleaved staggers).
- Global: `ScrollTrigger.refresh()` recalculates all start/end positions;
  call after content loads (images, fonts, accordions). Use
  `invalidateOnRefresh: true` so function-based values recompute.

## Lifecycle and integration

- Creation order matters for refresh math: create triggers in document order
  or set `refreshPriority`. Sort issues usually mean triggers were created
  out of order.
- Resize is handled automatically; dynamic layout changes need `refresh()`.
- In React, create inside `useGSAP` so triggers are reverted on unmount
  (see gsap-react); orphaned ScrollTriggers are the top cause of ghost pins.
- Responsive variants: wrap trigger creation in `gsap.matchMedia()`; triggers
  revert when the media condition flips.
- Smooth scrolling: prefer ScrollSmoother (same team, now free,
  `ScrollSmoother.create({ smooth: 1, effects: true })`) over third-party
  smooth-scroll libraries; for those, wire `ScrollTrigger.scrollerProxy()`.
- Debug: `markers: true`, and remember start positions are computed at
  refresh time, not live per frame.
