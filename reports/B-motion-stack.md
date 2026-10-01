# The Web Animation Stack — September 2026 Reference

Dense, verified reference for the tools award-winning sites actually use. Every version number and API shape below was checked against official docs/changelogs in September 2026 (not recalled from training). Where a source disagreed with itself or coverage was thin, that's flagged explicitly rather than smoothed over.

---

## 1. GSAP

### 1.1 Version & licensing — the Webflow story

- **Current version: 3.15.0** (released ~April 2026). Timeline: 3.13 → April 30, 2025 (the "everything free" release), 3.14 → Dec 8, 2025, 3.15 → April 2026 (adds `easeReverse`, deprecates `yoyoEase`). [gsap.com/blog archive](https://gsap.com/blog/archive/), [cdnjs](https://cdnjs.com/libraries/gsap)
- **Webflow acquired GreenSock on October 15, 2024.** The GSAP team now works full-time *at* Webflow; the footer literally reads "A Webflow Product." [webflow.com/blog/gsap-becomes-free](https://webflow.com/blog/gsap-becomes-free), [gsap.com/pricing](https://gsap.com/pricing/)
- **As of GSAP 3.13 (April 30, 2025), GSAP is 100% free for everyone, including every plugin that used to require a Club GreenSock membership** — ScrollTrigger, ScrollSmoother, SplitText, MorphSVG, DrawSVG, Physics2D, InertiaPlugin, ScrambleText, MotionPath, Flip, Observer, CustomEase/CustomBounce/CustomWiggle, GSDevTools, Draggable. **Club GreenSock no longer exists as a paid tier.** The old private npm registry (`npm.greensock.com`) is deprecated — everything is on public npm now. [gsap.com/pricing](https://gsap.com/pricing/), [community.webflow.com](https://community.webflow.com/updates/post/webflow-makes-gsap-100-free-fugRRt8eUL1we1k)
- The standard license covers commercial use broadly; there is no more "Business Green" paid license gate mentioned anywhere on the current pricing page — it was retired along with Club GreenSock.
- **SplitText was fully rewritten** for the free release: ~50% smaller, and accessibility (proper `aria` handling for screen readers) is now baked in rather than bolted on. [webflow.com/blog/gsap-splittext-rewrite](https://webflow.com/blog/gsap-splittext-rewrite)

**CDN (verified paths, pin the version):**
```html
<!-- jsDelivr (recommended) -->
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/gsap.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/ScrollTrigger.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/ScrollSmoother.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/SplitText.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/Flip.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/Observer.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/DrawSVGPlugin.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/MorphSVGPlugin.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/MotionPathPlugin.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/Physics2DPlugin.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/InertiaPlugin.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/CustomEase.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/CustomBounce.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/CustomWiggle.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/ScrambleTextPlugin.min.js"></script>

<!-- cdnjs alternative -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.15.0/gsap.min.js"></script>
```
```bash
npm install gsap          # core + ALL plugins ship in one free package now
npm install @gsap/react   # useGSAP() hook for React
```
Import in a bundler: `import { gsap } from "gsap"; import { ScrollTrigger } from "gsap/ScrollTrigger"; gsap.registerPlugin(ScrollTrigger);`

### 1.2 Core API

**The four tween constructors:**
```js
gsap.to(".box", { x: 300, duration: 1 });                       // animates TO these values
gsap.from(".box", { opacity: 0, y: 50, duration: 1 });           // animates FROM these values to current
gsap.fromTo(".box", { opacity: 0 }, { opacity: 1, duration: 1 }); // explicit start AND end
gsap.set(".box", { x: 300 });                                    // instant, no animation
```
- Default ease when none specified: **`"power1.out"`**.
- Common special properties: `duration`, `delay`, `ease`, `easeReverse` (3.15+, replaces deprecated `yoyoEase`), `stagger`, `repeat`, `repeatDelay`, `yoyo`, `paused`, `overwrite`, `onComplete`/`onStart`/`onUpdate` (+ `Params` variants), `onInterrupt`, `id`, `data`, `keyframes`.

**Timelines** — the backbone of choreography:
```js
const tl = gsap.timeline({ repeat: -1, repeatDelay: 1, defaults: { duration: 0.8, ease: "power2.out" } });

tl.to(".hero-title", { y: 0, opacity: 1 })
  .to(".hero-sub", { y: 0, opacity: 1 }, "-=0.5")   // overlap by 0.5s
  .to(".hero-cta", { scale: 1 }, "<")                // start at same time as previous tween started
  .addLabel("done")
  .to(".bg", { opacity: 0.4 }, "done+=0.2");         // 0.2s after the label
```
Position-parameter cheat sheet: absolute seconds (`3`), relative (`"+=1"`/`"-=1"`), label (`"myLabel"`), label offset (`"myLabel+=2"`), `"<"`/`"<1"` (start of previous), `">"`/`">1"` (end of previous).

**`defaults` on a timeline** cascade to every child tween unless overridden — the standard way to avoid repeating `duration`/`ease` on every line.

**Stagger objects** — real choreography, not just a flat delay:
```js
gsap.to(".card", {
  opacity: 1, y: 0,
  stagger: {
    each: 0.08,        // 80ms between each start (use instead of `amount` when item count varies)
    from: "center",    // or "start"/"end"/an index/[x,y] normalized coords for grids
    grid: "auto",      // or [rows, cols] — makes stagger 2D, radiating from `from`
    axis: "y",         // restrict grid distance calc to one axis
    ease: "power2.in"  // eases the START-TIME distribution, NOT each tween's own motion curve
  }
});
// amount: total seconds split across ALL elements regardless of count (use when you need a fixed total duration)
gsap.to(".item", { opacity: 1, stagger: { amount: 1.2 } });
```
[gsap.com/resources/getting-started/Staggers](https://gsap.com/resources/getting-started/Staggers/)

**`gsap.matchMedia()`** — the correct way to do responsive animation logic (internally creates its own `gsap.context()`, so don't wrap it in one):
```js
let mm = gsap.matchMedia();

mm.add({
  isDesktop: "(min-width: 800px)",
  isMobile: "(max-width: 799px)",
  reduceMotion: "(prefers-reduced-motion: reduce)"
}, (context) => {
  let { isDesktop, reduceMotion } = context.conditions;
  gsap.to(".box", { rotation: isDesktop ? 360 : 180, duration: reduceMotion ? 0 : 2 });
  return () => { /* optional cleanup when this condition stops matching */ };
});
// mm.revert() reverts everything mm created, across all conditions
```

**`gsap.context()`** — still exists and works, but **in React, use the `useGSAP()` hook instead** (see §1.4). Context is for: (a) bulk revert/kill of everything created inside the function, (b) scoping selector text to a ref/element.

**`gsap.quickTo()`** — the answer for high-frequency updates (mousemove, drag, scroll-linked cursor followers). Skips unit conversion/relative-value/plugin parsing overhead that a full `gsap.to()` pays on every call:
```js
let xTo = gsap.quickTo("#cursor", "x", { duration: 0.4, ease: "power3" });
let yTo = gsap.quickTo("#cursor", "y", { duration: 0.4, ease: "power3" });
window.addEventListener("mousemove", (e) => { xTo(e.clientX); yTo(e.clientY); });
```

**Ticker** — GSAP's single shared `requestAnimationFrame` loop; hook into it instead of spinning your own rAF:
```js
function tick(time, deltaTime, frame) { /* time: elapsed sec, deltaTime: ms since last tick */ }
gsap.ticker.add(tick);
gsap.ticker.remove(tick);
gsap.ticker.fps(30);              // throttle
gsap.ticker.lagSmoothing(0);      // disable lag compensation — needed when driving Lenis (§3.1)
```

### 1.3 ScrollTrigger

Config essentials: `trigger`, `start`, `end`, `scrub`, `pin`, `pinSpacing`, `snap`, `toggleActions`, `markers` (dev only).

**start/end syntax** (`"<trigger-position> <scroller-position>"`): keywords `top`/`center`/`bottom`/`left`/`right`, percentages (`"80%"`), pixels, relative (`"+=300px"`), `"max"`, and `clamp(...)` to prevent runaway values when content is short.

```js
gsap.registerPlugin(ScrollTrigger);

gsap.to(".panel", {
  x: -500,
  scrollTrigger: {
    trigger: ".panel",
    start: "top 80%",
    end: "bottom 20%",
    scrub: 1,             // true = 1:1, or a number = seconds to catch up (smoothed scrub)
    pin: true,
    pinSpacing: true,     // false/"margin" to change how space is reserved for the pin
    markers: false
  }
});
```

**`toggleActions`** — four space-separated keywords for `onEnter onLeave onEnterBack onLeaveBack`, each one of `play|pause|resume|reverse|restart|reset|complete|none`:
```js
scrollTrigger: { trigger: ".box", toggleActions: "play pause resume reverse" }
```

**`ScrollTrigger.batch()`** — coordinated staggered reveal for many elements without one ScrollTrigger per element:
```js
ScrollTrigger.batch(".card", {
  interval: 0.1,
  batchMax: 4,
  onEnter: batch => gsap.to(batch, { opacity: 1, y: 0, stagger: 0.15, overwrite: true }),
  onLeave: batch => gsap.set(batch, { opacity: 0, y: 100, overwrite: true }),
  onEnterBack: batch => gsap.to(batch, { opacity: 1, y: 0, stagger: 0.15, overwrite: true }),
  onLeaveBack: batch => gsap.set(batch, { opacity: 0, y: -100, overwrite: true }),
  start: "top 85%"
});
```

**Snapping**:
```js
scrollTrigger: {
  snap: {
    snapTo: 1 / (sections.length - 1),
    duration: { min: 0.2, max: 0.6 },
    ease: "power1.inOut",
    inertia: false,          // account for scroll momentum when deciding snap target
    directional: true        // biases toward the direction you were scrolling
  }
}
```

**`ScrollTrigger.refresh()`** — recompute all start/end positions (call after dynamic content/image loads/font swaps resize the page). `ScrollTrigger.getAll()` / `ScrollTrigger.isScrolling()` are the other two you'll reach for.

**Horizontal scroll** (the classic "pin a wide track, drive it with vertical scroll" recipe):
```js
const track = document.querySelector(".h-track");
gsap.to(track, {
  x: () => -(track.scrollWidth - window.innerWidth) + "px",
  ease: "none",
  scrollTrigger: {
    trigger: ".h-wrapper",
    start: "top top",
    end: () => "+=" + (track.scrollWidth - window.innerWidth),
    scrub: 1,
    pin: true,
    invalidateOnRefresh: true
  }
});
```

### 1.4 SplitText — the rewritten version (3.13+)

```js
gsap.registerPlugin(SplitText);

SplitText.create(".headline", {
  type: "words, chars",     // any comma-combo of "chars" | "words" | "lines"
  mask: "lines",             // wraps that unit in an overflow-clip container — the classic "line reveal" mask
  autoSplit: true,           // re-splits automatically on resize / webfont load (replaces manual resize listeners)
  smartWrap: true,           // prevents orphan characters at line-wrap points
  aria: "auto",              // "auto" | "hidden" | "none" — screen-reader handling is now built in
  onSplit(self) {             // fires on every (re)split, including auto-resplits — return the tween for cleanup
    return gsap.from(self.chars, {
      yPercent: 110,
      opacity: 0,
      stagger: 0.03,
      duration: 0.6,
      ease: "power3.out"
    });
  }
});
```
- Instance arrays: `self.chars`, `self.words`, `self.lines`, `self.masks` (present when `mask` is set).
- `mask` accepts only **one** of `"lines"|"words"|"chars"` at a time — it wraps that unit in an extra `visibility:clip` element for cheap, GPU-friendly reveal-through-a-slot animation without you hand-rolling `overflow:hidden` wrappers.
- Other options: `linesClass`/`wordsClass`/`charsClass` (append `"++"` to auto-increment), `propIndex` (adds `--word`/`--char` CSS custom properties per element), `reduceWhiteSpace` (default `true`), `wordDelimiter`, `prepareText`, `ignore`, `deepSlice` (default `true`, handles nesting across lines correctly), `tag` (default `"div"`).
- Methods: `.split()` (manual re-split), `.revert()` (restore original markup — **always revert on unmount/breakpoint change** to avoid duplicate splits).
- [gsap.com/docs/v3/Plugins/SplitText](https://gsap.com/docs/v3/Plugins/SplitText/)

### 1.5 ScrollSmoother

Wraps native scrolling in an easing layer — position: sticky, anchor links and native scrollbar all keep working, unlike scroll-hijacking libraries.
```html
<body>
  <div id="smooth-wrapper">
    <div id="smooth-content">
      <!-- all your normal page content -->
      <div data-speed="0.5">moves slower — parallax back layer</div>
      <div data-speed="1.5">moves faster — parallax front layer</div>
      <div data-lag="0.5">lags behind scroll by up to 0.5s</div>
    </div>
  </div>
</body>
```
```js
gsap.registerPlugin(ScrollTrigger, ScrollSmoother);

ScrollSmoother.create({
  wrapper: "#smooth-wrapper",
  content: "#smooth-content",
  smooth: 1,          // seconds to "catch up" to the target scroll position
  effects: true,       // turns on data-speed / data-lag parsing
  smoothTouch: 0.1,    // light smoothing on touch (default: none, native feel)
  normalizeScroll: true // forces scroll onto the JS thread — fixes mobile browser-chrome-jump bugs
});
```
[gsap.com/docs/v3/Plugins/ScrollSmoother](https://gsap.com/docs/v3/Plugins/ScrollSmoother/) — **ScrollSmoother vs. Lenis**: see §3.4.

### 1.6 Flip

FLIP (First-Last-Invert-Play) for layout transitions — animate a DOM change (grid→list, modal expand, reorder) as if it tweened smoothly, when really it's teleporting instantly and faking the in-between:
```js
gsap.registerPlugin(Flip);

const state = Flip.getState(".card");     // 1. record BEFORE
container.classList.toggle("expanded");    // 2. make the instant DOM/class change
Flip.from(state, {                         // 3. animate FROM the recorded state to the new layout
  duration: 0.6,
  ease: "power1.inOut",
  absolute: true,                          // position:absolute during the flip so siblings reflow correctly
  stagger: 0.03,
  onEnter: els => gsap.fromTo(els, { opacity: 0 }, { opacity: 1 }),
  onLeave: els => gsap.to(els, { opacity: 0 })
});
```

### 1.7 Observer

Unified input listener (wheel/touch/pointer/scroll) — the backbone of most custom slide-decks and section-snapping without ScrollTrigger's DOM-position assumptions:
```js
gsap.registerPlugin(Observer);

Observer.create({
  target: window,
  type: "wheel,touch",
  wheelSpeed: 1,
  tolerance: 10,
  onUp: () => goToSlide(current - 1),
  onDown: () => goToSlide(current + 1),
  preventDefault: true
});
```

### 1.8 DrawSVG, MorphSVG, MotionPath, Physics2D, InertiaPlugin

```js
// DrawSVGPlugin — progressive stroke reveal
gsap.from("#path", { drawSVG: "0%", duration: 1.5 });      // 0% → 100% (full reveal)
gsap.to("#path", { drawSVG: "20% 80% live", duration: 1 }); // "live" recalculates each frame — for responsive paths

// MorphSVGPlugin — shape A becomes shape B
gsap.to("#diamond", {
  morphSVG: { shape: "#lightning", type: "rotational", shapeIndex: 2 },
  duration: 1.2, ease: "power2.inOut"
});

// MotionPathPlugin — move an element along an SVG path
gsap.to("#dot", {
  motionPath: { path: "#curve", align: "#curve", alignOrigin: [0.5, 0.5], autoRotate: true },
  duration: 4, ease: "power1.inOut"
});

// Physics2DPlugin — velocity/angle/gravity-driven motion (easing is ignored; it's simulated)
gsap.to("#confetti", { physics2D: { velocity: 300, angle: -60, gravity: 400 }, duration: 2 });

// InertiaPlugin — natural momentum deceleration (drag-and-throw)
gsap.registerPlugin(InertiaPlugin);
InertiaPlugin.track(el, "x,y");
gsap.to(el, { inertia: { x: "auto", y: "auto", resistance: 200 } }); // reads tracked velocity automatically
```

### 1.9 CustomEase, CustomBounce, CustomWiggle, ScrambleText

```js
gsap.registerPlugin(CustomEase, CustomBounce, CustomWiggle, ScrambleTextPlugin);

CustomEase.create("myEase", "M0,0 C0.126,0.382 0.282,0.674 0.44,0.822 0.632,1.002 0.818,1 1,1");
gsap.to(".el", { x: 300, ease: "myEase" });

CustomBounce.create("myBounce", { strength: 0.6, squash: 3 });
CustomWiggle.create("myWiggle", { wiggles: 6, type: "easeOut" });

gsap.to(".text", {
  duration: 1.5,
  scrambleText: { text: "DECODED", chars: "upperCase", revealDelay: 0.4, speed: 0.4 }
});
```
Built-in ease families (all with `.in`/`.out`/`.inOut`): `power1`–`power4`, `back`, `elastic`, `bounce`, `circ`, `expo`, `sine`, plus `steps()`, `rough()`, `slow()`. GSAP's free visual [ease picker](https://gsap.com/docs/v3/Eases) lets you drag control points and copies the exact string.

### 1.10 React: `useGSAP()` over `gsap.context()`

```bash
npm install @gsap/react
```
```jsx
import { useGSAP } from "@gsap/react";
import { gsap } from "gsap";

function Hero() {
  const container = useRef(null);

  useGSAP(() => {
    // selectors here are auto-scoped to `container`
    gsap.from(".title", { opacity: 0, y: 40, duration: 1 });
  }, { scope: container }); // auto-reverted on unmount, handles StrictMode double-invoke correctly

  return <div ref={container}><h1 className="title">Hello</h1></div>;
}
```
`useGSAP()` is a drop-in replacement for `useEffect`/`useLayoutEffect` that wraps `gsap.context()` for you — this is now the documented, recommended pattern; hand-rolling `useEffect` + `gsap.context()` + manual `.revert()` is the old way. [gsap.com/resources/React](https://gsap.com/resources/React/)

### 1.11 Ten copy-pasteable GSAP recipes

**1. Fade-up on scroll (the single most common effect on the web):**
```js
gsap.utils.toArray(".reveal").forEach(el => {
  gsap.from(el, {
    opacity: 0, y: 40, duration: 0.8, ease: "power3.out",
    scrollTrigger: { trigger: el, start: "top 85%", toggleActions: "play none none reverse" }
  });
});
```

**2. SplitText line-mask headline reveal:**
```js
SplitText.create("h1", {
  type: "lines", mask: "lines",
  onSplit: self => gsap.from(self.lines, { yPercent: 110, duration: 0.9, stagger: 0.08, ease: "power4.out" })
});
```

**3. Pinned scrub-driven hero:**
```js
gsap.timeline({ scrollTrigger: { trigger: ".hero", start: "top top", end: "+=1000", scrub: 1, pin: true } })
  .to(".hero img", { scale: 1.3, ease: "none" })
  .to(".hero h1", { opacity: 0, y: -100, ease: "none" }, 0);
```

**4. Sticky horizontal gallery** — see §1.3.

**5. Magnetic cursor follower with `quickTo`** — see §1.2.

**6. Staggered card grid entrance:**
```js
gsap.from(".card", {
  opacity: 0, y: 60, duration: 0.7, ease: "power2.out",
  stagger: { each: 0.08, from: "start", grid: "auto" },
  scrollTrigger: { trigger: ".grid", start: "top 80%" }
});
```

**7. FLIP-powered filter/sort re-layout** — see §1.6.

**8. Batch reveal for long lists (perf-safe vs. one ScrollTrigger per item)** — see §1.3 `ScrollTrigger.batch`.

**9. Number counter tied to scroll progress:**
```js
let obj = { val: 0 };
gsap.to(obj, {
  val: 1000, duration: 2, ease: "power1.out",
  onUpdate: () => counterEl.textContent = Math.round(obj.val).toLocaleString(),
  scrollTrigger: { trigger: counterEl, start: "top 80%" }
});
```

**10. SVG logo draw-in on load:**
```js
gsap.from("#logo path", { drawSVG: "0%", duration: 1.4, stagger: 0.1, ease: "power2.inOut" });
```

---

## 2. Motion (motion.dev — formerly Framer Motion)

### 2.1 Version & package split

- **Current version: 13.3.0** (September 14, 2026) — added "Hooks for Motion Editor," `useSpring`/`springValue` retargeting 80% faster, `animate` 10% smaller & 20% faster startup, 10–60% less render time, 10% faster frame scheduling. 13.2.0 (Sept 3, 2026) introduced `animate.addEffect()` and a `motion/three` effect for Three.js objects. [motion.dev/changelog](https://motion.dev/changelog)
- **Renamed from Framer Motion to Motion in February 2025.** `framer-motion` still exists on npm for back-compat but the maintained package is `motion`; import path is **`motion/react`**, not `framer-motion`. [npmjs.com/package/motion](https://www.npmjs.com/package/motion)
```bash
npm install motion
```
```js
import { motion, AnimatePresence } from "motion/react";           // React
import * as motion from "motion/react-client";                     // React Server Components — no 'use client' needed just to import; components using interactivity/hooks still need it
import { animate, scroll, inView } from "motion";                  // vanilla JS/TS, framework-agnostic
```
- **Hybrid engine**: runs on the Web Animations API natively in-browser where possible, JS fallback otherwise — targets 120fps, GPU-accelerated. [motion.dev](https://motion.dev/)

### 2.2 Core declarative API

```jsx
<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  exit={{ opacity: 0, y: -20 }}
  transition={{ duration: 0.4, ease: [0.16, 1, 0.3, 1] }}
  whileHover={{ scale: 1.05 }}
  whileTap={{ scale: 0.97 }}
/>
```

**`AnimatePresence`** for exit animations (React can't otherwise animate something it's about to unmount):
```jsx
<AnimatePresence mode="wait">   {/* "sync" (default) | "wait" | "popLayout" */}
  {isVisible && (
    <motion.div key="panel" initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }} />
  )}
</AnimatePresence>
```
- `mode="wait"`: next element waits for the exiting one to finish — good for one-at-a-time content swaps.
- `mode="popLayout"`: exiting element is pulled out of layout flow immediately so siblings reflow live; pairs with `layout`. Any custom component that's a direct child must `forwardRef` to the DOM node.

### 2.3 Springs

```jsx
// physics params
<motion.div animate={{ x: 100 }} transition={{ type: "spring", stiffness: 300, damping: 20, mass: 1 }} />
// duration+bounce (often more intuitive): duration = settle time, bounce 0–1 = overshoot amount
<motion.div animate={{ x: 100 }} transition={{ type: "spring", duration: 0.5, bounce: 0.25 }} />
```
Defaults: `stiffness: 100`, `damping: 10`, `mass: 1`. [motion.dev/docs/spring](https://motion.dev/docs/spring)

### 2.4 Scroll: linked vs. triggered

```jsx
// SCROLL-LINKED — value tracks scroll position continuously (progress bars, parallax)
import { useScroll, useTransform, motion } from "motion/react";
const { scrollYProgress } = useScroll();
const scale = useTransform(scrollYProgress, [0, 1], [1, 1.5]);
return <motion.div style={{ scaleX: scrollYProgress, originX: 0 }} />; // e.g. a top progress bar

// SCROLL-TRIGGERED — fires once (or toggles) when element enters view
<motion.div initial={{ opacity: 0 }} whileInView={{ opacity: 1 }} viewport={{ once: true, amount: 0.3 }} />
```
Motion's scroll-linked animations run on the browser's native `ScrollTimeline` when available (hardware-accelerated, same mechanism as CSS `animation-timeline: scroll()` — see §4.1), falling back to JS. Vanilla equivalents: `scroll(callback, options)` and `inView(selector, callback)` from the `motion` package (no React needed).

### 2.5 Layout animations & `AnimatePresence`

```jsx
<motion.div layout />                       // auto-animates any layout change from a React re-render, via transform only (no width/height thrash)
<motion.div layoutId="underline" />          // shared-element transitions — matching layoutId across renders morphs between them (tab underlines, card→modal)
<LayoutGroup>                                // syncs layout animation timing across components that don't re-render together (e.g. sibling accordions)
  <Accordion /><Accordion />
</LayoutGroup>
```

### 2.6 GSAP vs. Motion — the actual decision rule

- **Pick Motion** when animation is driven by React state/props — enter/exit, layout reflow, gestures (hover/tap/drag), and you want the animation to understand the component lifecycle. Bundle ~30KB gzip.
- **Pick GSAP** when you need imperative, millisecond-precise, multi-element timeline choreography — scroll-driven hero sequences, SVG morphing/drawing, text splitting, physics-feel easing, dozens of elements sequenced together. Framework-agnostic; core ~27KB, plugins add more. This is what almost every Awwwards/FWA scroll-story site is built on.
- **They coexist fine on one project**: Motion for the React UI layer (modals, list reorders, page-level enter/exit), GSAP for the one or two scroll-driven signature sequences. [annnimate.com/compare/gsap-vs-motion](https://annnimate.com/compare/gsap-vs-motion), [hontran.dev/blog/gsap-vs-framer-motion](https://www.hontran.dev/blog/gsap-vs-framer-motion)

---

## 3. Smooth Scroll

### 3.1 Lenis

- Moved from `lenis.darkroom.engineering` to **lenis.dev** (old URL 301-redirects). Maintained by darkroom.engineering. **Current version: 1.3.26.** [github.com/darkroomengineering/lenis](https://github.com/darkroomengineering/lenis)
```bash
npm i lenis
```
```html
<script src="https://unpkg.com/lenis@1.3.26/dist/lenis.min.js"></script>
<link rel="stylesheet" href="https://unpkg.com/lenis@1.3.26/dist/lenis.css">
```
**Key options** (defaults shown): `duration: 1.2`, `easing` (custom exponential-out by default), `orientation: "vertical"`, `gestureOrientation: "vertical"`, `smoothWheel: true`, `wheelMultiplier: 1`, `touchMultiplier: 1`, `syncTouch: false`, `infinite: false`, `lerp: 0.1`.

**GSAP `ScrollTrigger` integration** (the canonical pattern — Lenis drives the scroll, GSAP's ticker drives Lenis, so there's exactly one rAF loop in the whole page):
```js
const lenis = new Lenis();
lenis.on("scroll", ScrollTrigger.update);
gsap.ticker.add((time) => lenis.raf(time * 1000));
gsap.ticker.lagSmoothing(0); // must disable — GSAP's own lag compensation fights Lenis's
```
Or the simpler self-contained loop when GSAP isn't involved: `new Lenis({ autoRaf: true })`.

**Nested/prevented scroll areas**:
```html
<div data-lenis-prevent>native-scrolling modal or dropdown content</div>
```
Also available: `data-lenis-prevent-wheel`, `data-lenis-prevent-touch`, `data-lenis-prevent-vertical/horizontal`, or a JS predicate: `new Lenis({ prevent: (node) => node.id === "modal" })`.

**Anchor links**: `new Lenis({ anchors: { offset: 100, onComplete: () => {} } })`, or programmatically `lenis.scrollTo(".target" | element | pixelValue)`.

**Accessibility / arguments against smooth scroll**: Lenis rides on *native* scroll (unlike old scroll-hijacking libs), so `position: sticky`, anchor links, find-in-page and keyboard scrolling keep working — this is the main reason it displaced Locomotive v4/older approaches. Known limitations to design around: no native CSS scroll-snap support (needs the separate `lenis/snap` addon), Safari performance caps around 60fps regardless of display refresh rate, scrolling stops over `<iframe>`s, and nested scroll containers need explicit setup. **Always wrap the smoothing in a `prefers-reduced-motion` check** — smoothing is a comfort layer, not a requirement, and vestibular-sensitive users should get native instant scroll (see §8).

### 3.2 ScrollSmoother vs. Lenis

| | ScrollSmoother | Lenis |
|---|---|---|
| Cost | Free (GSAP 3.13+) | Free/OSS |
| Coupling | GSAP-only, deep ScrollTrigger integration (`data-speed`/`data-lag` built in) | Framework-agnostic; needs manual ScrollTrigger proxy wiring |
| Scrollbar | Native | Native |
| Best for | Projects already all-in on GSAP wanting one less moving part | Projects mixing GSAP + WebGL (Three.js/OGL) + React, or not using GSAP at all |

Functionally they've converged (both ride native scroll, both smooth via lerp/easing) — pick based on what else is in your stack.

### 3.3 Locomotive Scroll v5

- **v5.0.1** is a **complete rewrite on top of Lenis** — 9.4KB gzipped (down from ~12.1KB in v4), full TypeScript support. [npmjs.com/package/@studio-freight/locomotive-scroll](https://www.npmjs.com/package/@studio-freight/locomotive-scroll)
- **Maintenance status is genuinely mixed**: the standalone npm package for the pre-v5 line was deprecated with an explicit "no longer supported" author note, while the GitHub repo shows some 2026 dependency-update activity. Given v5 is architecturally just "Lenis plus a compatibility API," **the pragmatic recommendation for new projects in Sept 2026 is to use Lenis directly** rather than Locomotive v5, unless you're migrating an existing Locomotive-API codebase.

---

## 4. Native CSS

### 4.1 Scroll-driven animations (`animation-timeline`, `view()`, `scroll()`)

**Browser support (Sept 2026):** Chrome/Edge 115+ full support. Safari added support in Safari 26 (2025). **Firefox remains flag-gated / not production-ready** per current sources — treat as a Chromium+Safari feature with a graceful-degradation fallback (JS/IntersectionObserver or just skip the effect) for Firefox until confirmed otherwise. Verify current Firefox status via caniuse before shipping Firefox-only fallback logic. [developer.chrome.com/docs/css-ui/scroll-driven-animations](https://developer.chrome.com/docs/css-ui/scroll-driven-animations), [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Scroll-driven_animations)

```css
/* scroll() — progress tied to a scrollable container's own scroll offset */
main { scroll-timeline: --page-scroll block; }
.progress-bar {
  animation: fill linear;
  animation-timeline: --page-scroll;   /* or inline: animation-timeline: scroll(block nearest); */
  transform-origin: left;
}
@keyframes fill { from { transform: scaleX(0); } to { transform: scaleX(1); } }

/* view() — progress tied to an element's visibility within its scrollport */
.reveal {
  animation: fade-up linear;
  animation-timeline: view();
  animation-range: entry 0% cover 40%;   /* only animate during the entry phase */
}
@keyframes fade-up { from { opacity: 0; translate: 0 30px; } to { opacity: 1; translate: 0 0; } }
```
- Named timelines: `view-timeline: --name block;` / `view-timeline-name`, `view-timeline-axis`, `view-timeline-inset` (like a margin before triggering). `scroll-timeline` has the parallel `scroll-timeline-name`/`-axis`.
- **`timeline-scope`** lets a descendant's `animation-timeline` reference a named timeline defined *outside* its own subtree (e.g., a fixed progress bar reacting to a `view-timeline` on a section elsewhere in the DOM).
- **`animation-range`** phase keywords: `cover` (full entry→exit span), `contain` (only while fully visible), `entry`, `exit`, `entry-crossing`, `exit-crossing`, each combinable with a percentage (`entry 0% entry 100%` = the entire entry phase only).
- Feature-detect with `@supports not (animation-timeline: --x) { /* fallback */ }`.
- Codrops has a good from-scratch intro: [tympanus.net/codrops — scroll() and view()](https://tympanus.net/codrops/2024/01/17/a-practical-introduction-to-scroll-driven-animations-with-css-scroll-and-view/)

### 4.2 `@starting-style` + `transition-behavior: allow-discrete`

The combo that finally lets you `transition` an element **into** existence (including from `display: none`) without JS.
```css
.dialog {
  opacity: 0;
  transition: opacity 0.3s, display 0.3s allow-discrete;
  transition-behavior: allow-discrete; /* or fold into the shorthand above */
}
.dialog[open] { opacity: 1; }
@starting-style {
  .dialog[open] { opacity: 0; }   /* the "from" state for the entry transition */
}
```
- **Baseline Newly Available** since Firefox 129: Chrome 117+, Edge 117+, Safari 17.5+, Firefox 129+. ~91% global support. [web.dev/blog/baseline-entry-animations](https://web.dev/blog/baseline-entry-animations)
- **Gotcha**: Safari does not yet support `allow-discrete` for the top-layer/overlay case — popover and `<dialog>` *closing* animations fall back to an instant snap in Safari specifically (opening works).
- Unsupported browsers just skip the entry animation gracefully — safe progressive enhancement.

### 4.3 `interpolate-size` — NOT yet safe to rely on

```css
:root { interpolate-size: allow-keywords; }   /* lets one side of a transition be `auto`/`max-content`/`fit-content`/`min-content` */
section { height: 2.5rem; overflow: hidden; transition: height 0.5s ease; }
section:hover { height: max-content; }         /* now animates smoothly instead of snapping */
```
**Explicitly flagged by MDN as "limited availability," not yet Baseline**, as of the current docs. Only one side of the transition may be a keyword (you can't interpolate between two intrinsic sizes). Use `@supports (interpolate-size: allow-keywords)` and keep a JS-measured-height fallback (`scrollHeight`) for the else-branch — do not ship this as your only mechanism for animating height-from-auto in 2026. [MDN interpolate-size](https://developer.mozilla.org/en-US/docs/Web/CSS/interpolate-size)

### 4.4 `text-wrap: balance` / `pretty`

```css
h1, h2, h3 { text-wrap: balance; }   /* even line lengths — Chromium caps this at ≤6 lines, falls back to normal wrap beyond that */
p { text-wrap: pretty; }              /* better last-line/orphan avoidance for body copy */
```
- `pretty`: Chrome/Edge 117+, Opera 103+, Safari 26+. **Firefox does not support `pretty`** as of early 2026 sources.
- `stable`: Safari 17.5+, Firefox 121+, Chrome/Edge 130+ (keeps line breaks stable while editing contenteditable content).
- All three degrade gracefully to default wrapping — safe to ship unconditionally.

### 4.5 CSS Anchor Positioning

```css
.trigger { anchor-name: --my-anchor; }
.tooltip {
  position: fixed;
  position-anchor: --my-anchor;
  top: anchor(bottom);
  left: anchor(center);
  translate: -50% 0;
}
```
- Chrome/Edge 125+. **Firefox**: landed behind a flag earlier, **enabled by default in Firefox 147 (stable, Jan 13 2026)**. **Safari 18.x**: core (`anchor-name`, `position-anchor`, `anchor()`) works; `@position-try` and the `inset-area` shorthand are still missing, expected around Safari 19.
- This is the native replacement for Floating UI/Popper in the simple-tooltip case; keep a JS positioning library for complex collision-avoidance needs until `@position-try` is universal. [nexgismo.com — anchor positioning 2026](https://www.nexgismo.com/blog/css-anchor-positioning-replace-javascript-tooltip-library-2026)

### 4.6 `::scroll-marker`, `::scroll-button` — CSS-only carousels

```css
.carousel { overflow-x: auto; scroll-snap-type: x mandatory; }
.carousel li { scroll-snap-align: center; }
.carousel li::scroll-marker { content: ""; }             /* auto-generates a dot per item */
.carousel::scroll-marker-group { display: flex; gap: 8px; } /* container for the dots */
.carousel::scroll-button(left) { content: "‹"; }
.carousel::scroll-button(right) { content: "›"; }
:target-current::scroll-marker { background: currentColor; } /* style the active dot */
```
Part of CSS Overflow Level 5. **Browser support: Chrome 135+, Safari 19+; Firefox partial/behind-flag as of mid-2026.** Guard with `@supports` and keep a JS carousel (Embla, §6.9) as the fallback path for Firefox. [jomaendle.com/blog/css-carousel](https://www.jomaendle.com/blog/css-carousel), [sitepoint.com — scroll-driven CSS 2026](https://www.sitepoint.com/scrolldriven-css-in-2026-building-carousels-without-javascript/)

### 4.7 `@property` — typed, animatable custom properties

```css
@property --angle {
  syntax: "<angle>";
  inherits: false;
  initial-value: 0deg;
}
.conic { background: conic-gradient(from var(--angle), red, blue); transition: --angle 0.4s ease; }
.conic:hover { --angle: 180deg; }
```
Plain custom properties are opaque strings the browser can't interpolate; `@property` gives it a type so it can. **Baseline Widely Available** across Chrome/Firefox/Safari/Edge. JS equivalent: `CSS.registerProperty({ name: "--angle", syntax: "<angle>", inherits: false, initialValue: "0deg" })`.

### 4.8 `@scope`

```css
@scope (.card) to (.card__nested-card) {   /* "donut scope" — applies inside .card but not inside a nested .card */
  img { border-radius: 8px; }
}
```
**Marked "Baseline 2026 (newly available since March 2026)"** by MDN — younger than commonly assumed; don't rely on it for anything that needs 2024-era browser reach. Verify before shipping as load-bearing.

### 4.9 Container queries, `color-mix()`, `oklch()`

```css
.card { container-type: inline-size; }
@container (min-width: 400px) { .card-title { font-size: 1.5rem; } }

:root { --brand: oklch(65% 0.2 250); }
.button:hover { background: color-mix(in oklch, var(--brand), white 20%); }
```
Both container queries and `color-mix()`/`oklch()` are long-settled Baseline features by 2026 (color functions: Chrome 111+, Firefox 113+, Safari 15.4+, ~90% global support) — safe to use unconditionally, with a solid-color fallback declared before the modern one for the last few percent.

---

## 5. View Transitions API

### 5.1 Same-document (SPA)

```js
function updateDOM() { /* your state/DOM change */ }

if (document.startViewTransition) {
  document.startViewTransition(() => updateDOM());
} else {
  updateDOM(); // no-transition fallback
}
```
Support: **Chrome 111+, Edge 111+, Firefox 144+, Safari 18+** — this shape is now cross-browser.

### 5.2 Cross-document (MPA) — CSS-only opt-in

```css
/* on BOTH the old and new page */
@view-transition {
  navigation: auto;
}
```
No JavaScript required — same-origin navigations between two opted-in pages get an automatic crossfade. Support: **Chrome 126+, Edge 126+, Safari 18.2+. Firefox: not yet in stable** (flag only as of current sources).
- **The old `<meta name="view-transition" content="same-origin">` tag is dead** — Chrome replaced it with the CSS at-rule around v126 and the meta tag now fails *silently* with zero warning. If you're following an older tutorial, this is the #1 trap.

### 5.3 Named transitions & pseudo-elements

```css
.product-image { view-transition-name: product-hero; }  /* must be unique per element */

::view-transition-old(product-hero),
::view-transition-new(product-hero) {
  object-fit: cover;      /* fixes the stretch/distortion browsers introduce when aspect ratio changes between states */
  overflow: hidden;
  animation-duration: 0.4s;
}
::view-transition-group(product-hero) { animation-timing-function: cubic-bezier(0.16, 1, 0.3, 1); }
```
Pseudo-element tree per named transition: `::view-transition-group()` → `::view-transition-image-pair()` → `::view-transition-old()` / `::view-transition-new()`.

### 5.4 Gotchas

1. **Hard 4-second timeout on cross-document transitions.** If the new document doesn't reach a renderable state within 4s (slow TTFB, render-blocking CSS/fonts), the browser **silently cancels the transition** — no error, no event, page just navigates normally. This bites in production under real latency even when it looked fine on localhost.
2. **Image distortion**: browsers snapshot old/new state as bitmaps and cross-fade/scale them — mismatched aspect ratios stretch visibly unless you set `object-fit: cover` on the `::view-transition-old/new` pseudo-elements (§5.3).
3. **Deprecated meta-tag syntax** (§5.2) — silent failure, not an error.
4. Treat cross-document transitions as **progressive enhancement**: ship the CSS opt-in, verify the page still works with zero visual transition in Firefox, and don't make any transition state load-bearing for functionality. [css-tricks.com/cross-document-view-transitions-part-1](https://css-tricks.com/cross-document-view-transitions-part-1/), [developer.chrome.com/docs/web-platform/view-transitions](https://developer.chrome.com/docs/web-platform/view-transitions)

---

## 6. Other tools

### 6.1 Rive

```bash
npm install @rive-app/webgl2   # recommended renderer; @rive-app/canvas and @rive-app/canvas-lite also available
```
```html
<script src="https://unpkg.com/@rive-app/webgl2@latest"></script>
```
```js
const r = new rive.Rive({
  src: "character.riv",
  canvas: document.getElementById("canvas"),
  autoplay: true,
  stateMachine: "MainSM",
  autoBind: true,
  onLoad: () => r.resizeDrawingSurfaceToCanvas()
});

const inputs = r.stateMachineInputs("MainSM");
inputs.forEach(input => {
  if (input.name === "isHovering") input.value = true;   // boolean input
  if (input.name === "speed") input.value = 42;            // number input
  if (input.name === "jump") input.fire();                  // trigger input
});

// r.cleanup() when unmounting
```
**Rive vs. Lottie**: Rive's killer feature is **state machines** — you define states (idle/hover/pressed/loading/success/error) with transitions gated on boolean/number/trigger inputs, and your app just sets input values; Rive resolves the rest. Lottie has no state graph — it plays/pauses/reverses/scrubs a fixed timeline. Tradeoffs: Rive's web runtime is **~200KB gzipped (WASM)** vs. lottie-web's ~60KB, but a Rive `.riv` file is typically **10–15x smaller** than the equivalent Lottie JSON (a 240KB Lottie animation can be ~16KB in Rive), and at 20+ simultaneous animations on a page Rive's shared-runtime dirty-rect model stays flat while Lottie's per-instance overhead compounds. **Rule of thumb: Lottie for motion that plays, Rive for motion that responds.**

### 6.2 Lottie / dotLottie

- `.lottie` is the current recommended production format: a ZIP bundle of one-or-more Lottie JSON animations plus assets/fonts/themes/state-machine data, typically **~90% smaller** than raw JSON + inlined data-URI assets.
- `dotlottie-web` — Rust+WASM player, React/Vue/Svelte/Solid/Web-Component wrappers, plays both `.lottie` and legacy `.json` seamlessly. [github.com/LottieFiles/dotlottie-web](https://github.com/LottieFiles/dotlottie-web)
- Existing `lottie-web` + JSON projects: no urgency to migrate unless file size is actually a problem — it's still supported.

### 6.3 Theatre.js

- `@theatre/core` + `@theatre/studio` (visual editor) + `@theatre/r3f` (react-three-fiber integration — this absorbed the old standalone `react-three-editable` project). `@theatre/dataverse` and `@theatre/react` are now stable enough to use standalone, outside Theatre.js proper.
- Mental model: a DAW-like **sequence** system — timelines you can play/scrub/reverse/nest, keyframing arbitrary JS values (not just DOM/CSS), which is why it's the go-to for choreographing WebGL/Three.js scenes rather than DOM animation. `npm install @theatre/core@latest @theatre/studio@latest @theatre/r3f@latest` to stay current. [theatrejs.com/docs/latest/api/r3f](https://www.theatrejs.com/docs/latest/api/r3f)

### 6.4 anime.js v4

- **Full ESM rewrite** — every feature tree-shakes individually; core is **~10KB gzipped**. Built-in TypeScript types generated from JSDoc (no separate `@types` package).
- Tween engine revamped to correctly **blend overlapping animations** touching the same property via `composition: "add"` (previously they'd just fight/override).
- New `registerAdapter()` lets `animate()`/`utils.set()` target **non-DOM** objects; ships a built-in **Three.js adapter** (Object3D, materials, lights, cameras, audio nodes, TSL `UniformNode`, instanced meshes) out of the box.
- Perf target: steady 60fps animating transforms on ~3,000 DOM elements (~6,000 tweens). Official v3→v4 migration guide exists on the wiki. [github.com/juliangarnier/anime — What's new in v4](https://github.com/juliangarnier/anime/wiki/What's-new-in-Anime.js-V4)

### 6.5 Page-transition routers: Barba.js / Swup / Taxi.js

- **Swup**: **v4** is current. Lighter than Barba, plugin-based (progress bar, native-feel scroll restoration, head/meta-tag swapping, preloading, a11y announcements, JS-hook animation control, form handling, debug mode). Best default choice today for a server-rendered multi-page site that wants SPA-feeling transitions. [github.com/swup/swup](https://github.com/swup/swup)
- **Barba.js**: lightweight, AJAX+CSS driven, highly flexible transition hooks; still widely used in agency/award-site work but shows less active 2026 development signal than Swup.
- **Taxi.js**: the spiritual/drop-in successor to **Highway.js**, which is unmaintained — pick Taxi if you were already on a Highway-shaped API.
- **2026 decision rule**: for a simple crossfade/shared-element MPA transition, try the native **View Transitions API cross-document mode first** (§5.2) — zero dependencies. Reach for Swup/Barba/Taxi when you need choreographed multi-step transitions, Firefox parity today, or fine-grained JS hooks the native API doesn't expose yet.

### 6.6 Splitting.js / SplitType

- **SplitType** (`lukePeavey/SplitType`): explicitly inspired by GSAP's SplitText but animation-library-agnostic — wraps lines/words/chars in spans, you drive the animation with whatever (Motion, anime.js, vanilla WAAPI). Good default when GSAP isn't in the stack.
- **Splitting.js** (`shshaw/Splitting`): same goal, populates CSS custom properties (`--word-index` etc.) per fragment so you can animate purely in CSS with `animation-delay: calc(var(--char-index) * 30ms)` — no JS animation library needed at all.
- **If GSAP is already a dependency, just use GSAP SplitText** (§1.4) — it's free now and more capable (masking, autoSplit, built-in a11y). These two remain the right call specifically for non-GSAP stacks.

### 6.7 Matter.js / Rapier

- **Rapier** (Rust → WASM, `rapier2d-simd`/`rapier3d-simd`): the 2026 performance leader — SIMD builds run **2–5x faster than Rapier's own 2024 releases**. Best choice for 3D (pairs naturally with Three.js), scenes with hundreds+ of bodies, or projects needing both 2D and 3D physics from one engine.
- **Matter.js**: pure JS, 2D-only, simpler API and much lower learning curve — still the right pick for small/medium 2D physics (falling elements, simple ragdolls, drag-toy interactions) where Rapier's WASM setup overhead isn't worth it.

### 6.8 Darkroom Engineering ecosystem (Tempus, Hamo)

Same team as Lenis; useful companions for WebGL/creative-dev builds:
- **Tempus** (`@darkroom.engineering/tempus`) — "use only one `requestAnimationFrame` for your whole app": a shared rAF scheduler so Lenis, GSAP ticker, Three.js render loop, and custom code don't each spin their own competing rAF callback.
- **Hamo** (`@darkroom.engineering/hamo`) — React hooks: `useRect`, `useWindowSize`, `useFrame`, `useResizeObserver` — built to compose with Tempus/Lenis.

### 6.9 Number Flow, Embla

- **Number Flow** (`@number-flow/react`, current **v0.6.2**) — dependency-free animated-number component (React/Vue/Svelte/vanilla), digit-rolling transitions, `Intl.NumberFormat` locale formatting; just re-render with a new `value` prop and it animates automatically. The standard for KPI/counter/price-change UI in 2026.
```jsx
<NumberFlow value={1234.56} format={{ style: "currency", currency: "USD" }} />
```
- **Embla Carousel** (current **v8.6.0**, v9 in development) — ~7KB gzipped, dependency-free, powers shadcn/ui's carousel component. v9 focus: SSR-safe APIs (`ssrStyles()` for layout-stable first paint), a typed cancellable event model, and a new Accessibility plugin. Reach for Embla over a CSS-only `::scroll-marker` carousel (§4.6) whenever you need Firefox parity today or JS-level control (autoplay, fade, sync'd multi-carousels).

---

## 7. Craft — the taste rules that separate "animated" from "expensive-feeling"

Sourced from Emil Kowalski ([animations.dev](https://animations.dev/), [emilkowal.ski/ui](https://emilkowal.ski/ui/7-practical-animation-tips)), Rauno Freiberg ([Web Interface Guidelines](https://github.com/raunofreiberg/interfaces)), and Val Head ([A List Apart](https://alistapart.com/article/designing-safer-web-animation-for-motion-sensitivity/)).

### 7.1 Duration — concrete numbers, not vibes
- **General UI ceiling: stay under 300ms.** A 180ms dropdown reads as responsive; the same dropdown at 400ms reads as sluggish, even though 400ms sounds small in the abstract.
- Real examples from Kowalski's own component work: **tooltip transition 125ms**, **dropdown 300ms**, select-menu comparison **180ms (good) vs. 400ms (bad)**.
- Scroll-triggered entrance animations (the GSAP/ScrollTrigger world) run longer than micro-interactions — **0.6–1.2s** is the common range for a headline or card reveal; **stagger gaps of 20–80ms** for tight list rhythm, **100–300ms** when you want each item to read as individually "arriving" (see Codrops production examples below).
- Codrops production tutorials in 2026 show real stagger values in the wild: SVG mask transitions at **0.02s** stagger with `power3.out`; character-by-character reveals at **0.01s** stagger with `power3.out`; grid item reveals at **0.06s** ("60ms between each item").

### 7.2 Easing families
- **Never use `ease-in` for anything UI-initiated** — starting slow reads as sluggish/unresponsive. Use `ease-out`-family curves for anything entering or responding to input; save `ease-in` (or `ease-in-out`) for things *leaving* the screen.
- **Built-in CSS easings are too weak for premium-feeling motion** — `ease-out` in the CSS spec is a gentle curve; the curves top studios use are much more aggressive at the start. Reach for named GSAP eases (`power3.out`, `power4.out`, `expo.out`) or a custom cubic-bezier rather than the CSS keyword.
- Studios commonly reach for **`expo.out`/`quart.out`-family curves** for entrances — fast initial motion that decelerates hard into the rest position — and CustomEase (§1.9) or a hand-tuned `cubic-bezier()` when a named ease isn't quite right. Pull exact bezier numbers from a visual tool (GSAP's Ease Visualizer, easings.co-style resources) rather than guessing — hand-typed cubic-beziers that "look right" in a code editor routinely feel wrong at real animation speed.
- For springs (Motion, native CSS via `linear()` approximation, or GSAP's physics feel), lower `damping` = more oscillation/bounce, higher `stiffness` = snappier; **duration+bounce is more art-directable than raw physics params** for designers who don't think in mass/stiffness units (§2.3).

### 7.3 Stagger choreography
- `each` (fixed gap per item, scales with count) vs. `amount` (fixed total, divides by count) — pick `each` when you want consistent *rhythm* regardless of list length, `amount` when you need the whole reveal to finish in a fixed window no matter how many items.
- Grid staggers radiating `from: "center"` or a focal `[x, y]` coordinate read as more "designed" than linear index-order — GSAP's `grid`/`axis`/`from` combo (§1.2) is the tool for this.
- The stagger's own `ease` (e.g. `"power2"`) shapes how *start times* bunch up, independent of each element's individual motion curve — these are two separate easing decisions, easy to conflate.

### 7.4 Disney principles applied to UI (Rauno Freiberg)
- **Follow-through / overlapping action**: secondary elements (icon inside a button, label under an image, metadata under a title) should trail the primary element by **~100–200ms**, not move in lockstep. Simultaneous motion across unrelated elements reads as mechanical; staggered, causally-ordered motion reads as organic.
- **Frequent, low-novelty interactions should carry little-to-no animation** — the hundredth time a user opens a menu they've opened all day, a flourish becomes friction, not delight. Reserve motion budget for things the user does *occasionally* or the *first* time.
- Practical corollary from Kowalski's tooltip pattern: **delay the first trigger** (prevents accidental activation from a passing cursor) but make **subsequent triggers within the same group instant, no delay and no animation** — the group behaves as one continuous interaction rather than N separate ones.

### 7.5 "One idea per motion" / motion hierarchy
- A single animation event should communicate one state change. Don't simultaneously animate opacity + scale + rotation + color on unrelated axes "because you can" — layer effects only when each one is carrying distinct meaning (e.g., scale = "this is the thing you interacted with," opacity = "this is appearing/disappearing").
- **Motion hierarchy**: the most important content on screen should have the *clearest, simplest* motion signature; decorative/ambient motion (background gradients, floating shapes) must sit visually and durationally *beneath* it — slower, lower-contrast, never competing for the eye's attention budget.
- **Entrance vs. ambient**: entrance motion is a one-time event tied to a state change (page load, scroll-into-view, filter applied) — it should have a clear start and end and get out of the way. Ambient motion (subtle looping gradient drift, breathing glow) exists continuously and must be subtle enough to be background — if a user consciously notices a loop animation after the first few seconds, it's too strong.
- **Scroll-linked vs. scroll-triggered** (see §2.4/§4.1 for the technical mechanism): scroll-*linked* motion (progress bars, parallax, scrub-driven timelines) is appropriate for storytelling/editorial sections where the scrollbar itself becomes the "playhead." Scroll-*triggered* motion (fade-up on enter, `toggleActions`, `whileInView`) is appropriate for ordinary content reveal — it should fire once and be done, not force the user to scroll at a particular speed to see it correctly.
- **FLIP for any layout change** (§1.6, §2.5) — reordering, resizing, or repositioning DOM elements should almost never be done with ad-hoc width/height/top/left tweens; record state, make the instant change, animate the delta via `transform` (GSAP Flip or Motion's `layout`/`layoutId`).
- **When NOT to animate**: real-time/high-frequency updates (live-typing feedback, fast-scrolling lists, dashboards streaming data) — motion here adds latency to the user's perception that a state change is *complete*, which reads as slow even if the underlying update was instant. Also skip animation for anything that would delay a user completing a repeated, mechanical task, and always respect `prefers-reduced-motion` rather than treating it as optional polish (§8).

---

## 8. Accessibility & performance

### 8.1 `prefers-reduced-motion` — patterns that still look intentional

**Global-reset pattern** (retrofit onto an already-animated site — shortens everything without touching each animation individually):
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```
**Opt-in pattern** (better for new builds — motion is additive, not something to undo):
```css
.reveal { opacity: 1; transform: none; } /* baseline: visible, static */
@media (prefers-reduced-motion: no-preference) {
  .reveal { opacity: 0; transform: translateY(40px); transition: opacity 0.6s, transform 0.6s; }
  .reveal.in-view { opacity: 1; transform: none; }
}
```
**JS-driven motion (GSAP/Motion/Lenis) must check the same media query and react live to changes**, not just at load:
```js
const mq = window.matchMedia("(prefers-reduced-motion: reduce)");
let reduced = mq.matches;
mq.addEventListener("change", (e) => { reduced = e.matches; /* re-init or tear down smoothing/parallax */ });

const lenis = new Lenis({ duration: reduced ? 0 : 1.2, smoothWheel: !reduced });
gsap.globalTimeline.timeScale(reduced ? 100 : 1); // or gate scroll-linked effects entirely
```
**What "still looks good" reduced actually means**: don't just delete motion — swap large-displacement transforms for **opacity/color/blur-only** transitions (per Val Head's risk model, §8.2), keep the content's final state instant and correct, and disable parallax/scrolljacking/smooth-scroll specifically (the vestibular-trigger category) while leaving small, contained hover feedback (scale 0.97 on `:active`, for instance) — WCAG's concern is *large, spatially-disorienting* motion, not all motion whatsoever.

### 8.2 Vestibular safety model (Val Head)
Three concrete triggers to avoid for motion-sensitive users, independent of duration:
1. **Relative size of movement vs. viewport** — a full-screen parallax shift is far riskier than a small button micro-animation, regardless of speed.
2. **Mismatched direction/speed between layers** — multi-layer parallax and scrolljacking are the classic offenders because foreground/background move at different rates, creating a vection mismatch with the inner ear.
3. **Distance covered in perceived 3D/virtual space** — simulated camera movement or zoom-through effects are riskier than flat 2D motion of equivalent screen-pixel distance.
- **Safe-by-default properties**: opacity, color, blur.
- **Risky properties**: large-scale `transform` (translate/scale across a big % of viewport), parallax, scroll-hijacking.
- Beyond the OS-level media query, **give users an in-page motion toggle** for sites where motion is central to the experience (WCAG 2.3.3 AAA guidance) — `prefers-reduced-motion` alone doesn't cover users who haven't discovered/set the OS setting.

### 8.3 Compositor-only properties & `will-change` discipline
- Only **`transform`** and **`opacity`** (plus, increasingly, `filter`/`backdrop-filter`) can animate on the compositor thread without triggering layout or paint — this is *the* rule for jank-free 60/120fps animation. Animating `width`, `height`, `top`, `left`, `margin`, or anything that affects layout forces a synchronous layout recalculation every frame.
- `will-change` is a **hint, not a magic-fast button**: its real job is pre-promoting an element to its own GPU compositing layer *before* the animation starts, so the promotion cost doesn't land as a visible first-frame stutter mid-animation.
- **Each promoted layer costs GPU memory.** Blanket `will-change: transform` across dozens of elements creates dozens of layers and can make performance *worse* than not using it at all. Scope it tightly: add it just before the animation begins, remove it once the animation settles (GSAP's `onComplete` / Motion's `onAnimationComplete` are the natural place), don't leave it on statically in a stylesheet.

### 8.4 Avoiding layout thrash
- Batch all DOM **reads** (`getBoundingClientRect`, `offsetWidth`, etc.) before all **writes** (style mutations) in any loop — interleaving them forces the browser to synchronously recalculate layout on every iteration ("layout thrashing"). Libraries like GSAP's `ScrollTrigger` already batch this internally (it computes start/end positions upfront rather than polling continuously) — the risk is almost always in *your* code around them, not the library.

### 8.5 `requestAnimationFrame` budget & jank debugging
- Frame budget: **~16.7ms at 60Hz**, **~8.3ms at 120Hz** (ProMotion/high-refresh displays) — everything your JS does per frame (layout reads, style writes, non-compositor animation math) has to fit inside that window or you drop a frame.
- Prefer **one shared rAF loop** (GSAP's `gsap.ticker`, or darkroom's Tempus, §6.8) over multiple independent `requestAnimationFrame` callbacks scattered across libraries — each additional independent loop adds scheduling overhead and makes jank harder to attribute.
- **Debugging**: Chrome DevTools Performance panel — use the "Graphics"-oriented capture (not the general "Web Developer" preset) specifically when chasing animation jank; look for Layout/Paint/GC markers clustering inside a single frame as the smoking gun, and cross-reference against the Layers panel to confirm which elements are actually on their own compositor layer versus repainting.

---

## Copy-paste starter: GSAP + ScrollTrigger + SplitText + Lenis + reduced-motion

A complete, correct, single-file boilerplate. Pin the version, drop in your content, done.

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
<title>Motion Starter</title>

<link rel="stylesheet" href="https://unpkg.com/lenis@1.3.26/dist/lenis.css" />

<style>
  :root { color-scheme: light dark; }
  * { box-sizing: border-box; }
  body { margin: 0; font-family: system-ui, sans-serif; background: #0b0b0c; color: #f4f4f5; }

  .hero { min-height: 100svh; display: grid; place-items: center; padding: 2rem; }
  .hero h1 { font-size: clamp(2rem, 6vw, 5rem); text-wrap: balance; margin: 0; }

  .reveal { opacity: 0; transform: translateY(40px); }

  /* Line-mask target: SplitText's mask option wraps each .line-mask line in a clip container automatically —
     no manual overflow:hidden markup needed here. */

  .section { min-height: 60vh; display: grid; place-items: center; padding: 4rem 2rem; }

  /* --- Reduced motion: opt-in pattern (§8.1). Baseline is static + visible. --- */
  @media (prefers-reduced-motion: no-preference) {
    .reveal { transition: opacity 0.3s ease; } /* CSS fallback only; JS drives the real animation below */
  }
</style>
</head>
<body>

<section class="hero">
  <h1 class="split-title">Built for motion.</h1>
</section>

<section class="section">
  <div class="reveal">
    <p>Scroll-triggered content fades up once, on entry, and stays.</p>
  </div>
</section>

<section class="section">
  <div class="reveal">
    <p>Second block — staggers in behind the first.</p>
  </div>
</section>

<!-- Pin exact versions. gsap.min.js bundles core only; plugins are separate files. -->
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/gsap.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/ScrollTrigger.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/gsap@3.15/dist/SplitText.min.js"></script>
<script src="https://unpkg.com/lenis@1.3.26/dist/lenis.min.js"></script>

<script>
(function () {
  "use strict";

  var reduceMotionQuery = window.matchMedia("(prefers-reduced-motion: reduce)");
  var prefersReduced = reduceMotionQuery.matches;

  gsap.registerPlugin(ScrollTrigger, SplitText);

  // ---- Lenis smooth scroll, driven by GSAP's single shared ticker (§3.1) ----
  // Skipped entirely for reduced-motion users: they get native instant scroll, which is
  // both more comfortable (no vestibular risk) and zero extra JS cost for them.
  var lenis = null;
  if (!prefersReduced) {
    lenis = new Lenis({
      duration: 1.2,
      smoothWheel: true,
      wheelMultiplier: 1,
      touchMultiplier: 1
    });
    lenis.on("scroll", ScrollTrigger.update);
    gsap.ticker.add(function (time) { lenis.raf(time * 1000); });
    gsap.ticker.lagSmoothing(0);
  }

  // ---- SplitText headline: word-level entrance, respects reduced motion by skipping straight to end state ----
  SplitText.create(".split-title", {
    type: "words,chars",
    mask: "chars",
    autoSplit: true,
    onSplit: function (self) {
      if (prefersReduced) {
        return gsap.set(self.chars, { yPercent: 0, opacity: 1 });
      }
      return gsap.from(self.chars, {
        yPercent: 110,
        opacity: 0,
        stagger: 0.02,
        duration: 0.8,
        ease: "power4.out",
        delay: 0.1
      });
    }
  });

  // ---- Scroll-triggered fade-ups (§7.5: fires once, plays independently of scroll speed) ----
  gsap.utils.toArray(".reveal").forEach(function (el) {
    if (prefersReduced) {
      gsap.set(el, { opacity: 1, y: 0 });
      return;
    }
    gsap.to(el, {
      opacity: 1,
      y: 0,
      duration: 0.8,
      ease: "power3.out",
      scrollTrigger: {
        trigger: el,
        start: "top 85%",
        toggleActions: "play none none reverse"
      }
    });
  });

  // ---- React live if the OS setting changes mid-session ----
  reduceMotionQuery.addEventListener("change", function () {
    window.location.reload(); // simplest correct behavior: re-init everything in the right mode
  });

  // ---- Recompute ScrollTrigger positions after webfonts / images settle layout ----
  window.addEventListener("load", function () { ScrollTrigger.refresh(); });
})();
</script>
</body>
</html>
```

**What this boilerplate deliberately does:**
- Single shared rAF loop (`gsap.ticker` drives Lenis; §3.1/§8.5) — not two competing loops.
- `prefers-reduced-motion` is checked **once at load and branched around**, not just "shortened" — reduced-motion users skip Lenis entirely (native scroll) and get `gsap.set()` end-states instead of animations, per the opt-in philosophy in §8.1.
- `SplitText` uses `mask: "chars"` + `autoSplit: true` so it re-splits correctly on resize/font-load with zero manual listener code (§1.4).
- `ScrollTrigger.refresh()` on `load` guards against the classic "positions computed before web fonts finished loading" bug.
- `toggleActions: "play none none reverse"` — replays correctly if the user scrolls back up past the trigger (§1.3).

---

## Source index

GSAP: [gsap.com/pricing](https://gsap.com/pricing/) · [gsap.com/docs/v3](https://gsap.com/docs/v3/) · [ScrollTrigger](https://gsap.com/docs/v3/Plugins/ScrollTrigger/) · [SplitText](https://gsap.com/docs/v3/Plugins/SplitText/) · [ScrollSmoother](https://gsap.com/docs/v3/Plugins/ScrollSmoother/) · [React/useGSAP](https://gsap.com/resources/React/)
Motion: [motion.dev](https://motion.dev/) · [motion.dev/docs/react](https://motion.dev/docs/react) · [motion.dev/changelog](https://motion.dev/changelog) · [motion.dev/docs/spring](https://motion.dev/docs/spring)
Lenis: [lenis.dev](https://lenis.dev/) · [github.com/darkroomengineering/lenis](https://github.com/darkroomengineering/lenis)
CSS: [developer.mozilla.org — Scroll-driven animations](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Scroll-driven_animations) · [developer.chrome.com — scroll-driven animations](https://developer.chrome.com/docs/css-ui/scroll-driven-animations) · [web.dev — baseline entry animations](https://web.dev/blog/baseline-entry-animations) · [MDN interpolate-size](https://developer.mozilla.org/en-US/docs/Web/CSS/interpolate-size) · [MDN @scope](https://developer.mozilla.org/en-US/docs/Web/CSS/@scope) · [MDN @property](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@property)
View Transitions: [developer.chrome.com/docs/web-platform/view-transitions](https://developer.chrome.com/docs/web-platform/view-transitions) · [css-tricks.com — cross-document gotchas](https://css-tricks.com/cross-document-view-transitions-part-1/)
Craft: [emilkowal.ski/ui](https://emilkowal.ski/ui/7-practical-animation-tips) · [animations.dev](https://animations.dev/) · [github.com/raunofreiberg/interfaces](https://github.com/raunofreiberg/interfaces) · [alistapart.com — designing safer web animation](https://alistapart.com/article/designing-safer-web-animation-for-motion-sensitivity/)
Other tools: [rive.app](https://rive.app/) · [dotlottie.io](https://dotlottie.io/) · [theatrejs.com](https://www.theatrejs.com/) · [github.com/juliangarnier/anime](https://github.com/juliangarnier/anime) · [github.com/swup/swup](https://github.com/swup/swup) · [embla-carousel.com](https://www.embla-carousel.com/) · [number-flow.barvian.me](https://number-flow.barvian.me/)
