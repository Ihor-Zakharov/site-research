# Scroll choreography and entrance motion — 2025–2026 field notes
Dated 2026-09-15. Extends `motion-system.md` (stack, restraint standard, accessibility) and `devices.md` (§4 Scroll) — new material only, every claim sourced.

## 1. The "stick, then change state" hold
- **Apple — Vision Pro launch page**: one `position: sticky` container holds the product; internal parts translate/cross-fade roughly 1–2 viewport-heights per beat before the container releases. Scroll-up reversal is free — the sticky range just falls out from the top. [css-tricks.com](https://css-tricks.com/recreating-apples-vision-pro-animation-in-css/)
- **Apple — AirPods Pro page**: a pinned canvas frame-sequence with a stack of clips wiped away one at a time; each wipe reads as a match cut, never a slide change. [awwwards.com](https://www.awwwards.com/inspiration/product-scroll-triggered-animation-apple-airpods-pro), [medium.com](https://ankittrehan2000.medium.com/creating-scroll-animations-similar-to-apples-airpods-pro-page-bc5c1c0814df)
- **Shopify Editions, Spring 2026** (Awwwards): every section runs as its own entrance→hold→exit beat — type scatters and reforms, panels stack in depth — so a long informational page still reads as one performance. [utsubo.com](https://www.utsubo.com/blog/best-threejs-websites-2026)
- **Sleep Well Creative**, Awwwards SOTD Jan 2026: an illustrated 3D stage holds while each scroll increment advances narrative and visuals together — "like an animated editorial," not discrete cards. [utsubo.com](https://www.utsubo.com/blog/best-threejs-websites-2026)
- **Cartier Watches & Wonders 2026** (Awwwards SOTD + CSS Design Awards): scroll moves you between six self-contained 3D "rooms" with hidden gestures — the room is the unit that holds, not a generic panel. [utsubo.com](https://www.utsubo.com/blog/best-threejs-websites-2026)
- **By-Kin** (2026 juror pick): "weighted smooth scroll, and transitions that never call attention to themselves" — the hold is felt, not staged. [hontran.dev](https://www.hontran.dev/blog/best-award-winning-websites-2026)
- Hold length: no source gives a house number except the Vision Pro rebuild's ~1–2vh per beat — treat that as a working default, not a rule.
- Anti-slide-deck trick shared by all six: the thing that changes is continuous (a translate, a wipe, a cross-fade), never a hard cut between two static states.
- Mobile: sticky-based holds invert for free; GSAP-pinned holds need `normalizeScroll` so address-bar show/hide doesn't desync the pin (§4).

## 2. Scroll-linked counters and measured values
- Standard build: IntersectionObserver fires a 0→target count once the block enters view; common on KPI cards and fundraising meters, skip on utility UI where a static number reads faster. [codefronts.com](https://codefronts.com/motion/css-number-counter-animations/)
- Formatting rule the guides agree on: `font-variant-numeric: tabular-nums` on the counter so digits don't jitter the layout as they change — one real bug report describes a ticking timer shifting a table by a pixel every second until this was added. [theosoti.com](https://theosoti.com/short/tabular-nums/)
- Easing: ease-out, never elastic/bounce — a counter overshooting past its own final number reads as a bug, not a flourish (same "no bounce" standard already in `motion-system.md` §3).
- Accessibility gap most builds miss: screen readers announce every intermediate frame unless the count is frozen to the final value under `prefers-reduced-motion`; keyboard triggers need `tabindex`/`:focus`. [css-tricks.com](https://css-tricks.com/animating-number-counters/)
- The CSS-only `@property`-interpolated counter (no JS) is Chromium-only as of this writing — ship a JS fallback for Safari/Firefox. [css-tricks.com](https://css-tricks.com/animating-number-counters/)
- Gimmicky verdict: the same jury standard used for hero motion applies — a tally with no real stat behind it reads as filler exactly like a low-fps 3D hero; the number must do informational work, not decoration. [hontran.dev](https://www.hontran.dev/blog/best-award-winning-websites-2026)

## 3. Self-drawing line work
- Base technique unchanged since it was first documented and still correct: read the real path length, set `stroke-dasharray` to that length, animate `stroke-dashoffset` from length→0. [codyhouse.co](https://codyhouse.co/nuggets/self-drawing-svg-animation)
- 2025–26 tooling shift: hand-rolled Vivus.js-style libraries are being replaced by GSAP DrawSVG-equivalent timelines for multi-path "plotter order" sequencing; SMIL still works but GSAP gives scrub control tied to scroll. [portalzine.de](https://portalzine.de/svg-line-drawing-animation-solutions-vivus-js-alternatives-modern-approaches/), [svgai.org](https://www.svgai.org/blog/research/svg-animation-encyclopedia-complete-guide)
- Performance trap 1: animating `stroke-dashoffset` on hundreds of small elements at once isn't GPU-accelerated the way `transform`/`opacity` are — batch it, or fake it with one combined long path instead. [tiny.cloud](https://www.tiny.cloud/blog/guide-svg-animation/)
- Performance trap 2: a scaled diagram needs `vector-effect="non-scaling-stroke"` on every stroked element or line weight balloons under transform — set it per path, not per `<use>` instance, since dash values don't reliably re-resolve through `<use>` scaling. [svggenie.com](https://www.svggenie.com/blog/svg-stroke-width-scaling-fix)
- Performance trap 3: huge imported paths (traced logos, map outlines) carry thousands of redundant nodes — simplify before shipping; draw time should read as craft (1–3s), not a stutter.
- Accessibility: mark decorative drawn paths `aria-hidden="true"`, and under `prefers-reduced-motion` render the finished line immediately rather than skip it invisibly. [smashingmagazine.com](https://www.smashingmagazine.com/2021/10/respecting-users-motion-preferences/)

## 4. Lenis + GSAP ScrollTrigger in 2026
- Current package: `npm install gsap lenis` — the old `@studio-freight/lenis` scoped package is dead; React projects import the wrapper from `lenis/react`. [devdreaming.com](https://devdreaming.com/blogs/nextjs-smooth-scrolling-with-lenis-gsap)
- Recommended wiring: drive Lenis off GSAP's own ticker instead of a separate rAF loop, and kill lag-smoothing so the two clocks never drift: `gsap.ticker.add(t => lenis.raf(t*1000)); gsap.ticker.lagSmoothing(0); lenis.on('scroll', ScrollTrigger.update)`. [dev.to](https://dev.to/thebitforge/your-scroll-animations-look-amateur-heres-the-gsap-lenis-setup-that-fixes-it-48ni)
- `scrub`: pass a number, not `true` — `scrub: 1` gives a slight, intentional lag that reads as the animation following scroll; `scrub: true` feels rigidly welded. [dev.to](https://dev.to/thebitforge/your-scroll-animations-look-amateur-heres-the-gsap-lenis-setup-that-fixes-it-48ni)
- `anticipatePin: 1` on large pinned sections stops a one-frame flash of unpinned content on fast scroll; most sections need no `anticipatePin` at all. [gsap.com](https://gsap.com/docs/v3/Plugins/ScrollTrigger/)
- `normalizeScroll` forces scrolling onto the JS thread so a mobile browser's address-bar show/hide doesn't desync pinned math — a mobile-only fix, not default-on. [gsap.com](https://gsap.com/docs/v3/Plugins/ScrollTrigger/)
- Touch: `syncTouch: true` replaced the deprecated `smoothTouch`; several teams still disable Lenis smoothing below a breakpoint because native touch scrolling already feels fluid. [devdreaming.com](https://devdreaming.com/blogs/nextjs-smooth-scrolling-with-lenis-gsap), [dev.to](https://dev.to/thebitforge/your-scroll-animations-look-amateur-heres-the-gsap-lenis-setup-that-fixes-it-48ni)
- INP/long-task pitfall: scope `will-change` to the element actually transforming — applying it broadly alongside Lenis causes stutter on Windows Chrome specifically. [dev.to](https://dev.to/thebitforge/your-scroll-animations-look-amateur-heres-the-gsap-lenis-setup-that-fixes-it-48ni)
- Refresh discipline: call `ScrollTrigger.refresh()` after images/fonts settle or pinned start/end points drift from real layout. [dev.to](https://dev.to/thebitforge/your-scroll-animations-look-amateur-heres-the-gsap-lenis-setup-that-fixes-it-48ni)
- `prefers-reduced-motion`: wrap scroll-driven setup in `gsap.matchMedia()` with a `(prefers-reduced-motion: reduce)` branch so ScrollTriggers are never created (not just paused); `gsap.matchMediaRefresh()` handles a live in-app toggle. [gsap.com](https://gsap.com/docs/v3/GSAP/gsap.matchMedia()/), [gsap.com forum](https://gsap.com/community/forums/topic/27141-scrolltriggermatchmedia-and-prefers-reduced-motion/)

## 5. Restraint evidence
- Usability testing on scroll-hijacked pages found most participants at least mildly disoriented; one tester on a scroll-speed-altered page: "That was a full swipe, and it moved nowhere... if I were looking at this as a prospect, I would get severely agitated." [dontfuckwithscroll.com](https://dontfuckwithscroll.com/)
- The core complaint is control, not aesthetics: altered scroll breaks the assumption "you scroll, content moves," worse on mobile's longer scroll distances and smaller screens. [dontfuckwithscroll.com](https://dontfuckwithscroll.com/)
- A working 2026 juror is blunt about the trade-off being punished: "A 3D hero that drops to 18fps on a mid-range Android...will not win," and warns against "animation hiding weak art direction." [hontran.dev](https://www.hontran.dev/blog/best-award-winning-websites-2026)
- What the same juror rewards instead: "directed motion, not animation for its own sake — choreography," craft living "between states" rather than in the states themselves. [hontran.dev](https://www.hontran.dev/blog/best-award-winning-websites-2026)

## 6. Entrance cascades that don't read as "AOS fade-ups"
- Concrete numbers from a 2026 stagger breakdown: 20–30px travel (not 60–100px), 0.5–0.7s duration, 0.06–0.1s delay between siblings, eased with `power3.out`/`power2.out` — deceleration only, no overshoot. [lab.good-fella.com](https://lab.good-fella.com/blog/gsap-stagger-animation)
- Follow-through example: character-level "typewriter" stagger, each glyph on its own micro-delay off the same curve, reads as handwriting rather than a mass fade. [lab.good-fella.com](https://lab.good-fella.com/blog/gsap-stagger-animation)
- Shopify Editions' type treatment: letters scatter from a dispersed state and reform into the headline, rather than rising uniformly from below. [utsubo.com](https://www.utsubo.com/blog/best-threejs-websites-2026)
- Mat Voyce (2026 juror pick): "letters that stretch, snap, and recombine on scroll, all timeline-driven and tuned so the animation never blocks reading" — motion with a legibility budget, not just a stagger. [hontran.dev](https://www.hontran.dev/blog/best-award-winning-websites-2026)
- CSS-native direction for 2026: `sibling-index()` as a one-line stagger source with an `nth-child` fallback, replacing manual per-child delay lists. [abduarrahman.com](https://abduarrahman.com/blog/css-entrance-animations-20-effects/)
- The tell for a lazy cascade is uniform distance/duration/delay on every element; varying at least one of the three per section is what separates "choreographed" from "AOS default."

## Steal list
1. Wire Lenis through GSAP's own ticker with `lagSmoothing(0)` so scroll and animation share one clock. [dev.to](https://dev.to/thebitforge/your-scroll-animations-look-amateur-heres-the-gsap-lenis-setup-that-fixes-it-48ni)
2. Use `scrub: 1`, not `scrub: true` — a number reads as motion following scroll, not welded to it. [dev.to](https://dev.to/thebitforge/your-scroll-animations-look-amateur-heres-the-gsap-lenis-setup-that-fixes-it-48ni)
3. `anticipatePin: 1` on big pins only; leave the default 0 everywhere else. [gsap.com](https://gsap.com/docs/v3/Plugins/ScrollTrigger/)
4. `normalizeScroll` on mobile so the address bar can't desync a pinned section. [gsap.com](https://gsap.com/docs/v3/Plugins/ScrollTrigger/)
5. Gate every ScrollTrigger build inside `gsap.matchMedia()`'s reduced-motion branch, not a global if-check. [gsap.com](https://gsap.com/docs/v3/GSAP/gsap.matchMedia()/)
6. One sticky container, ~1–2vh of internal translate per beat, let the sticky range itself handle reverse-scroll. [css-tricks.com](https://css-tricks.com/recreating-apples-vision-pro-animation-in-css/)
7. Treat a pinned section as a "room" you move between, not a panel — gives scroll a destination, not just a distance. [utsubo.com](https://www.utsubo.com/blog/best-threejs-websites-2026)
8. `font-variant-numeric: tabular-nums` on every animated counter so digits don't jitter the layout. [theosoti.com](https://theosoti.com/short/tabular-nums/)
9. Freeze counters to their final value under `prefers-reduced-motion` — the animated version gets read aloud frame-by-frame otherwise. [css-tricks.com](https://css-tricks.com/animating-number-counters/)
10. `vector-effect="non-scaling-stroke"` on every stroked path in a self-drawing diagram before it ships at more than one size. [svggenie.com](https://www.svggenie.com/blog/svg-stroke-width-scaling-fix)
