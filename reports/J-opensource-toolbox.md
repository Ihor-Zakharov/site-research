# Open-Source Toolbox for Award-Grade Web Design (Sep 2026)

Methodology note: WebSearch was unavailable for this pass. Every version/license/date below was verified live via `registry.npmjs.org`, `api.cdnjs.com`, GitHub's unauthenticated `commits.atom` feeds (a way to check last-commit activity without touching the exhausted `api.github.com` core rate limit), and direct HTTPS checks against each tool's own site/API. Dates are "as of 2026-09-15." **STALE** = last commit/publish more than 18 months ago. Where a fact could not be verified live it is marked *(unverified)*.

---

## 1. Creative coding / generative graphics

| Library | License | Version | Size (min, uncompressed) | Status | What it is | Beats the alternatives when… |
|---|---|---|---|---|---|---|
| [p5.js](https://p5js.org) | LGPL-2.1 | 2.3.3 (pub 2026-09-07) | 990 KB | Alive | Beginner-friendly `setup()/draw()` sketch API, huge education community | Teaching, workshops, quick sketches; best docs/community of anything here |
| [Paper.js](http://paperjs.org) | MIT | 0.12.18 | 240 KB | **STALE** (GH+npm both frozen since 2024-07) | Vector scene graph with real boolean path ops (union/subtract/intersect) | You need precise vector math (path booleans) in-browser, not just drawing |
| [Two.js](https://two.js.org) | MIT | 0.8.24 (pub 2026-08-29) | 205 KB | Alive | Renderer-agnostic 2D API — same code targets SVG, Canvas, or WebGL | Output must gracefully degrade to crisp SVG, or you want to swap renderers later |
| [Zdog](https://zzz.dog) | MIT | 1.1.3 | 30 KB | **STALE** (frozen since 2022-01, Metafizzy inactive) | "Round, flat" pseudo-3D illustration engine | Static logo/icon animation — API is small and complete, abandonment matters less |
| [Rough.js](https://roughjs.com) | MIT | 4.6.6 | 28 KB | **STALE** (frozen since 2023-11) but feature-complete | Hand-drawn/sketchy rendering for Canvas & SVG | The sketchy aesthetic itself — no actively maintained alternative exists |
| [canvas-sketch](https://github.com/mattdesl/canvas-sketch) | MIT | 0.7.8 (pub **2026-05-26**) | — (CLI/framework) | Alive — mattdesl revived it | CLI + framework for generative art: seeded random, GIF/video/PNG export, print-res canvases | Plotter/print-resolution generative art with reproducible seeds |
| [Hydra](https://hydra.ojack.xyz) (`hydra-synth`) | **AGPL-3.0** | 1.4.0 (pub 2025-09-14) | — | Alive (Jul 2026) | Live-codable video synth — chain webcam/shader feedback in the browser | Live-coding/VJ visuals; **AGPL is a real license constraint**, check before embedding in a closed product |
| [regl](https://github.com/regl-project/regl) | MIT | 2.1.1 | 87 KB | Alive (regl-project fork, Jun 2026) — **original `mattdesl/regl` is dead since 2018, link the fork** | Functional/stateless WebGL1 wrapper, no scene graph | Raw shader control without Three.js's scene-graph overhead |
| [PixiJS](https://pixijs.com) v8 | MIT | 8.20.1 (pub 2026-08-26) | 819 KB | Alive, current major | Fast 2D WebGL/WebGPU renderer, sprites/filters/particles | Default choice for a hero canvas needing real GPU compositing at 60fps |
| [Konva](https://konvajs.org) | MIT | 10.5.0 (pub 2026-09-08) | 191 KB | Alive | Canvas2D scene graph with hit-detection + React/Vue bindings | Interactive drag/resize/layer UI (mini design-tool), not raw visual effects |
| [Fabric.js](http://fabricjs.com) | MIT | 7.4.0 (pub 2026-05-18) | 299 KB | Alive | Canvas object model with built-in SVG import/export + free-drawing | You need a "mini Figma" (image editor/annotation tool) in the browser |
| [Matter.js](https://brm.io/matter-js) | MIT | 0.20.0 | 83 KB | **STALE**-ish (~2.25 yr) but rock-solid | 2D rigid-body physics | Lightweight "physics playground" hero (falling/colliding shapes) |
| [Rapier](https://rapier.rs) (`@dimforge/rapier3d-compat`) | Apache-2.0 | 0.20.0 (pub 2026-08-08) | — (WASM, async init) | Alive | Rust→WASM 2D/3D physics, fastest option | Serious physics (not a toy demo); pmndrs' own recommended successor to cannon-es. Not a plain `<script>` drop-in — needs `await RAPIER.init()` |
| [cannon-es](https://github.com/pmndrs/cannon-es) | MIT | 0.20.0 | 346 KB | **STALE** (frozen since 2024-01) | 3D physics engine | Only for matching an existing old three.js tutorial; pmndrs itself now points people to Rapier |
| [d3-force](https://d3js.org) | ISC | 3.0.0 | 8 KB | Frozen but not really stale — the algorithm is "done"; d3 org alive (May 2026) | Force-directed graph layout | Network/bubble/collision graphs, not general physics |

---

## 2. SVG

| Tool | License | Version | Status | What it is | Beats the alternatives when… |
|---|---|---|---|---|---|
| [SVGO](https://github.com/svg/svgo) | MIT | 4.1.0 (pub 2026-08-24) | Alive | Node/CLI SVG optimizer | Always — run every exported SVG through it before shipping |
| [SVGOMG](https://jakearchibald.github.io/svgomg/) | MIT | — | Alive | Drag-and-drop web GUI for SVGO | No-install, one-off optimization with live before/after preview |
| [svg-path-commander](https://github.com/thednp/svg-path-commander) | MIT | 2.3.3 (pub 2026-09-02) | Alive | Modern TS toolkit: parse/transform/morph SVG `d` strings | Programmatic path manipulation — the current replacement for old Snap.svg |
| [Vivus](https://maxwellito.github.io/vivus/) | MIT | 0.4.6 | **STALE** (2021) | "Self-drawing" SVG stroke animation | Legacy only — for new work use plain CSS `stroke-dashoffset` or GSAP DrawSVG (now free, see §12) |
| [Lottie-web](https://airbnb.io/lottie/) | MIT | 5.13.0 (pub 2025-05-21) | Borderline alive (~16 mo) | After Effects → JSON → SVG/Canvas player | Designer-authored vector animation handoff, the established standard |
| [dotLottie](https://dotlottie.io) (`@lottiefiles/dotlottie-web`) | MIT | 0.80.0 (pub 2026-08-28) | Alive | Modern successor format (.lottie = zipped JSON+assets), smaller payload | New builds — supersedes `lottie-player` (**STALE**, superseded per LottieFiles' own guidance) |
| [Flubber](https://github.com/veltman/flubber) | MIT | 0.4.2 | **STALE** (2018) but tiny/dependency-free | Best-guess path-to-path shape interpolation | Free path morphing without GSAP — the problem doesn't rot even if the repo is quiet |
| [msdf-bmfont-xml](https://github.com/soimy/msdf-bmfont-xml) + [msdfgen](https://github.com/Chlumsky/msdfgen) | MIT | 2.8.0 (pub 2025-08-25) / msdfgen alive Aug 2026 | Alive | Generates MSDF bitmap fonts for GPU text rendering | Crisp scalable text inside a WebGL/PixiJS/three.js canvas |
| [SVGR](https://react-svgr.com) (`@svgr/core`) | MIT | 8.1.0 | Alive (GH Oct 2025) | SVG → React component transform | The standard; supersedes the old `svg-to-jsx` package (**STALE**, 2021) |
| [svg-sprite](https://github.com/svg-sprite/svg-sprite) | MIT | 2.0.4 | Alive (Aug 2025) | Build-time `<symbol>` sprite-sheet generator | >20 custom icons, want one HTTP request + `<use>` — `svgstore` (**STALE**, gulp-era) is the old way to do this |
| [opentype.js](https://opentype.js.org) | MIT | 2.0.0 (pub 2026-05-06) | Alive | Parses font files to path data in-browser | Custom text-on-path/kinetic-type effects without a server round-trip |
| Text-on-path | — | — | — | Native `<textPath href="#curve">` | Always — no library needed, universal support |
| `feTurbulence`/`feDisplacementMap` | — | — | — | Native SVG filter primitives | Zero-asset grain/glass/liquid-distortion — see fffuel (§5) for copy-paste recipes |
| Filter recipe galleries | — | — | — | [fecolormatrix.com](https://fecolormatrix.com), [yoksel.github.io/svg-filters](https://yoksel.github.io/svg-filters/) | Reference for hand-tuned `<filter>` primitives chains |

---

## 3. Data visualisation

| Library | License | Version | Bundle size (real, min.js) | Status | Designed or default-looking? | Beats the alternatives when… |
|---|---|---|---|---|---|---|
| [D3](https://d3js.org) | ISC | **7.9.0 — no v8 exists** (`next` dist-tag is a stale mislabel) | 280 KB | Alive (org, May 2026) | Neither — it's the substrate | You need a fully bespoke chart no off-the-shelf library can produce |
| [Observable Plot](https://observablehq.com/plot/) | ISC | 0.6.17 | 209 KB | Repo active Sep 2026, npm publish lag (~19 mo) | **Designed by default** — sensible type/margins out of the box | Best default aesthetic-per-line-of-code of anything in this table |
| [visx](https://airbnb.io/visx/) | MIT | 4.0.0 (pub 2026-06-11) | modular | Alive | Unstyled by design → never looks "off-the-shelf" | Already all-in on React, want full control over every pixel |
| [Chart.js](https://www.chartjs.org) | MIT | 4.5.1 | 209 KB | Alive | Looks like "a chart library" until restyled | Quick dashboards/stats, easiest API |
| [Apache ECharts](https://echarts.apache.org) | Apache-2.0 | 6.1.0 | 1.1 MB (tree-shakeable) | Alive | Default-looking but enormous feature range | Genuinely complex chart types (geo, 3D surface, large-data WebGL) |
| [uPlot](https://github.com/leeoniya/uPlot) | MIT | 1.6.32 | **51 KB**, renders 100k+ pts at 60fps | Alive (Sep 2026, today) | Bare-bones — you restyle everything | A chart must stay buttery-smooth (live telemetry/sensor data) |
| [Plotly.js](https://plotly.com/javascript/) | MIT | **4.1.1** (pub 2026-09-14, yesterday) | **4.8 MB(!)** | Alive but heavy | Default-looking, very "scientific" | 3D surfaces/contour/ternary plots a science site actually needs — lazy-load it |

**Honest note:** Plot and visx look designed with near-zero CSS because they ship no default visual identity to fight against. Chart.js/ECharts/Plotly all need real time invested in overriding colors, fonts, gridlines and tooltips to stop looking like "a charting library" on an award-grade site.

---

## 4. Audio

| Tool | License | Version | Status | What it is | Beats the alternatives when… |
|---|---|---|---|---|---|
| [Howler.js](https://howlerjs.com) | MIT | 2.2.4 | Alive (GH Nov 2025 despite 2023 npm pub) | Sprite-based playback, format fallback | Default "just play a sound" API for UI SFX / ambient loop + mute toggle |
| [Tone.js](https://tonejs.github.io) | MIT | 15.1.22 | Alive (Aug 2026) | Music-theory-aware Web Audio framework (transport, synths, effects) | Site generates/sequences audio rather than playing fixed clips |
| [Tuna](https://github.com/Theodeus/tuna) | BSD | — | Alive (GH Jan 2026) | Web Audio effects chain (reverb/chorus/bitcrusher) | **Load from GitHub/jsdelivr raw, not npm** — the npm package named `tuna` is an unrelated reverse-tunnel tool |
| [standardized-audio-context](https://github.com/chrisguttandin/standardized-audio-context) | MIT | 25.3.77 | Alive | Cross-browser Web Audio API shim | Hitting Safari/old-Chrome Web Audio inconsistencies directly |

**CC0/free sound sources:** [Freesound](https://freesound.org) (mixed licenses — filter explicitly for CC0), [Zapsplat](https://www.zapsplat.com) (free tier requires account + attribution unless subscribed, **not CC0**), BBC Sound Effects (RemArc licence — free for personal/educational, check terms for commercial).

**Rules for sound on a website:** opt-in only (autoplay-with-sound is blocked by every modern browser anyway), a persistent visible mute control, remember the choice in `localStorage`, keep effects short/low-volume, always provide a way to kill an ambient loop entirely.

---

## 5. Texture, noise, pattern, gradient generators

| Tool | License/cost | What it is | Beats the alternatives when… |
|---|---|---|---|
| [fffuel](https://www.fffuel.co) | Free, no attribution stated | SVG generators: gggrain, nnnoise, ffflux (liquid gradient), sssurface… | One-off hero background textures with tunable seed params, no runtime library |
| [Haikei](https://haikei.app) | Free | Blob/wave/gradient SVG background generator | Quick abstract shape backgrounds, exports SVG/PNG ready to paste |
| [Hero Patterns](https://heropatterns.com) | Free | Repeating low-opacity geometric SVG patterns, pick a color, copy the data-URI CSS | Subtle textured section backgrounds, zero image asset |
| [SVG Backgrounds](https://www.svgbackgrounds.com) | Free | Similar to Hero Patterns, broader/more illustrative pattern library | More pattern variety than Hero Patterns |
| [Magic Pattern](https://www.magicpattern.design) | Free | CSS background generator, blobs, mesh gradients, stripes/grid patterns | More agency-polish tool variety in one place |
| [meshgradient.com](https://meshgradient.com) + plain CSS multi-`radial-gradient()` | Free | Visual mesh-gradient editor → CSS/SVG/PNG export | Editor for exploring; hand-written layered `radial-gradient()`s for zero-asset production use |
| [Grained.js](https://github.com/rrag/grained) (npm `grained`) | MIT | Canvas grain-texture script | **STALE** (2017, ~1KB) — for new work, inline the `feTurbulence` recipe instead, same look, zero JS |
| [Poly Haven](https://polyhaven.com) | CC0 | HDRIs + PBR material textures (4K–8K, full map sets) + growing model library | Best single free PBR texture source, no attribution required |
| [ambientCG](https://ambientcg.com) (absorbed cc0textures.com) | CC0 | PBR materials, largest raw material count, consistent map naming | Programmatic fetching via its own API; more raw variety than Poly Haven |

**Noise: PNG tile vs `feTurbulence` vs Canvas** — PNG tile: cheapest to render, but a fixed pattern (visible repeat at scale) and an extra asset request. `feTurbulence`: infinite/no-repeat, zero asset, GPU-composited, but costly to *animate* (re-seeding every frame is expensive — throttle `baseFrequency` changes). Canvas `putImageData`: fully controllable, cheap if pre-rendered to a few tiles, but costs a paint at generation time. **For an award-grade site: prebaked static `feTurbulence` overlay is the default — cheapest and zero asset weight.**

---

## 6. CSS systems and utilities

| Tool | License | Version | Status | What it is | Beats the alternatives when… |
|---|---|---|---|---|---|
| [Open Props](https://open-props.style) | MIT | 1.7.23 (pub 2026-01-31) | Alive | Ready-made CSS custom-property design tokens (color ramps, sizes, easings, shadows) | Fastest way to a non-Bootstrap-default visual baseline with zero design-system authoring |
| [Every Layout](https://every-layout.dev) | Mostly paid (~$66) | — | — | Composable CSS layout primitives (Stack, Cluster, Sidebar, Cover…) | Treat as a *methodology* reference — core patterns are widely republished free as blog posts |
| [Utopia](https://utopia.fyi) (+`utopia-core` npm, ISC, 1.6.0) | Free | — | — | Fluid type/space scale calculator using `clamp()` | Paste the generated custom properties once; no runtime dependency needed |
| [modern-normalize](https://github.com/sindresorhus/modern-normalize) | MIT | 3.0.1 | Alive (GH Jun 2026) | Slimmed normalize.css for evergreen browsers | Default "consistent starting styles" choice |
| [Josh Comeau's CSS Reset](https://www.joshwcomeau.com/css/custom-css-reset/) | Free (blog post) | — | — | ~15-line opinionated reset tuned for component-based layout | Want a reset written for modern layout work, not legacy-browser normalization |
| [Andy Bell's "modern CSS reset"](https://piccalil.li/blog/a-more-modern-css-reset/) | Free (blog post) | — | — | Similar goal, bakes in `prefers-reduced-motion` handling directly | Want reduced-motion respect built into the reset itself |
| [sanitize.css](https://csstools.github.io/sanitize.css/) | **CC0-1.0** | 13.0.0 | Alive (GH Mar 2026) | Stricter normalize + optional forms/assets/reduce-motion add-ons | Want a fully unencumbered license — zero attribution needed anywhere |
| [PostCSS](https://postcss.org) | MIT | 8.5.28 | Alive | The transform engine | — |
| [postcss-preset-env](https://preset-env.cssdb.org) | MIT-0 | 11.5.3 | Alive | Write tomorrow's CSS, compile down for today | Check first — nesting/`@container`/`oklch()` now ship natively in most evergreen browsers |
| [cssnano](https://cssnano.co) | MIT | 9.0.4 | Alive | Standard PostCSS-based minifier | Build-step minification |
| [@csstools/postcss-oklab-function](https://github.com/csstools/postcss-plugins) | MIT-0 | 5.0.12 | Alive (monorepo committed *today*) | Compiles `oklch()`/`oklab()` to `rgb()` fallback | Need OKLCH colors with old-browser fallback |
| [css-doodle](https://css-doodle.com) | MIT | 0.51.0 | Alive | `<css-doodle>` web component for grid-based generative CSS art | Unique, zero-image-asset generative background purely in CSS |
| [container-query-polyfill](https://github.com/GoogleChromeLabs/container-query-polyfill) | Apache-2.0 | 1.0.2 | **Own README says "maintenance mode as of Nov 2022"** | `@container` query polyfill | Almost never in 2026 — native `@container` has shipped in all evergreen browsers since 2023 |
| CSS Houdini Paint Worklets | — | — | Chromium-only in practice | `CSS.paintWorklet.addModule()` custom `paint()` functions | Use [`css-paint-polyfill`](https://github.com/GoogleChromeLabs/css-paint-polyfill) (v3.4.0, Apache-2.0, **STALE** since 2022) as a fallback layer — don't depend on Houdini for anything load-bearing yet |

---

## 7. Icons

| Set | License | Version | Icon count / style | Status | Beats the alternatives when… |
|---|---|---|---|---|---|
| [Lucide](https://lucide.dev) | ISC | 1.46.0 (pub 2026-09-14) | 1600+, single weight | Alive, weekly releases | Default modern pick — community continuation of Feather |
| [Feather Icons](https://feathericons.com) | MIT | 4.29.2 | ~280 | Borderline **STALE** (frozen since ~2025-03; stopped merging PRs — *why* Lucide forked it) | Only for pixel-parity with an existing old design |
| [Phosphor Icons](https://phosphoricons.com) | MIT | 2.1.2 | 9000+ across 6 weights (thin→fill) | Alive (GH Aug 2026) | Need a weight/duotone system matching variable typography |
| [Tabler Icons](https://tabler.io/icons) | MIT | 3.46.0 (pub 2026-07-28) | 5900+ | Alive | Broadest technical/niche coverage — good for science/university feature icons |
| [Iconoir](https://iconoir.com) | MIT | 7.12.1 | 1600+ | Alive | Distinct rounded-stroke look when Lucide/Tabler feel "sameish" |
| [Remix Icon](https://remixicon.com) | Apache-2.0 | 4.9.1 | 2800+, line+fill pairs | Alive | Business/e-commerce iconography completeness |
| [Material Symbols](https://fonts.google.com/icons) (`@material-symbols/svg-400`) | Apache-2.0 | 0.47.3 (pub 2026-09-15, today) | Variable (fill/weight/grade/optical size axes) | Alive | Icon needs to visually "breathe" alongside variable-weight text |
| [Heroicons](https://heroicons.com) | MIT | 2.2.0 | ~300, outline+solid | Alive (GH May 2026) | Restraint/curation over 5000 options — less decision fatigue |
| [Radix Icons](https://icons.radix-ui.com) | MIT | 1.3.2 | ~320, 15×15 grid | Alive (GH Dec 2025) | Tight interface chrome (dropdown carets, checkboxes), not illustrative use |
| [Font Awesome Free](https://fontawesome.com) | **CC-BY-4.0 (icons) + OFL-1.1 (font) + MIT (code)** | 7.3.1 | Thousands | Alive | Brand recognition only — **the one set here that legally needs attribution** |
| [Bootstrap Icons](https://icons.getbootstrap.com) | MIT | 1.13.1 | 2000+ | Alive | Pairs natively with Bootstrap's visual language |
| [Simple Icons](https://simpleicons.org) | CC0-1.0 | 16.31.0 (pub 2026-09-13) | 3300+ **brand/logo marks** | Alive | The only correct source for "as seen on" brand rows — note trademark law still applies to the marks themselves |

**When a hand-drawn set is mandatory instead:** any set above reads as "generic SaaS" the moment a competitor uses the same one (near-certain with Lucide/Heroicons given ubiquity). For a brand-forward editorial site, commission or hand-draw a small (12–20) custom set matching the site's own linework — "complete" and "distinctive" are in tension by definition, so no free complete hand-drawn set exists.

---

## 8. Free 3D assets

| Source | License | What it is | Beats the alternatives when… |
|---|---|---|---|
| [Poly Haven](https://polyhaven.com) | CC0 | HDRIs, PBR textures, growing glTF/Blender model library | Best overall single source, no login/attribution required |
| [ambientCG](https://ambientcg.com) | CC0 | PBR materials, largest raw material count | Programmatic fetching, consistent glTF/USD/Blend export |
| [Sketchfab](https://sketchfab.com) | **Mixed, per-model** | Huge scanned/game-asset variety | Filter "Downloadable" + CC0/CC-BY explicitly — most models are *not* free by default |
| [Quaternius](https://quaternius.com) | CC0 | Low-poly stylized game-ready model packs | A playful low-poly WebGL hero scene |
| [Kenney](https://kenney.nl) | CC0 | Enormous asset library: 3D, textures, UI kits, **audio too** | Broadest CC0 library on the web, not 3D-only |
| [Smithsonian 3D](https://3d.si.edu) | Open access (verify per object) | Scanned museum/space/natural-history artifacts (incl. Apollo 11 module) | Perfect fit for a science/university subject — site has aggressive bot-protection, browse manually |
| [NASA 3D Resources](https://nasa3d.arc.nasa.gov) | Public domain (US gov work) | Spacecraft/planetary OBJ models | Any space-science page |
| [Scan the World](https://www.myminifactory.com/scantheworld) | Mostly CC-BY-NC/CC-BY (check per item) | 3D-scanned sculptures/artifacts archive | Classical-art/culture references |
| ~~pmndrs market (market.pmnd.rs)~~ | — | — | **Appears discontinued as of Sep 2026 — URL and GitHub repo both gone.** Use Poly Haven/Sketchfab CC0 instead |
| [KhronosGroup/glTF-Sample-Models](https://github.com/KhronosGroup/glTF-Sample-Models) | Mixed (mostly CC0/open) | glTF spec-compliance reference/test models | Testing format edge cases, not for design assets |

**Google Poly successors:** Poly shut down in 2021; its role is now split across Sketchfab (search/community), Poly Haven (curated CC0), and Khronos' sample repo (format testing). **Format note:** prefer glTF/GLB over OBJ/FBX for web use, and run everything through `gltf-transform`/`gltfpack` (meshopt compression) — raw exports are typically 3–10× larger than needed.

---

## 9. Free photo/video/archive (science & university angle)

| Source | License notes | Beats the alternatives when… |
|---|---|---|
| Unsplash / Pexels / Pixabay | Custom "free" licenses, **not CC0** — Unsplash forbids competing stock products/implied endorsement; Pexels forbids resale "as is"/implied sponsorship; Pixabay's 2019 update changed identifiable-people/brand rules | General photography — but re-read the current license text before commercial use, never assume "free" = "no restrictions" |
| [Cosmos](https://www.cosmos.so) | **Not a free-image source** | A moodboard/visual-discovery tool that *aggregates and attributes* images from elsewhere — use only for reference/moodboarding, then license the original at its source |
| [Wikimedia Commons](https://commons.wikimedia.org) | Genuinely mixed, filterable to CC0/PD | Default first stop for historical/scientific imagery |
| [NASA Image and Video Library](https://images.nasa.gov) | Public domain (US gov work) | Space/science site's best friend |
| [ESA Images](https://www.esa.int/ESA_Multimedia/Images) | Mostly CC BY-SA 3.0 IGO | European space-science imagery, attribution + share-alike required |
| [CERN Document Server](https://cds.cern.ch) | Mostly CC-BY-4.0 | Particle-physics imagery, direct from source |
| [LOC "Free to Use"](https://www.loc.gov/free-to-use/) | No known restrictions (curated subsets) | Themed, pre-cleared subsets inside LOC's much larger mixed-rights catalog |
| [Rijksmuseum / Rijksstudio](https://www.rijksmuseum.nl/en/rijksstudio) | Public domain | Full original-resolution art downloads; free API (register at data.rijksmuseum.nl) |
| [Met Open Access](https://www.metmuseum.org/art/collection) | CC0 where `isPublicDomain:true` | **Verified live:** `collectionapi.metmuseum.org` is a fully open REST API, **no key required**, returns the PD flag + direct image URLs — most automatable museum source here |
| [Public Domain Review](https://publicdomainreview.org) | Curated PD | Editorial curation / finding one specific rare image |
| [Internet Archive](https://archive.org) | Mixed — check each item's rights field | Enormous scale, don't assume PD |
| [Europeana](https://www.europeana.eu) | Mixed, strong rights-filter UI | Aggregates hundreds of European institutions with CC0/PDM/CC-BY facets |
| [Old Book Illustrations](https://www.oldbookillustrations.com) | Public domain | Distinctive 18th–19th-century engraving aesthetic, less overused than stock PD sets |

---

## 10. Colour tools

| Tool | License / version | What it is | Beats the alternatives when… |
|---|---|---|---|
| [oklch.com](https://oklch.com) | Free | OKLCH picker/converter with gamut-mapping | Fastest way to eyeball-pick a perceptually even color |
| [Huetone](https://huetone.ardov.me) | Free | Builds an accessible multi-hue palette, checks contrast across the whole ramp at once | Need a full contrast-safe UI palette (5–10 hues × 10 shades), not one color |
| [Adobe Leonardo](https://leonardocolor.io) (`@adobe/leonardo-contrast-colors`) | Apache-2.0, 1.1.0 (pub 2026-02-18, alive) | Generates a scale from a target *contrast ratio*, not a fixed lightness step | Requirement is literally "must hit contrast X against this background" |
| [Realtime Colors](https://www.realtimecolors.com) | Free | Live-previews a palette against a fake UI mockup | Sanity-checking a palette in context before committing |
| [Coolors](https://coolors.co) | Free | Classic fast palette generator/locker | Quick exploration — less rigorous than Huetone/Leonardo on accessibility |
| [Radix Colors](https://www.radix-ui.com/colors) (`@radix-ui/colors`) | MIT, 3.0.0 | Pre-built 12-step light+dark scales, step 9 always the accessible "solid" | Don't want to design a ramp at all — a battle-tested default |
| [Open Color](https://yeun.github.io/open-color/) | MIT, 1.9.1 | **STALE** (dormant since 2022-12) | Fixed 10-step palette | Fine to vendor frozen — needs no updates, just don't expect new hues |
| Tailwind's default palette | Free (published in OSS repo) | Well-tested neutral+accent starting point | Usable even without Tailwind itself |
| [ColorBox](https://www.colorbox.io) (Lyft Design) | Free | Full scale from 2–3 anchors with curve control | Spiritual predecessor to Leonardo/Huetone, still usable |
| [APCA](https://apcacontrast.com) (`apca-w3`) | **Custom "Limited W3 License" — not OSI-approved** | 0.1.9, active (Myndex/apca-w3 GH alive May 2026) | Perceptually-accurate contrast, WCAG 3 candidate | Read the license before shipping in a commercial product; best for *design guidance*, not yet a hard CI gate |
| [Chroma.js](https://gka.github.io/chroma.js/) | BSD-3/Apache-2.0, 3.2.0 | Runtime color math workhorse | Most battle-tested option, strong legacy color-space support |
| [culori](https://culorijs.org) | MIT, 4.0.2 | Modern, tree-shakeable, first-class OKLCH/OKLab | Targets the newer perceptual spaces natively rather than bolted on |
| [Color.js](https://colorjs.io) | MIT, 0.7.1 (pub 2026-07-24, alive) | Lea Verou/Chris Lilley's lib, mirrors CSS Color 4/5 spec function names | Want JS color math to mirror your CSS `oklch()` 1:1 |

**Perceptually-even ramp in OKLCH, in short:** fix Chroma and Hue, vary only Lightness across steps (`oklch(L% C H)`, L from ~98% to ~15%) — because OKLCH's L is perceptually uniform, equal steps *look* equally spaced (unlike HSL). Nudge Chroma down slightly at the extreme light/dark ends to avoid uneven sRGB gamut-clipping, and gate with `@supports (color: oklch(0% 0 0))` or compile via `@csstools/postcss-oklab-function` for older engines.

---

## 11. Free fonts with real Cyrillic (і ї є ґ)

Verified live against the Google Fonts API (`fonts.googleapis.com/css?family=NAME&subset=cyrillic-ext` returns HTTP 400 if that family lacks the subset — `cyrillic-ext` is specifically the subset carrying і, ї, є, ґ and other non-Russian Cyrillic, as opposed to plain `cyrillic` which only covers Russian).

**Confirmed cyrillic-ext support (free, SIL OFL, Google Fonts / Fontsource):** Inter, Montserrat, Manrope, Golos Text, PT Sans, PT Serif, PT Mono, Noto Sans, Roboto, Ubuntu, Onest, Unbounded, Wix Madefor Text, Rubik, Nunito, Fira Sans, Source Sans 3, Playfair Display, Comfortaa, Caveat, Bad Script, Yeseva One, Jost, Space Grotesk, Marck Script, Bricolage Grotesque, Instrument Sans, Instrument Serif, Geist, Sora, Schibsted Grotesk, Archivo, Hanken Grotesk, Plus Jakarta Sans, Outfit, Lexend, Fraunces, DM Sans, Big Shoulders, Anton.

PT Sans/Serif/Mono and Golos Text are **Cyrillic-native designs** (drawn Cyrillic-first) — the best pick when Cyrillic is the primary script, not a retrofitted subset.

| Source | Verified finding |
|---|---|
| [Fontshare](https://www.fontshare.com) | **Zero Cyrillic across its entire 100-font catalog** — queried the API directly (`api.fontshare.com/v2/fonts`), every single entry reports `script: "latin"`. **Do not use Clash Display/Satoshi/General Sans/Cabinet Grotesk for Ukrainian or Russian copy, no exceptions.** |
| [Velvetyne](https://velvetyne.fr) | Free/open (mostly OFL), French experimental collective — mostly Latin-only display faces; a few workhorse text faces have added Cyrillic, check each specimen individually |
| [Collletttttivo](https://www.collletttttivo.it) | Free, Italian collective, similarly Latin-experimental-leaning — site could not be reached programmatically this pass; spot-check per family before use |
| Uncut.wtf | Curated free-font directory — blocks scripted requests, so its catalog couldn't be enumerated live; re-verify Cyrillic on whichever source foundry it links to |
| [Open Foundry](https://open-foundry.com) | Curated libre (mostly OFL) collection from multiple foundries — mixed Cyrillic support, check per family |
| [Google Fonts](https://fonts.google.com) | Single most reliable Cyrillic source by volume — see verified list above |
| [Fontsource](https://fontsource.org) / [Brick](https://brick.im) | Self-host Google Fonts (npm install or direct download) instead of a Google network call — same Cyrillic coverage as the underlying family, use when avoiding third-party requests matters |

**Ukrainian-specific:** **e-Ukraine / e-Ukraine Head** — released by Ukraine's Ministry of Digital Transformation as the official Diia/state typeface, free to use, full Ukrainian Cyrillic by design. Confirmed **not** on Google Fonts (400 on live check) — self-hosted only, distributed via Diia/Ministry design resources. The most on-brand pick specifically for a Ukrainian-subject site.

**On your own open questions:** **Fedra** (Peter Bilak/Typotheque) is commercial — excellent Cyrillic but doesn't belong in a free/open toolbox. Could not confirm an established foundry named "Kyiv Type Foundry" or "MSCR" during this pass — worth a direct name search rather than assuming they exist as described.

---

## 12. Micro-libraries

| Library | License | Version | Status | What it is | Beats the alternatives when… |
|---|---|---|---|---|---|
| [Lenis](https://lenis.darkroom.engineering) | MIT | 1.3.26 (pub 2026-08-05) | Alive | Smooth-scroll normalizer | **Note:** `@studio-freight/lenis` is the old scope — **STALE** since 2024-03, the studio rebranded to darkroom.engineering and moved the package to plain `lenis` |
| Tempus (`@darkroom.engineering/tempus`) | MIT | 0.0.46 | Low activity | One shared `requestAnimationFrame` loop for other libs to subscribe to | Multiple libraries (Lenis, Theatre.js) all trying to run their own rAF |
| Hamo (`@darkroom.engineering/hamo`) | MIT | 0.6.46 | Low activity | React hooks grab-bag (resize observer, rAF, window size) | Already using Lenis/Tempus, want matching React glue |
| split-type | ISC | 0.3.4 | **STALE**-ish (~3 yr) | Splits text into lines/words/chars as spans | **GSAP's SplitText is now free** (see below) and more robust — prefer it for new work |
| Splitting.js | MIT | 1.1.0 | **STALE** (dormant since 2024-05) | CSS-custom-property-driven text split | Want to animate purely in CSS with zero JS animation code |
| Rough Notation | MIT | 0.5.1 | **STALE** (2020, ~6 yr) | Hand-drawn circle/underline/highlight annotations | No actively-maintained replacement exists for this exact effect — still safe, frozen API |
| tsParticles | MIT | 4.4.0 (pub 2026-08-31) | Alive | Confetti/fireworks/particles engine, tree-shakeable bundles | Active successor to particles.js |
| Embla Carousel | MIT | 8.6.0 | Alive | Headless/unstyled, tiny, best touch/swipe physics of the free options | Default pick over Swiper unless you need built-in UI chrome |
| Swiper | MIT | 14.2.0 (pub 2026-08-26) | Alive, current major | Batteries-included carousel (nav/pagination/coverflow/cube effects) | Want a carousel "done" fast without assembling controls yourself |
| Vaul | MIT | 1.1.2 | Alive (GH Oct 2025) | React-only bottom-sheet/drawer, mobile-native feel | Mobile drawer/sheet pattern — nothing else here does this for free |
| Sonner | MIT | 2.0.8 (pub 2026-08-09) | Alive | React toast notifications with good default stacking/motion | Default React toast choice today |
| Number Flow | MIT | 0.6.2 (pub 2026-07-18) | Alive | Framework-agnostic `<number-flow>` web component, animates digit changes | Landing-page "impact numbers" stat counters |
| Motion (`motion` npm) | MIT | 13.3.0 (pub 2026-09-14, yesterday) | Alive | Renamed/merged Framer Motion + Motion One (**Popmotion's successor too**) | Default React animation; vanilla sites use its DOM `animate()`/`scroll()` API instead of pulling in GSAP |
| Popmotion | MIT | 11.0.5 | **STALE/discontinued by its own author** | — | Don't start new work on it — folded into `motion` |
| Atropos | MIT | 2.0.2 | Alive (GH May 2026) | Layered 3D parallax hover cards, by the Swiper author | More elaborate "award-site" hover than plain tilt |
| Vanilla-tilt.js | MIT | 1.8.1 | **STALE** (frozen Nov 2023) | Tiny (9 KB), dependency-free tilt-on-hover | Simple case — API is frozen but trivial and reliable |
| Barba.js (`@barba/core`) | MIT | 2.10.3 | Low activity | Full-page AJAX transitions | Long-time standard, huge community examples |
| Swup | MIT | 4.10.0 (pub 2026-09-03) | Alive, current major | Page-transition library with a first-party plugin ecosystem | More actively developed than Barba, plugin-driven (fade/slide/progress-bar) |
| Taxi.js (`@unseenco/taxi`) | BSD-3-Clause | 1.9.1 (pub 2025-11-01) | Alive | Routing-aware page-transition manager, agency/award-site favorite | **Note:** the plain `taxi` npm package is an unrelated Selenium tool — use the `@unseenco/` scope |

**Update worth flagging:** GSAP (all bonus plugins — SplitText, MorphSVG, DrawSVG, ScrollTrigger) became **free for everyone** after the Webflow acquisition; license field now reads "Standard 'no charge' license" (not OSI-approved, but zero-cost). This changes the calculus for §2's Flubber and this section's split-type/Splitting — GSAP's official plugins are now a free, more robust option for the same jobs.

---

## 13. Accessibility & testing tooling (no paid service required)

| Tool | License | Version | Status | What it is | Beats the alternatives when… |
|---|---|---|---|---|---|
| [axe-core](https://github.com/dequelabs/axe-core) | MPL-2.0 | 4.13.0 (pub 2026-08-05) | Alive | Industry-standard automated a11y rule engine | Drop in via CDN + `axe.run()` in devtools for a zero-install spot check, or wire into CI |
| [Pa11y](https://pa11y.org) | LGPL-3.0-only | 10.0.0 (pub 2026-08-28) | Alive | CLI/CI wrapper (axe + HTML CodeSniffer rules), clean pass/fail output | Want a simple CI gate, not an interactive report |
| IBM Equal Access (`accessibility-checker`) | Apache-2.0 | 4.0.34 (pub 2026-09-08) | Alive | IBM's own rule engine (genuinely different implementation from axe) | Second opinion/cross-check, not a rebrand — also ships as a browser extension |
| [Lighthouse](https://github.com/GoogleChrome/lighthouse) | Apache-2.0 | 13.4.1 | Alive | Perf+a11y+SEO+best-practices auditor | Quick overview score — its a11y rule set is lighter than axe/IBM, not a substitute |
| [unlighthouse](https://unlighthouse.dev) | MIT | 0.18.0 (pub 2026-06-29) | Alive | Crawls a whole site, runs Lighthouse on every route, aggregate dashboard | Many pages/templates need a site-wide sweep, not just the homepage |
| [html-validate](https://html-validate.org) | MIT | 11.15.0 | Alive | Offline Node HTML5 validator/linter | CI use without hitting a hosted rate limit |
| [W3C Nu Html Checker](https://validator.w3.org/nu/) | Free, self-hostable (`vnu.jar`) | — | — | Canonical HTML validator | Self-host in CI to avoid rate limits on the hosted service |
| [WAVE](https://wave.webaim.org) | Free tool / **paid API** | — | — | Interactive browser-extension a11y checker | The extension/web tool is free by hand — **the API is a paid, credit-metered service**, don't script against it for a "no paid service" requirement |
| wcag-contrast (npm) | BSD-2-Clause | 3.0.0 | **STALE** (2019) | Tiny WCAG2 contrast function | Prefer colorjs.io's built-in `.contrast()` (covers WCAG2 + APCA) for new work |

**Screen-reader testing note:** automated tools above catch roughly 30–40% of real WCAG issues by nature — anything about meaning, reading order, or focus logic needs a human pass with NVDA (free, Windows) or VoiceOver (free, built into macOS/iOS) navigating by keyboard + screen reader only. No amount of tooling above replaces that pass.

---

## The shortlist

The 25 things to actually vendor or reach for by default on a no-build, award-grade landing page. All CDN URLs below were live-tested during this research (HTTP 200, real byte size shown = as served, uncompressed).

| # | Tool | Install / CDN | Size |
|---|---|---|---|
| 1 | Open Props (design tokens) | `<link rel="stylesheet" href="https://unpkg.com/open-props@1.7.23/open-props.min.css">` | 30 KB |
| 2 | Josh Comeau CSS reset | paste ~15 lines inline — [joshwcomeau.com/css/custom-css-reset](https://www.joshwcomeau.com/css/custom-css-reset/) | — |
| 3 | modern-normalize | `https://cdn.jsdelivr.net/npm/modern-normalize@3.0.1/modern-normalize.min.css` | 1 KB |
| 4 | Google Fonts, Cyrillic-verified (Inter + PT Sans) | `<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&family=PT+Sans:wght@400;700&display=swap" rel="stylesheet">` | — |
| 5 | Lucide icons | `https://cdn.jsdelivr.net/npm/lucide-static@1.46.0/font/lucide.css` (or per-icon SVG) | 102 KB (full sheet) |
| 6 | Simple Icons (brand logos) | `https://cdn.jsdelivr.net/npm/simple-icons@16.31.0/icons/<brand>.svg` | ~1 KB/icon |
| 7 | GSAP core | `https://cdnjs.cloudflare.com/ajax/libs/gsap/3.15.0/gsap.min.js` | 73 KB |
| 8 | GSAP ScrollTrigger | `https://cdnjs.cloudflare.com/ajax/libs/gsap/3.15.0/ScrollTrigger.min.js` | 45 KB |
| 9 | GSAP SplitText (free since Webflow acquisition) | `https://cdnjs.cloudflare.com/ajax/libs/gsap/3.15.0/SplitText.min.js` | 8 KB |
| 10 | Lenis smooth scroll | `https://cdn.jsdelivr.net/npm/lenis@1.3.26/dist/lenis.min.js` | 19 KB |
| 11 | Motion (component sites) | `npm install motion` or `https://cdn.jsdelivr.net/npm/motion@13.3.0/dist/motion.js` | 147 KB |
| 12 | Rough.js (sketchy aesthetic) | `https://cdn.jsdelivr.net/npm/roughjs@4.6.6/bundled/rough.js` | 28 KB |
| 13 | Rough Notation (hand-drawn annotations) | `npm install rough-notation` | small |
| 14 | Two.js (vector hero graphics) | `https://cdn.jsdelivr.net/npm/two.js@0.8.24/build/two.min.js` | 205 KB |
| 15 | PixiJS (WebGL hero canvas) | `https://cdn.jsdelivr.net/npm/pixi.js@8.20.1/dist/pixi.min.js` | 819 KB — lazy-load only |
| 16 | Matter.js (physics playground) | `https://cdnjs.cloudflare.com/ajax/libs/matter-js/0.20.0/matter.min.js` | 83 KB |
| 17 | tsParticles confetti | `https://cdn.jsdelivr.net/npm/@tsparticles/confetti@3.9.0/tsparticles.confetti.bundle.min.js` | 144 KB |
| 18 | Atropos (3D hover cards) | `https://cdn.jsdelivr.net/npm/atropos@2.0.2/atropos.min.js` | 7 KB |
| 19 | Howler.js (opt-in sound + mute control) | `https://cdnjs.cloudflare.com/ajax/libs/howler/2.2.4/howler.min.js` | 36 KB |
| 20 | colorjs.io (OKLCH runtime math) | `https://cdn.jsdelivr.net/npm/colorjs.io@0.7.1/dist/color.global.min.js` | 79 KB |
| 21 | Embla Carousel | `https://cdn.jsdelivr.net/npm/embla-carousel@8.6.0/embla-carousel.umd.js` | 18 KB |
| 22 | Number Flow (stat counters) | `https://cdn.jsdelivr.net/npm/number-flow@0.6.2/+esm` | 18 KB |
| 23 | axe-core (dev-time only, never ship) | `https://cdn.jsdelivr.net/npm/axe-core@4.13.0/axe.min.js` | 580 KB — devtools console only |
| 24 | SVGO (optimize every exported SVG) | `npx svgo@4.1.0 input.svg -o output.svg` | build-time CLI |
| 25 | fffuel + Poly Haven (design-time textures) | visit [fffuel.co](https://www.fffuel.co) / [polyhaven.com](https://polyhaven.com), export once, self-host the result | — |
