# Field & antenna on the GPU: ink-line interference and array-factor geometry

*Dated 2026-09-15. Extends `webgl.md` (the ladder, loading, non-negotiables, effect catalogue) with the two specific GPU moments this site uses: a two-source interference field drawn as ink lines, and a scroll-grown antenna radiation pattern. Read `webgl.md` first; this file does not repeat its general rules.*

## 1. Interference field in a fragment shader

Two coherent sources, `S1` fixed and `S2` = smoothed cursor. Per fragment at world position `p`:

```
r1 = length(p - S1); r2 = length(p - S2)
phi = (r1 - r2) / lambda          // fringe order, continuous
```

Integer `phi` = constructive fringes. Draw as thin lines by measuring distance to the nearest fringe and antialiasing against `fwidth(phi)`, not a fixed epsilon:

```
d    = abs(fract(phi + 0.5) - 0.5)   // distance to nearest fringe, fringe-units
g    = fwidth(phi)                    // ~ d(phi)/d(pixel)
w    = 0.5 * PEN_PX * g               // pen half-width in phi-space
line = 1.0 - smoothstep(w - g, w + g, d)
```

This is Inigo Quilez's derivative-based analytic-AA approach: filter the *distance to the feature* by its own screen-space rate of change (`dpdx`/`dpdy` → `fwidth`) instead of supersampling. His filterable-procedurals article works the identical case for a checkerboard and is the clearest primary source for the technique: https://iquilezles.org/articles/filterableprocedurals/ (companion piece on band-limiting: https://iquilezles.org/articles/bandlimiting/). Concrete two-source examples: Shadertoy "Waves interference" https://www.shadertoy.com/view/ddByDy, and Book of Shapes' "Interference Mesh" — two point sources, hyperbolic fringes, described explicitly as "a contour view of the classic double-slit fringe pattern": https://bookofshapes.com/patterns/interference-mesh/.

**Denser than ~2px → wash to mean.** Quilez's technique: as the derivative grows, the analytic integral collapses toward the pattern's average (0.5 for an alternating pattern) instead of aliasing. Apply the same fade here once on-screen spacing drops under ~2px:

```
spacingPx = 1.0 / max(g, 1e-4)
fade = smoothstep(1.0, 2.0, spacingPx)   // 0 = flat mean tone, 1 = full contrast
```

**Near-source singularity.** `1/r` diverges at `r=0`; soften as `A = 1/sqrt(r*r + eps*eps)` (Plummer-style softening, standard for point-source singularities), `eps` a few px, so amplitude saturates instead of `Inf`/`NaN`.

**Falloff & mask.** `A_i(p) = 1/sqrt(r_i^2+eps^2)` sets line opacity only, never phase. Paper below the source axis (`p.y < axisY`) is blanked with one `step()` before the line term applies.

**Uniforms.** `uS1, uS2 (vec2)`, `uLambda (px)`, `uPenPx (~1.4)`, `uTime`, `uScrollT`, `uResolution`, `uDpr`; `uS2` eases toward the pointer via `mix(uS2, pointerPos, 1-exp(-dt/tau))`.

**Breathing under the naming threshold.** `lambda = lambda0*(1+0.02*sin(TAU*0.02*uTime))` — ±2%, ~50s period. WCAG's animation guidance targets motion that is *substantial or distracting* (SC 2.3.3 is about interaction-triggered motion; ambient/ decorative motion this small and slow is the case the guidance doesn't reach for) — https://dequeuniversity.com/resources/wcag2.1/2.3.3-animations-from-interactions, https://web.dev/learn/accessibility/motion. Freeze it outright under `prefers-reduced-motion` regardless.

## 2. Performance on mobile

- **DPR cap.** Cost scales as `w*h*dpr^2`. A 2026 three.js performance-tips collection states it plainly: "High-DPI phones report a devicePixelRatio of 3 or more — 9x the pixels of a 1x screen. Cap it; few people can see the difference above 2" → `renderer.setPixelRatio(Math.min(devicePixelRatio, 2))`, tighter (1–1.5) for this fullscreen effect on small viewports. https://www.utsubo.com/blog/threejs-best-practices-100-tips (Tip 80).
- **Half-res FBO vs full-res.** Same source, same tip: rendering at half resolution and upscaling "can roughly double frame rate in fill-rate-bound scenes." `fwidth` at half-res just reports a proportionally larger derivative, so the line AA degrades gracefully (softer, not broken) — appropriate here because the shader is fill-rate-, not ALU-, bound.
- **Per-pixel cost.** ~2x `length()`, a handful of scalar ops, 1–2 `fwidth`/`smoothstep` — cheap ALU; cost is essentially pixel-count × DPR². For comparison, the same tips collection anchors mobile budgets around "~100 draw calls per frame" (Tip 30) — this effect is 1 draw call, so the constraint here is fill-rate, not draw overhead.
- **Frame budget.** 60fps = 16.6ms total for JS, style, layout, paint and composite combined (web.dev: "you have 16.6 milliseconds per frame to do everything" — https://web.dev/rendering-performance/). Budget one fullscreen effect on a mid-range Android GPU (Adreno 6xx/Mali-G7x class) at ~4–6ms GPU time, leaving headroom for the rest of the page.
- **When to pause.** Stop the loop, don't just skip drawing, on `document.hidden` and when the canvas leaves the viewport (`IntersectionObserver`); three.js's own `Timer` utility does the tab-visibility half for free via `timer.connect(document)` ("pauses on hidden tabs, so there's no huge delta on return" — same Tip-80/100 source above). Since rest state is static, throttle the idle "breathing" tick to ~6–10Hz rather than 60Hz; ramp to full cadence only while the pointer is active or a scroll is in flight.

## 3. Antenna pattern as geometry

**Uniform linear array factor** (N elements, spacing d, steering phase β, `k=2π/λ`):

```
psi = k*d*cos(theta) + beta
AF(theta) = sum_{n=0}^{N-1} exp(j*n*psi) = sin(N*psi/2) / sin(psi/2)
```

closed form via the geometric-series identity; steer the main lobe to `theta0` via `beta = -k*d*cos(theta0)`. Sanity check at N=2: `|AF| = 2*|cos(psi/2)|`. Standard derivation: https://www.antenna-theory.com/arrays/weights/uniform.php, https://en.wikipedia.org/wiki/Array_factor; applied/visual treatment: https://www.analog.com/en/resources/analog-dialogue/articles/phased-array-antenna-patterns-part1.html.

**Element pattern.** Total pattern via pattern multiplication: `F(theta) = f_e(theta) * |AF(theta)|` — "the total field of an array... formed by multiplying the field of a single element and the array factor," treating element and array separately: https://www.gaussianwaves.com/2020/06/array-pattern-multiplication/. `f_e`: isotropic = 1, short-dipole-like = `|cos(theta)|`/`|sin(theta)|` by orientation.

**dB vs linear.** `F_n = F/max(F)`. Linear: plot `F_n` directly — true zero nulls, clean silhouette. dB: `F_dB = 20*log10(F_n)`, clamp at a floor (−30 or −40dB) and remap to `[0,1]`: `r(theta) = clamp((F_dB-floor)/(0-floor), 0, 1)`, avoiding `-Infinity` at nulls — MATLAB's antenna-pattern tools take the same floor-clamped approach by default (dB polar magnitudes "may be negative when dB units are used," clamped/scaled for display): https://www.mathworks.com/help/antenna/ref/patterncustom.html, https://www.mathworks.com/help/phased/ref/polarpattern.html.

**Geometry.** Sample `theta` over M steps (256–512), compute `r(theta)`, map to `(x,y)=(r*cos theta, r*sin theta)` (extend to spherical `(theta,phi)` for a 2-D array's full 3-D lobe). Build once as `THREE.BufferGeometry` with a `Float32Array` position attribute (`DynamicDrawUsage`); render `LineLoop`/`LineSegments` for the blueprint wireframe, or an indexed triangle fan for a filled lobe. In the WebGPU/TSL stack the same closed form is a small TSL node evaluated per vertex/compute: official node reference https://github.com/mrdoob/three.js/wiki/Three.js-Shading-Language; a 2026 walkthrough of building parametric/procedural geometry this way: https://tympanus.net/codrops/2026/08/11/exploring-procedural-geometry-with-three-js-and-webgpu/.

**Growing N from 1 to 8 on scroll.** The closed form takes `N` as a scalar — no per-element loop to animate, just re-evaluate `|AF(theta,N)|` with scroll-driven `N`. For a continuous feel, blend the two neighbouring integer-N curves sample-for-sample: `AF_interp = mix(AF(theta,floor(N)), AF(theta,ceil(N)), fract(N))` — not a physically exact fractional array, but a smooth monotonic morph landing exactly on the true pattern at each integer N. Update the same M-length position buffer every frame (`needsUpdate = true`); vertex count never changes, so no geometry rebuild or GC churn.

## 4. three.js r186 specifics (verified Sept 2026)

r186 shipped 2026-09-08: https://github.com/mrdoob/three.js/releases/tag/r186. Two changes bite here: the **CommonJS build and minified builds are gone** (an old `three.min.js` CDN tag now 404s — use the ESM build), and **render targets no longer auto-scale their viewport by the pixel ratio**, so a hand-rolled half-res FBO must size itself explicitly rather than relying on old implicit scaling.

- `import { WebGPURenderer } from 'three/webgpu'`; TSL nodes from `'three/tsl'` — a separate module graph from classic `'three'`. Hand-written raw-GLSL `ShaderMaterial` (the field shader) stays on the classic `WebGLRenderer`/OGL path — hence the two-stack split here. Docs: https://threejs.org/docs/pages/WebGPURenderer.html.
- `new WebGPURenderer({ forceWebGL: true })` — "set to true to test WebGL fallback" — forces the WebGL2 backend on the same renderer API, the supported way to develop/ship the fallback without branching scene-graph code (community comparison thread: https://discourse.threejs.org/t/webgpurenderer-forcewebgl-true-vs-webglrenderer/87805/4). `three/webgpu`'s entry point already bundles "renderer, materials, lights" with the fallback wired in: https://www.utsubo.com/blog/webgpu-threejs-migration-guide.
- Import-map wiring without a bundler: point `three` **and** `three/webgpu` at `three.webgpu.js`, and `three/tsl` at `three.tsl.js` (do not use plain `three.module.js` if TSL is needed) — https://threejs.org/manual/en/installation.html. A live Sept-2026 pitfall: mixing a bundler-resolved `three` with a CDN-resolved `three/tsl` (or vice versa) throws "Multiple instances of Three.js being imported," so every subpath must resolve through the *same* import map/package instance: https://discourse.threejs.org/t/import-from-three-tsl-warning-multiple-instances-of-three-js-being-imported/70303.
- TSL line materials in the webgpu examples are newer/thinner than the classic `LineMaterial`; default to plain `LineBasicMaterial`/`LineSegments` for the antenna wireframe and reach for a TSL node material only where a per-vertex/fragment computation earns its keep (colour by theta or dB). Deep-dive reference: https://blog.maximeheckel.com/posts/field-guide-to-tsl-and-webgpu/.
- WebGPU coverage has grown fast — one 2026 migration guide estimates "~95% of users globally" post-Safari-rollout — treat that as an estimate, and keep developing against `forceWebGL` by default, not as a rare edge case. Live matrix: https://caniuse.com/webgpu.

## 5. Fallbacks

- **No WebGL2/WebGPU at all:** detect once at init and swap the canvas for a static pre-rendered image (same crop/colours), explicit `width`/`height` to avoid layout shift.
- **`prefers-reduced-motion: reduce`:** skip the rAF loop after first paint; freeze `uTime`, disable breathing and cursor-follow; one static crisp frame only.
- **Touch/coarse pointer:** gate cursor-coupling on `matchMedia('(hover: hover) and (pointer: fine)')`; on touch, the second source runs a slow scripted idle path instead of following the finger.
- **A11y for decorative canvases:** both canvases are decorative only. Set `tabindex="-1"` then `aria-hidden="true"` — MDN warns `aria-hidden` must never sit on, or above, something still focusable, so "not focusable" should precede "hidden from the tree": https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Reference/Attributes/aria-hidden. Nothing essential should be conveyed only inside the canvas.

## 6. Brief for the shader agent

1. Rest-state fringe pitch measures 10–26px on screen wherever fringes are visible (`1/fwidth(phi)`).
2. Pen stroke is 1.4px ±0.2px FWHM at DPR1, constant across DPR via the fwidth-scaled AA band.
3. No visible strobe/Moiré as `d` or `lambda` changes slowly — lines wash to flat mean tone once on-screen spacing <2px.
4. Area below the source axis is flat paper colour, zero fringe leakage, checked at 2x zoom on the boundary.
5. Both `1/r` terms are singularity-safe: no NaN/Inf/flash within an eps-radius of either source, including cursor exactly on the fixed source.
6. `uS2` tracks the pointer only under `(hover:hover) and (pointer:fine)`; touch uses the scripted idle path — verified on a real touch device.
7. Breathing modulates `lambda` by ≤2% with period ≥30s and is fully frozen under `prefers-reduced-motion`.
8. Idle tick rate ≤10Hz when nothing moves and tab is visible; rAF fully stops on `document.hidden` and outside the viewport.
9. DPR is clamped (≤1.5 mobile / ≤2 desktop, never raw `devicePixelRatio`); field pass runs at half-res FBO on mobile widths, confirmed via backing-store pixel size.
10. Antenna mesh vertex count is fixed across N=1→8; only the position buffer's radii update per frame, no geometry reallocation.
11. Antenna pattern comes from the closed form `sin(N*psi/2)/sin(psi/2)`, not a per-element loop, and matches `2*|cos(psi/2)|` at N=2 as a unit check.
12. `forceWebGL` path is exercised in dev and visually matches native WebGPU for the antenna geometry; total API absence swaps in the static fallback with no console errors and no CLS.
