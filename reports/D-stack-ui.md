# Frontend build stack + component/asset ecosystem for award-winning landing pages — September 2026

Research snapshot date: **2026-09-15**. All version numbers below were pulled live from npm registry / official release notes during this research pass; pin exact patch versions in `package.json` and re-check before a production build since patch releases land weekly on some of these projects.

---

## 0. TL;DR — decision table

| If the job is... | Use | Why |
|---|---|---|
| One page, ships once, lives <3 months, might be emailed as a single file | **No-build HTML/CSS/JS** | Zero tooling, zero dependency rot, instant TTFB |
| One scroll-driven page needing custom cursor / magnetic buttons / carousel, no CMS | **Vite 8 + vanilla TS** | Real HMR + TS safety without a framework's routing/data opinions |
| Multi-page marketing site (home/pricing/about/blog), SEO-critical, a few interactive widgets | **Astro 7 + islands** | Ships ~0 KB JS by default; React/Vue/Svelte only where needed |
| Landing page is the front door of a real product/app (auth, dashboard behind it) | **Next.js 16** | Shares App Router, data layer, design system with the product |
| Team is Vue-native and wants Astro-like content story | **Nuxt 4** | Nuxt Content module + Vue ecosystem |
| Team is Svelte-native | **SvelteKit 2** | Smallest runtime of the "real framework" options |
| Data-heavy app with forms/mutations, not really a marketing page | **React Router 8 (framework mode)** | Loaders/actions model — wrong tool for a landing page, mention only for completeness |

For **your specific WSL2 situation** (§8): install Node LTS *inside* WSL via `nvm`/`fnm` and keep the project on the Linux filesystem (`~/dev/...`). Only fall back to invoking `/mnt/c/Program Files/nodejs/node.exe` directly when you want a zero-install one-off smoke test.

---

## 1. Meta-frameworks

### 1.1 Next.js — currently **16.3.5** (16.0 shipped 2025-10-21, ahead of Next.js Conf 2025)

Confirmed from `nextjs.org/blog/next-16` and npm registry (`next@16.3.5`, requires Node ≥20.9.0, TypeScript ≥5.1.0).

Headline changes in 16:
- **Turbopack is the default bundler**, stable for dev *and* production builds (`next build --webpack` to opt back out). 2–5× faster production builds, up to 10× faster Fast Refresh; >50% of dev sessions and >20% of prod builds on 15.3+ were already on it before the flip.
- **Cache Components** (`cacheComponents: true` in `next.config.ts`) — the "use cache" directive replaces the old implicit-caching model. All dynamic code now executes at request time **by default**; caching is opt-in per page/component/function. This finally completes the Partial Prerendering (PPR) story started in 2023. The old `experimental.ppr` / `experimental.dynamicIO` flags are gone.
- **`proxy.ts` replaces `middleware.ts`** — same code, renamed file + exported function (`export default function proxy(...)`), runs on the Node.js runtime. `middleware.ts` still works but is deprecated.
- **Async `params`/`searchParams`/`cookies()`/`headers()`/`draftMode()`** — all must be awaited now; sync access is a removed feature, not just deprecated.
- **New caching APIs**: `revalidateTag(tag, cacheLifeProfile)` (now requires a second argument — `'max' | 'hours' | 'days'` or `{expire: n}`), `updateTag()` (Server Actions only, read-your-writes), `refresh()` (Server Actions only, refreshes uncached data without touching cache).
- **React Compiler support is stable** (`reactCompiler: true`), built on React Compiler 1.0 — automatic memoization, no manual `useMemo`/`useCallback`.
- **React 19.2 canary features**: native `<ViewTransition>`, `useEffectEvent()`, `<Activity>` (background-render with `display:none` while preserving state).
- **Next.js DevTools MCP** — an MCP server exposing routing/caching/rendering state, browser+server logs, and stack traces to AI coding agents (relevant to you specifically since you're driving this from Claude Code).
- Routing overhaul: layout deduplication on prefetch (a page with 50 links to pages sharing a layout downloads that layout once, not 50 times) + incremental prefetching that only fetches what's not already cached.
- `create-next-app` simplified: App Router + TypeScript + Tailwind + ESLint by default.

```bash
npx create-next-app@latest my-landing
# or upgrade an existing app
npx @next/codemod@canary upgrade latest
```

**Verdict for a landing page**: powerful, but Cache Components + Turbopack + proxy.ts is a lot of machinery to adopt for a page that has no backend behind it. Use it when the landing page is literally a route inside a larger Next.js product.

### 1.2 Astro — currently **7.3.2** (Astro 7.0 shipped 2026-06-22; Astro 6 was a short-lived beta-only line from January 2026)

Astro's release cadence has accelerated — 6 went beta January 2026 (first-class Cloudflare Workers support, stable Live Content Collections, built-in CSP) and was superseded by 7 within the same year. Always scaffold with `@latest` rather than pinning a specific major from memory.

Astro 7 highlights (from `astro.build/blog/astro-7`, Netlify's changelog, and InfoQ coverage):
- The **`.astro` compiler was rewritten in Rust**, Markdown/MDX now run through a new Rust-powered pipeline (replacing remark/rehype as the default processor), and the rendering engine moved to a queue-based approach. Combined with the Vite 8 upgrade, builds are **15–61% faster** in Astro's own benchmarks.
- **Upgrades to Vite 8** (Rolldown-powered — see §1.5).
- **Advanced Routing**: a `src/fetch.ts` entrypoint gives full control over the request pipeline.
- **Route caching stabilized** + experimental CDN cache providers for Netlify, Vercel, and Cloudflare.
- Minimum Node version is now **22+**.

Core architecture (unchanged philosophy since Astro 5, still the reason it's the best default for a marketing landing page):
- **Islands architecture**: pages render to static HTML on the server; only explicitly marked components hydrate JS on the client. Directives control *when*:
  - `client:load` — hydrate immediately (above-the-fold interactive widget)
  - `client:idle` — hydrate when the browser is idle (`requestIdleCallback`)
  - `client:visible` — hydrate when scrolled into view (perfect for a testimonial carousel or pricing toggle far down the page)
  - `client:media="(min-width: 768px)"` — hydrate only when a media query matches (e.g. skip a desktop-only interaction on mobile)
  - `client:only="react"` — skip server render entirely, client-render only
- **Content Collections** (`src/content.config.ts`, `defineCollection()` + `glob()` loader + Zod schema): typed, validated Markdown/MDX/JSON content sets (case studies, testimonials, blog posts) with full editor autocomplete and build-time validation. As of Astro 5+/6+ collections can also pull from a headless CMS or API via custom loaders, not just local files.
- **View Transitions**: native wrapper around the browser View Transitions API for animated cross-page navigation without shipping a client-side router.
- **Server Islands**: defer rendering of a specific dynamic region so the rest of the page stays fully static/cacheable — useful for a personalized "Welcome back" strip on an otherwise static page.

```bash
npm create astro@latest my-landing
# add React only where an island truly needs it
npx astro add react tailwind
```

**Why it's usually the best choice for a marketing landing page**: you get near-zero shipped JS by default (the biggest lever for Core Web Vitals and, indirectly, SEO), content collections give you a typed CMS-lite for case studies/testimonials without a database, and you can still drop in a shadcn/React component exactly where interactivity is actually needed instead of hydrating the whole page. Astro's own selling point vs. Next.js in 2026 write-ups is precisely this "content site" niche.

### 1.3 SvelteKit — currently **2.70.3** (SvelteKit 3 is in Release Candidate as of Sept 2026, not yet stable)

`@sveltejs/kit@2.70.3` on npm; three patch releases shipped in September 2026 alone, so the 2.x line is still actively maintained while 3.0 finishes its RC cycle. Svelte's runtime is the smallest of the "real framework" options (compiles away, no virtual DOM), which matters for a landing page's JS payload if the team is Svelte-native. Not a first choice for a React-centric agency workflow, though, because almost none of the copy-paste component registries in §3 target Svelte.

### 1.4 Nuxt — currently **4.5.2** (Nuxt 3 reached end-of-life 2026-07-31)

Nuxt 4.5 (July 2026) ships **Vite 8**, an **Rspack 2 (via Rsbuild)** bundler alternative, experimental SSR streaming, a stable error-code system, a new `useLayout` composable, and groundwork for Nuxt 5. Nuxt 4's headline DX change vs Nuxt 3 is the `app/` directory restructure for cleaner organization and faster IDE performance. Nuxt Content is the closest Vue-world analogue to Astro Content Collections — pick Nuxt over Astro specifically when the team/agency is Vue-native and wants that same "content-first, SSG-by-default" story.

```bash
npx nuxi@latest init my-landing
```

### 1.5 Vite — currently **8.3.0** (Vite 8.0 shipped 2026-03-12) + vanilla TS

Vite 8 is a genuinely big release: it ships **Rolldown as the single, unified Rust-based bundler**, replacing Rollup (build) + esbuild (dev/transform) with one engine. Rolldown itself hit 1.0 stable on 2026-05-07 with a locked API. Reported gains: 10–30× faster production builds than Vite 7 in the Vite team's own benchmarks (one project went 46s → 6s). `@vitejs/plugin-react` v6 uses **Oxc** instead of Babel for Fast Refresh, shrinking install size. A compatibility layer auto-converts old `esbuild`/`rollupOptions` config into Rolldown/Oxc equivalents, so most Vite 7 configs upgrade with no changes.

Vite 7 (the prior major, June 2025) is still relevant context: it dropped Node 18, raised the default browser target to "baseline widely available" (Chrome 107+, Edge 107+, Firefox 104+, Safari 16+), and removed the legacy Sass API.

```bash
npm create vite@latest my-landing -- --template vanilla-ts
cd my-landing && npm install && npm run dev
```

**Vite + vanilla TS is the right call** when you want real component-free interactivity (a hero canvas effect, a scroll-driven timeline, a magnetic-button micro-interaction system) with hot module reload and type safety, but the page has no routing, no CMS, and won't grow into a multi-page site. `vite build` outputs plain static `dist/` — deploy it anywhere, including the no-JS-framework hosts.

### 1.6 Remix / React Router — now unified as **React Router 8.3.1**

The Remix → React Router merger completed in 2024 (Remix v2's code became React Router v7's "Framework Mode"). **React Router v8** (released 2026-06-17, npm `react-router@8.3.1`) is described by its own team as "deliberately boring": ESM-only builds, middleware enabled by default, new baselines (**Node ≥22.22.0, React ≥19.2.7, Vite ≥7**), and the `react-router-dom` package is gone — everything is `react-router` / `react-router/dom`. React Router v6 and Remix v2 are both now End-of-Life (no more security patches). It ships in three modes: plain client router, custom Data Mode, or full Framework Mode (loaders/actions/SSR).

**Not a landing-page tool.** It's a data-mutation-heavy app router. Only relevant here because a landing page sometimes lives inside a React Router app's marketing routes — in that case, use Framework Mode's static/prerendered routes for the landing pages specifically rather than fighting the loader model.

### 1.7 Decision matrix — no-build vs Vite-vanilla vs Astro vs Next

| Signal | No-build HTML | Vite + vanilla TS | Astro + islands | Next.js |
|---|---|---|---|---|
| Page count | 1 | 1 (maybe a couple) | 2–50+ | Any, esp. if behind app |
| Lifespan | Days–weeks (campaign) | Weeks–months | Months–years | Ongoing product |
| CMS / recurring content edits | No | No | Yes (Content Collections) | Yes (via CMS integration) |
| Needs client interactivity | Minimal (a `<script>` tag) | Yes, custom-built | Yes, but *localized* (islands) | Yes, app-wide |
| SEO is the #1 priority | Fine if hand-optimized | Fine | **Best default** (0 JS baseline) | Good, but more JS ships by default |
| Team already has a Next.js product | N/A | N/A | N/A | **Yes → reuse it** |
| Build tooling tolerance | Zero | Low | Medium | Higher |
| Hosting budget | Any static host, even FTP | Any static host | Any static host (or SSR adapter) | Best on Vercel |

Rule of thumb for *your* work specifically (decks/landing pages, often single-client one-offs, per your memory notes on the deck-workflow pipeline): **default to Astro** for anything that will be revisited or has more than one section worth deep-linking to, and drop to **Vite-vanilla** or **no-build** for a single hero-plus-pitch page that ships once.

### 1.8 SSG/prerender + hosting

All four frameworks above can output pure static files (`astro build`, `next build && next export`-equivalent via static export or Cache Components' static shell, `vite build`, `nuxt generate`). What differs is what the *host* optimizes for:

| Host | Best for | 2026 notes |
|---|---|---|
| **Vercel** | Next.js-centric teams | Built by the Next.js maintainers; ISR, Edge Config, Cache Components all land here first. Hobby plan: free, 100 GB bandwidth/mo, 6,000 build-minutes/mo, unlimited deployments. |
| **Netlify** | Forms/auth/split-testing without extra backend | Most "batteries-included" for a landing page with a waitlist/contact form and zero server code. Free tier build-minutes were cut 300→100/mo in 2025 — budget accordingly. Strong Astro support (official CDN cache provider added in Astro 7). |
| **Cloudflare** | Best raw price/performance, global reach | As of March 2026, **Workers has full feature parity with Pages** for static hosting, SSR, and custom domains — Cloudflare's own docs now say "start new projects with Workers," and Pages is in maintenance mode. Deploy static output via `wrangler.jsonc`: `{"assets": {"directory": "./dist", "binding": "ASSETS"}}`, `wrangler deploy`. Zero egress fees, 300+ PoPs, sub-50ms latency essentially everywhere. Best free-tier value for a personal/portfolio/agency site. |
| **GitHub Pages** | The true no-build preset | Free, dead simple, no build step needed at all if you're hand-authoring HTML. |

All three commercial hosts perform within single-digit milliseconds of each other for *pure static assets* — the differentiators are DX (Vercel wins for Next.js), included features (Netlify wins for forms), and price/global-reach (Cloudflare wins). For your RU/UA client base specifically, sanity-check actual reachability/latency from the target audience's ISPs before committing — CDN edge reachability into that region varies by provider and has shifted more than once in the last few years; a quick `curl -w` timing test from a VPN endpoint in the target city beats guessing.

---

## 2. Styling

### 2.1 Tailwind CSS v4 — currently **4.3.3**

v4.0 shipped 2025-01-22 (beta Nov 2024), v4.1 in April 2025, and the 4.2/4.3 lines through 2026 added first-party scrollbar styling, more logical-property utilities, `zoom`/`tab-size` utilities, and better `@variant` support. Confirmed via npm (`tailwindcss@4.3.3`) and the official v4 blog post.

**The core change: CSS-first configuration.** `tailwind.config.js` is gone. Everything lives in CSS:

```css
/* app.css */
@import "tailwindcss";

@theme {
  --font-display: "Bricolage Grotesque", "sans-serif";
  --breakpoint-3xl: 1920px;
  --color-brand-100: oklch(0.97 0.02 250);
  --color-brand-500: oklch(0.58 0.19 255);
  --color-brand-900: oklch(0.22 0.08 260);
  --ease-fluid: cubic-bezier(0.3, 0, 0, 1);
}

/* custom utility, same directive-driven philosophy */
@utility btn-brand {
  border-radius: var(--radius-md);
  background: var(--color-brand-500);
  padding-inline: 1.25rem;
}

/* custom variant, e.g. data-attribute dark mode toggle */
@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));

/* explicitly pull in classes used only inside a dependency */
@source "../node_modules/@my-org/ui-lib";
```

Install:

```bash
npm install tailwindcss @tailwindcss/vite   # Vite/Astro projects
# or
npm install tailwindcss @tailwindcss/postcss  # PostCSS-based pipelines (Next.js, Webpack)
```

```ts
// vite.config.ts
import tailwindcss from "@tailwindcss/vite";
export default defineConfig({ plugins: [tailwindcss()] });
```

Notable v4 features beyond CSS-first config:
- **New engine on Lightning CSS** (Rust): full rebuilds ~3.8× faster, incremental rebuilds with new CSS ~8.8× faster, incremental rebuilds with *no* new CSS ~182× faster (microsecond-range) than v3.
- **Container queries are built in**, no plugin: `<div class="@container"><div class="grid @sm:grid-cols-3 @lg:grid-cols-4">`.
- **3D transforms**: `rotate-x-*`, `rotate-y-*`, `scale-z-*`, `translate-z-*`, `perspective-*`, `transform-3d`.
- **Gradient upgrades**: angled linear gradients (`bg-linear-45`), explicit interpolation color space (`bg-linear-to-r/oklch`), conic/radial gradients with arbitrary positions (`bg-conic/[in_hsl_longer_hue]`, `bg-radial-[at_25%_25%]`).
- Dynamic utility values without config (`grid-cols-15`, `mt-8 w-17 pr-29` just work), `data-*` variants (`data-current:opacity-100`), `not-*` variant, `@starting-style` support for entry transitions, `nth-*`/`in-*` variants, `:popover-open` support.

**Migration gotchas from v3 (via `npx @tailwindcss/upgrade` codemod, handles ~80% automatically):**
- `bg-opacity-*` / `text-opacity-*` are gone — use the slash syntax (`bg-blue-500/50`).
- `hover:` is now wrapped in `@media (hover: hover)` by default — pure-touch devices won't trigger hover styles that used to fire on tap.
- v4 targets **modern browsers only**: Safari 16.4+, Chrome 111+, Firefox 128+ (it relies on native `@property` and `color-mix()`). No compatibility mode for older browsers — if you need to support them, stay on v3.4.
- v4 ships its **own dedicated PostCSS plugin** (`@tailwindcss/postcss`) rather than the old generic `tailwindcss` PostCSS plugin.
- v4 is **stricter**: classes referencing undefined theme values that silently produced no CSS in v3 now **throw build errors**. Good for catching typos, but budget time to fix every stale reference during migration.
- Realistic migration time: 1–4 hours for a small/medium project with the codemod doing the bulk of the work.

### 2.2 Vanilla CSS: custom properties + `@layer`

Still completely viable for a single landing page and often the *right* choice for the no-build/Vite-vanilla presets (§9). Pattern that scales without a framework:

```css
@layer reset, tokens, base, components, utilities;

@layer tokens {
  :root {
    color-scheme: light dark;
    --color-bg: light-dark(oklch(0.98 0.005 90), oklch(0.14 0.01 260));
    --color-fg: light-dark(oklch(0.18 0.02 260), oklch(0.95 0.005 90));
    --space-fluid-lg: clamp(2rem, 1rem + 4vw, 6rem);
  }
}

@layer components {
  .card { border-radius: 1rem; background: var(--color-bg); }
}
```

Cascade layers give you predictable specificity ordering without `!important` wars — critical once you mix hand-written CSS with a couple of copy-pasted component snippets from §3, since you can slot `@layer components` below your own overrides.

### 2.3 CSS Modules

Still the pragmatic default inside any Vite/Next/Astro/Nuxt project when you want scoped class names without committing to a utility framework or CSS-in-JS runtime — zero extra dependency (built into every one of these bundlers), works with plain CSS or Sass, and composes fine with Tailwind utilities for the 5% of styling utilities don't cover well (complex keyframe animations, `:has()`-driven state).

### 2.4 Panda CSS / vanilla-extract — both alive, both niche in 2026

- **vanilla-extract**: zero-runtime CSS-in-TypeScript — you write styles as typed `.css.ts` files, it extracts static CSS at build time. Framework-agnostic integrations for Vite, esbuild, webpack, Next.js. Good when a team wants CSS colocated with components *and* full TypeScript type-safety on design tokens, without Tailwind's utility-class verbosity in markup.
- **Panda CSS** (`npm i -D @pandacss/dev && npx panda init --postcss`): also build-time CSS generation, RSC-compatible, multi-variant "recipe" API, works across Next/Vite/Astro. Park UI (§3) is built on Panda + Ark UI as its styling layer — pick Panda specifically if you're adopting Park UI wholesale.

Neither has meaningfully displaced Tailwind v4 for landing-page work in 2026; both remain better fits for design-system-grade component libraries than for a single marketing page, where Tailwind's copy-paste-component ecosystem (§3) is simply where all the pre-built sections live.

### 2.5 Modern CSS worth using in 2026

All of the following are Baseline-widely-available or close to it as of September 2026 — safe to use without fallbacks for a landing page targeting evergreen browsers (which is a safe bet for marketing traffic):

```css
/* oklch(): perceptually uniform color, wide P3 gamut */
.badge { background: oklch(70% 0.15 145); }

/* color-mix(): programmatic tints/shades without a preprocessor */
.btn:hover { background: color-mix(in oklch, var(--brand) 85%, white); }

/* light-dark(): one declaration, both themes (needs color-scheme: light dark on :root) */
body { background: light-dark(#fafaf9, #111113); }

/* container queries: component-level responsiveness, not viewport-level */
.sidebar { container-type: inline-size; }
@container (min-width: 400px) { .card { grid-template-columns: 1fr 1fr; } }

/* :has() — parent/sibling-aware styling without JS */
.field:has(:invalid) { border-color: oklch(55% 0.2 25); }
.form:has(.field:focus-visible) .legend { opacity: 1; }

/* native nesting — no preprocessor required */
.card {
  & > .title { font-weight: 600; }
  &:hover { transform: translateY(-2px); }
}

/* @property — typed, animatable custom properties (animated gradient borders etc.) */
@property --angle {
  syntax: "<angle>";
  inherits: false;
  initial-value: 0deg;
}
.spin-border { background: conic-gradient(from var(--angle), var(--brand), transparent); animation: spin 4s linear infinite; }
@keyframes spin { to { --angle: 360deg; } }

/* subgrid — align nested grids to a parent's tracks (great for card decks with uneven content) */
.card-grid { display: grid; grid-template-columns: repeat(3, 1fr); }
.card { display: grid; grid-row: span 3; grid-template-rows: subgrid; }

/* clamp() fluid type — no media-query breakpoints for type scale */
h1 { font-size: clamp(2.25rem, 1.3rem + 3.5vw, 5.5rem); }

/* text-wrap: balance/pretty — huge, cheap typography win on headlines & body copy */
h1, h2 { text-wrap: balance; }
p { text-wrap: pretty; }

/* cascade layers — see §2.2 */
@layer reset, tokens, base, components, utilities;
```

These eight features are the actual difference between a landing page that "reads AI-generated" and one that reads hand-tuned: `text-wrap: balance` alone fixes the single most common tell (ragged, orphan-prone headlines), and `light-dark()` + OKLCH tokens are what let a dark-mode toggle look intentional instead of inverted.

---

## 3. Component libraries & copy-paste registries

The ecosystem in 2026 splits into three tiers: **primitive/headless libraries** (accessibility + behavior, zero style), **copy-paste registries** (own the code, styled with Tailwind, install via CLI), and **community grab-bags** (free-for-all, uneven quality). All the copy-paste registries below follow the shadcn pattern: you don't `npm install` the component as an opaque dependency, the CLI writes the source files into your repo and you own/edit them.

### 3.1 The primitive layer

| Library | What it is | Install | License | Notes |
|---|---|---|---|---|
| **Radix Primitives** | Unstyled, accessible React primitives (Dialog, Popover, Dropdown, etc.) | `npm i @radix-ui/react-dialog` (per-primitive packages) | MIT | Acquired by **WorkOS**; still stable and everywhere, but maintenance has visibly slowed — old issues sit unresolved. Still the safer bet if a project already has years of Radix usage. |
| **Base UI** | Unstyled React primitives from the *same engineers* who built Radix + Floating UI + Material UI, now under MUI | `npm i @base-ui-components/react` | MIT | Reached **v1.0 stable in December 2025** (35 components), full-time MUI engineering, monthly releases. As of mid-2026, **shadcn's `create` flow defaults new projects to Base UI over Radix** (ran 2-to-1 before the default even changed) — Radix is not deprecated, but Base UI is where the active development energy is now. |
| **React Aria** | Accessibility/behavior **hooks** (not styled components) from Adobe | `npm i react-aria` / `react-aria-components` | Apache-2.0 | 50+ primitives, WAI-ARIA-tested across NVDA/JAWS/VoiceOver/TalkBack, strong i18n/RTL/locale-aware date & number handling. Best when accessibility conformance is contractually required (gov/enterprise), or you need real internationalization out of the box. HeroUI v3 is now built on React Aria Components. |

### 3.2 shadcn/ui — the ecosystem hub, not just a library

`npx shadcn@latest init` writes a `components.json` describing your project's paths/aliases/styling choices, then `npx shadcn@latest add button` (etc.) pulls a component's *source* from a registry and drops it straight into your repo — no black-box npm dependency, you own and edit the code from day one. As of 2026:

- **Registry system matured**: `registry.json` lets anyone host their own component registry (with namespaces like `@acme/button`); May 2026 added `include` (composing a large registry from multiple `registry.json` files) and `shadcn registry validate`.
- **MCP server** (`ui.shadcn.com/docs/registry/mcp`): lets an AI coding agent request components by name/description, and the server resolves transitive deps, writes files, and runs your package manager for you — reads your existing `components.json` first so it matches your project's conventions. This is *exactly* the kind of tool worth wiring into a Claude Code session for landing-page component work.
- Default primitive layer is now **Base UI** (§3.1), Radix still selectable.

This registry pattern is why almost every library in §3.3–3.5 below describes itself as "shadcn-compatible" — they're all either forks of the same CLI mechanism or publish their own `registry.json`.

### 3.3 The "animated landing-page section" cluster — go here first for hero/bento/marquee/testimonial blocks

| Library | What it's *for* | Install | License | Landing-page fit |
|---|---|---|---|---|
| **Aceternity UI** | 200+ components/blocks/templates: spotlight effects, parallax, 3D cards, animated heroes | Copy-paste from `ui.aceternity.com`, or shadcn-CLI-compatible | Free tier + paid "All-Access" templates | The single most-used source for hero sections in 2025–26 AI-assisted builds — which is exactly why overusing its signature effects (spotlight cursor glow, background beams) unmodified is a top "AI slop" tell. Use 1–2 effects max, restyle colors/timing. |
| **Magic UI** | 150+ components: marquee, bento grid, globe, animated beams, text effects | `npx shadcn@latest add "https://magicui.design/r/marquee.json"` style CLI, MIT | MIT (free) + Pro $199 one-time (50+ sections, 9 templates) | **The** marquee and bento-grid source — its `Marquee` and `BentoGrid` components are the de facto standard implementations everyone else's tutorials link to. |
| **ReactBits** | 80+ animated components: text animations, backgrounds, general UI | CLI via `jsrepo` | Free + Pro tier (150 components, 280 blocks) | JS/TS and CSS/Tailwind variants of everything — good when you want the animation logic without a Tailwind dependency. |
| **Motion Primitives** | 30+ components built on Motion + Tailwind: text reveals, cursor-follow spotlight, macOS-style dock nav | CLI install, copy-paste | Open source | Smaller, more curated set — good editorial taste, less "kitchen sink" than Aceternity. |
| **Cult UI** | shadcn-compatible components + landing-page blocks (animated navbars, testimonial carousels, feature sections) | shadcn CLI-compatible registry | Free (+ Cult UI Pro, lifetime license) | Built by a solo design engineer (nolansym); more restrained animation style than Aceternity. |
| **Kokonut UI** | 100+ animated components, shadcn CLI installable | shadcn CLI-compatible | Free (+ Pro tier) | Similar niche to Cult UI; React + Tailwind + Motion. |
| **Tailark** | 300+ blocks laser-focused on **conversion surfaces**: hero, features, pricing, testimonials, FAQ, CTA, footer | Copy-paste, 4 cohesive themes (Quartz/Dusk/Mist/Veil) | Check per-license | If you specifically need a *coherent, matching set* of every section type for one landing page rather than mixing sources, Tailark's themed sets are the most internally-consistent option. |
| **Skiper UI** | 70+ "uncommon" shadcn-compatible components: scroll effects, interactive cards | Free + paid tiers | — | Marketed explicitly as *not* the default shadcn look — useful precisely to avoid the generic tell. |
| **Eldora UI / Syntax UI / Luxe** | Smaller copy-paste registries in the same genre | Copy-paste | Varies | Thinner catalogs; check current state before depending on them for anything beyond a one-off block — these move fast and some stall. |

### 3.4 Marketplaces and general registries

| Name | What it is | Notes |
|---|---|---|
| **21st.dev** | "npm for design engineers" — community marketplace/registry of shadcn-based React+Tailwind components, blocks, hooks; 1.4M developers, 200K MAU | Includes AI "remix" tooling to regenerate a component in a different style. Best used as a *search engine* across many of the libraries above rather than its own distinct design language — e.g. its Aceternity/Motion Primitives mirrors are a fast way to preview before committing. |
| **Origin UI** | Hundreds of copy-paste components following shadcn conventions | Free, MIT-style | More utility/app-UI focused (forms, tables, inputs) than landing-page hero sections — better source for the *rest* of the site (pricing forms, signup flows) than for the hero. |
| **Hover.dev** | Animated components/templates (buttons, forms, progress bars) via Framer Motion | Free + paid plans | Decent secondary source for micro-interactions. |
| **Animata** | Free, unstyled-by-default hand-crafted animations/effects | Free, copy-paste | Because it's unstyled, it blends into a custom design system better than pre-styled kits — lower "slop" risk. |
| **Uiverse.io** | Community-made library, 3,500–5,800+ pure CSS/Tailwind UI elements | MIT, free | Huge and uneven — treat as a mining ground for a specific hover-button or checkbox effect, not a coherent library. Exports to HTML/CSS, Tailwind, React, or Figma. |
| **Codrops** | Not a component library — a demo/tutorial hub (GSAP, Three.js, WebGPU, shaders) + a 2,000+-site inspiration gallery ("Webzibition") | 500+ MIT-licensed demos | This is where the genuinely *original* interaction ideas come from (scroll-driven 3D galleries, shader-based image effects) — use it for inspiration and technique, then hand-build, rather than copy-pasting a demo verbatim onto a client site. |

### 3.5 Full design-system component libraries

| Library | What it is | Install | License | Notes |
|---|---|---|---|---|
| **HeroUI** (formerly NextUI) | Full React component library, v3 (March 2026) is a ground-up rewrite: 75+ web components on **React Aria Components + Tailwind CSS v4**, OKLCH color tokens, BEM modifiers; separate React Native library (37 components) | `npm i @heroui/react` | MIT | The most "finished product" feeling of the full libraries — good when you want a complete, accessible design system fast and are willing to reskin its tokens hard to avoid the generic look. |
| **Park UI** | Components built on **Ark UI + Panda CSS**, multi-framework (React/Vue/Solid) | `npx park-ui` init | MIT | Pick this specifically if you've already committed to Panda CSS (§2.4). |

### 3.6 Which to reach for, by section

| Section | Go-to source |
|---|---|
| Hero (spotlight/parallax/3D) | Aceternity UI, Motion Primitives |
| Marquee (logo wall, testimonials) | Magic UI `Marquee`, or **pure CSS** (§6) — prefer CSS unless you need drag/pause-on-hover interaction |
| Bento grid | Magic UI `BentoGrid` |
| Testimonial carousel | Cult UI / Skiper UI blocks, or hand-build on Embla (§6) |
| Pricing table | Tailark, Origin UI |
| Footer | Tailark, Origin UI |
| Micro-interactions (buttons, hovers) | Animata, Hover.dev, Uiverse (mined individually) |

### 3.7 The "AI slop" tell list — and how to avoid it

The 2026 consensus (multiple design-criticism pieces converge on the same list) is that generic AI-assisted sites are identifiable by:

- **Unmodified shadcn defaults** — the stock zinc/slate palette, default `rounded-2xl`/`shadow-lg`/`backdrop-blur` combo applied uniformly, default border-radius scale never touched.
- **Inter as the body font**, untouched, at default weights.
- **A blue-to-purple gradient hero background** (`bg-blue-600` → `bg-purple-500`/`bg-purple-600`), often with soft blurred "gradient orb" shapes floating behind the hero — this specific combination is now the single most recognizable AI-slop signature.
- Copy that reads like the statistical average of a thousand SaaS landing pages.

**The fix** (also converged-upon): pick one real design direction (Swiss/editorial, brutalist, industrial-mono, organic/tactile, maximalist) and lock its tokens — palette (§5), fonts (§4), radius scale, texture, motion — in a short design brief *before* touching component libraries. Then: swap the default font pairing, build a custom OKLCH ramp instead of using Tailwind's defaults untouched, source real or bespoke imagery instead of the third stock photo that shows up in every demo, and restyle at least the hero and one signature section by hand rather than using any kit's block verbatim. Using Aceternity/Magic UI/shadcn *underneath* a real design system is fine and extremely common at agencies — using them *as* the design system is the tell.

---

## 4. Typography

### 4.1 Google Fonts that still read premium in 2026 — with verified Cyrillic support

Cyrillic coverage checked directly against each family's `METADATA.pb` in the `google/fonts` GitHub repo (authoritative source — this is what actually ships, unlike marketing pages). **This matters a lot for your RU/UA client work**: several of the trendiest 2026 "premium" Google Fonts have *no* Cyrillic glyphs at all.

| Family | Vibe | Variable? | Axes | Cyrillic? |
|---|---|---|---|---|
| **Manrope** | Clean geometric grotesk | Yes | wght | **Yes** |
| **Geist** (Vercel) | Neutral technical sans | Yes | wght | **Yes** |
| **Inter Tight** | Condensed Inter | Yes | wght | **Yes** |
| **Playfair Display** | High-contrast display serif | No (static weights) | — | **Yes** |
| **Unbounded** | Geometric display, Cyrillic-native | Yes | wght | **Yes** (designed Cyrillic-first) |
| **Golos Text** | Cyrillic-native workhorse grotesk (designers: Korolkova, Kuzmin) | No | — | **Yes** |
| **Onest** | Cyrillic-native modern sans (designers: Voloshin, Kudryavtsev) | Yes | wght | **Yes** |
| **Bricolage Grotesque** | Expressive variable grotesk | Yes | opsz, wdth, wght | **No** |
| **Instrument Serif** | Contemporary editorial serif | No | — | **No** |
| **Fraunces** | Editorial-to-display serif | Yes | opsz, SOFT, WONK, wght | **No** |
| **Space Grotesk** | Technical monospace-adjacent grotesk | No (5 static weights) | — | **No** |
| **Syne** | Maximalist display grotesk | No | — | **No** |
| **Newsreader** | Literary serif | Yes | opsz, wght | **No** |
| **DM Serif Display** | Compact high-contrast serif | No | — | **No** |
| **Libre Caslon Text** | Caslon revival text serif | No | — | **No** |

**Practical rule for your work**: for a Russian/Ukrainian site, shortlist from the confirmed-Cyrillic set first (Manrope, Geist, Inter Tight, Playfair Display, Unbounded, Golos Text, Onest — plus the old reliable Cyrillic-native workhorses PT Sans/PT Serif, Montserrat, and Rubik, all long-confirmed Cyrillic). If a No-Cyrillic display font (Fraunces, Instrument Serif, Syne) is non-negotiable for the brand look, restrict it to **numerals/Latin logotype only** and pair it with a Cyrillic-safe font for every word of actual Cyrillic body/headline copy — don't let Google silently fall back to a system font mid-word.

### 4.2 Free foundries beyond Google Fonts

| Source | What it offers | License | Cyrillic |
|---|---|---|---|
| **Fontshare** (Indian Type Foundry) | Free foundry-quality families: Satoshi, General Sans, Cabinet Grotesk, Clash Display/Grotesk, Switzer, Melodrama, Erode, Panchang, Gambetta, Boska, Synonym, Sentient | Free for commercial use | Mostly **Latin-only** — verify per-family before committing to a Cyrillic project |
| **Uncut.wtf** | Curated catalogue of ~160 contemporary free typefaces | Varies by family (OFL mostly) | Check per family |
| **Velvetyne (VTF)** | Collective/association of experimental and functional open-source typefaces since 2010 | OFL | Mostly Latin/experimental, a few with Cyrillic — check per family |
| **Collletttttivo** | Small curated collection of personal type-designer projects | OFL | Latin-focused |
| **The Designers Foundry, Atipo** | European independent foundries, occasional free/trial releases | Mixed | Check per family |
| **Pangram Pangram** free set | PP Neue Montreal, PP Mori, PP Object Sans, PP Supply Sans/Mono, and others offered free for commercial use | Free tier (some families) + paid full catalogue | Inconsistent — newer PP releases have started adding Cyrillic, older ones mostly don't; check each family's glyph PDF |
| **Displaay** trial fonts | Czech variable-font specialists, downloadable trial/test versions | Trial (time/weight-limited) then paid | Central-European coverage common; full Cyrillic varies by family |

### 4.3 Paid foundries agencies actually license

| Foundry | Country | Known for | Trial? | Cyrillic notes |
|---|---|---|---|---|
| **Klim Type Foundry** | NZ | Founders Grotesk, Tiempos, Söhne, Signifier, National, Calibre | Yes — downloadable test fonts | Check per-family language PDF; coverage varies across the catalogue |
| **Grilli Type** | Switzerland | GT America, GT Walsheim, GT Sectra, GT Alpina, GT Flexa, GT Super, **GT Eesti** | Yes — weight-limited trials | GT Eesti was designed with strong Cyrillic support and is popular specifically for RU/UA/Baltic branding work |
| **Pangram Pangram** | Canada | PP Neue Montreal, PP Editorial New, PP Right Grotesk, PP Fraktion Mono | Free tier doubles as a trial | Inconsistent, check per family |
| **ABC Dinamo** | Switzerland | ABC Diatype, ABC Favorit, ABC Whyte, ABC Ginto | Yes — time/weight-limited trials | **ABC Favorit** and **ABC Ginto** both have well-known Cyrillic extensions, widely used in RU-market branding |
| **Colophon Foundry** | UK | Custom + retail typefaces for major brands | Varies | Check per family |
| **Sharp Type** | USA | Reckless, Canela-adjacent serifs, National Sans | Some trials | Mostly Latin |
| **Lineto** | Switzerland | Akkurat, Aperçu | Some trials | A few families have Cyrillic; check individually |
| **Displaay** | Czech Republic | Variable-font specialists (optical-size axes as a specialty) | Yes — downloadable trial/test versions | Central-European coverage common; full Cyrillic varies by family, check before licensing |

**General rule**: foundry-wide Cyrillic support is inconsistent *even within one foundry's own catalogue* — always pull the specific family's language-coverage PDF before licensing for a Cyrillic project. Don't assume "the foundry is European so it probably has Cyrillic."

### 4.4 Variable fonts in practice

```css
/* animate weight on hover without swapping font files */
.hero-word {
  font-variation-settings: "wght" 425;
  transition: font-variation-settings 0.3s ease;
}
.hero-word:hover { font-variation-settings: "wght" 700; }

/* optical sizing — lets axis-aware fonts (Fraunces, Bricolage Grotesque both ship an opsz axis) self-adjust between display and text sizes */
h1 { font-optical-sizing: auto; }

/* fluid type scale, no breakpoints */
:root {
  --step-0: clamp(1rem, 0.95rem + 0.25vw, 1.125rem);
  --step-3: clamp(1.75rem, 1.3rem + 2vw, 2.75rem);
  --step-5: clamp(2.75rem, 1.6rem + 4.5vw, 5.5rem);
}
```

### 4.5 Self-hosting, `font-display`, subsetting, preloading

- **Self-host** via [Fontsource](https://fontsource.org) npm packages (`@fontsource-variable/manrope`) for framework-integrated builds, or download `.woff2` directly for the no-build preset — don't rely on Google's CDN for a performance-critical hero if you can bundle instead (one fewer DNS/TLS round trip).
- **`font-display`**: `swap` for body text (avoid invisible-text FOIT), consider `optional` for a non-critical display font where a fallback swap mid-read would be jarring, and preload the one weight that renders the hero headline.
- **Preloading**:
  ```html
  <link rel="preload" as="font" type="font/woff2" href="/fonts/manrope-var.woff2" crossorigin>
  ```
- **Subsetting tools**: `glyphhanger` (Zach Leatherman — crawls your built HTML/CSS and tells you the exact Unicode ranges actually used), Python `fonttools`' `pyftsubset` (the underlying engine most subsetting tools wrap; also does variable-font **axis instancing** via `fonttools varLib.instancer` — e.g. cut a full wght-axis variable font down to just the 2–3 static weights you actually use, which is often a bigger size win than glyph subsetting alone), `subfont` (automatic critical-subsetting by crawling your site).
- **For Cyrillic-heavy sites specifically**: split `@font-face` declarations by `unicode-range` so a Latin-only page doesn't download Cyrillic glyphs (and vice versa) — Cyrillic + Latin in one file roughly doubles glyph count for marginal per-page benefit if most pages are single-script:
  ```css
  @font-face {
    font-family: "Golos Text";
    src: url("/fonts/golos-latin.woff2") format("woff2");
    unicode-range: U+0000-00FF, U+0131, U+0152-0153;
  }
  @font-face {
    font-family: "Golos Text";
    src: url("/fonts/golos-cyrillic.woff2") format("woff2");
    unicode-range: U+0400-04FF, U+2116;
  }
  ```

---

## 5. Color & texture

### 5.1 OKLCH-based palette building

Build ramps by fixing hue, sweeping lightness (`L`, 0–1) and chroma (`C`) — unlike HSL, equal `L` steps in OKLCH *look* equally spaced perceptually, which is why it's replaced HSL for serious palette work. Tailwind v4's own default palette is defined in OKLCH internally (confirmed straight from the `@theme` docs example: `--color-avocado-500: oklch(0.84 0.18 117.33)`).

| Tool | What it does |
|---|---|
| **oklch.com** (Evil Martians) | Interactive OKLCH picker/converter — HEX/RGB/HSL↔OKLCH, 3D gamut visualizer, P3/Rec2020 preview with automatic sRGB fallback selection, built with Figma P3 workflows in mind |
| **Huetone** | Perceptual-contrast-driven palette builder — checks contrast *as you build* the ramp instead of after |
| **Leonardo** (Adobe) | Contrast-based, adaptive-theme color generation — good for a palette that must hit specific contrast targets across light/dark automatically |
| **Realtime Colors** | Live-preview your palette on a fake UI while you tweak — fast vibe-check, not a rigorous tool |
| **Coolors** | Fast random-palette generation, huge community database, many export formats — good for quick exploration, not perceptual rigor |
| **Radix Colors** | 46 pre-built scales × 12 steps each, tuned with **APCA** contrast (not just WCAG ratio), automatic dark mode via a class toggle, P3 gamut support, transparent alpha variant of every scale | 
| **Tailwind v4 palette** | Ships OKLCH-defined by default — extend it in `@theme` rather than replacing it wholesale unless the brand needs a fully bespoke ramp |

### 5.2 Contrast checking

**APCA** (Accessible Perceptual Contrast Algorithm) is the more perceptually accurate model for modern displays and is what Radix Colors tunes against — but **WCAG 2.1 AA/AAA ratio** is still what most contracts and accessibility audits legally require, so check both. Radix's custom palette builder and Huetone both surface contrast numbers live while you work; standalone checkers exist too (polypane, apcacontrast.com) for a final pass.

### 5.3 Film-grain / noise overlay techniques

| Technique | How | Pros | Cons |
|---|---|---|---|
| **SVG `feTurbulence`** | Inline `<svg><filter><feTurbulence type="fractalNoise" baseFrequency="0.9" numOctaves="2"/></filter></svg>`, applied via `filter: url(#grain)` or as a full-bleed `<svg>` overlay with `mix-blend-mode: overlay` | Infinitely scalable, zero HTTP request, tiny markup | GPU cost if large-area/animated; minor cross-browser rendering variance at fractional-pixel edges |
| **PNG/WebP tile** | Pre-rendered 128–256px seamless noise tile, `background-image: url(noise.webp); background-repeat: repeat;` | Perfectly consistent across browsers, cheap to render | Adds one small (~5–15 KB) HTTP request; static unless you swap frames with JS |
| **Canvas** | Procedurally drawn per-frame noise | Enables true animated/flickering film-grain | CPU/battery cost — use only for a hero moment, not the whole page |

**The recipe most top studios actually use**: a static PNG/WebP noise tile at low opacity (3–6%) with `mix-blend-mode: overlay` or `soft-light` laid over a gradient or photo background. Cheapest option, most consistent, and reads as "expensive" disproportionately to its cost.

### 5.4 Gradient mesh, duotone, dithering

- **Gradient mesh**: stack multiple `radial-gradient()` layers with `background-blend-mode: multiply`/`screen` differences, or design in Figma with a mesh-gradient plugin and export as PNG/SVG; `conic-gradient()` alone gets you painterly blob shapes cheaply.
- **Duotone**: `filter: grayscale(1)` plus a brand-colored overlay on `mix-blend-mode: color`, or an SVG `feColorMatrix` mapping shadows/highlights to two brand colors — the fastest way to make stock photography feel unified with a brand palette instead of generic.
- **Dithering**: ordered/Bayer dithering via combined `feTurbulence`+`feColorMatrix` or a small canvas pass — currently fashionable for a lo-fi/Y2K treatment on gradients, and useful practically to hide banding on large smooth gradients.
- **Paper/material textures**: same delivery mechanisms as grain (§5.3) — a tileable paper/linen/concrete photo texture at 4–10% opacity on `multiply` or `soft-light`, or a subtle `box-shadow`/`filter: drop-shadow()` stack to fake embossing/debossing on cards. Keep the tile large enough (512px+) that repetition isn't obvious on a wide hero section.

### 5.5 `background-blend-mode` vs `mix-blend-mode`

- **`background-blend-mode`** blends multiple backgrounds *within one element* (e.g., a noise texture `background-image` blended against a gradient `background-image` on the same `div`).
- **`mix-blend-mode`** blends an element against whatever is *behind it* (siblings, parent, page background).

Combos that read as expensive rather than gimmicky: `multiply` to ground a photo into a colored section band, `overlay`/`soft-light` for grain (§5.3), `difference` sparingly for a playful hover-invert on marquee text or a logo.

---

## 6. Ready-made site-level ingredients

| Need | Library | Install | Notes |
|---|---|---|---|
| Custom cursor | Hand-roll (a `position: fixed` element translated via `pointermove` + `requestAnimationFrame`, real cursor hidden with `cursor: none`) | — | No standard npm package dominates this niche; 15–30 lines is normal. `cuberto/mouse-follower` exists if you want a prebuilt one. |
| Magnetic buttons | Hand-roll on top of Motion's `useSpring`/`animate()` or GSAP's `quickTo` | `npm i motion` | Same story — a small bounding-box + proportional-translate-then-spring-back pattern, not a dedicated package. |
| Marquee | **Pure CSS** (`@keyframes` translateX loop on a duplicated flex row, paused via `prefers-reduced-motion`) preferred for perf; `react-fast-marquee` or Magic UI's `Marquee` (§3.3) if you need drag/pause-on-hover | — / `npm i react-fast-marquee` | Don't reach for JS unless you need interaction beyond autoplay. |
| Carousel | **Embla Carousel** (v9) — lightweight, plugin-based (Autoplay, WheelGestures, ClassNames), unstyled, React/Vue/Svelte/Solid/vanilla adapters, SSR-safe. **Swiper** — heavier, more batteries-included (built-in pagination/nav UI), better when you don't want to hand-build controls | `npm i embla-carousel` / `npm i swiper` | Embla = design-engineer choice for a bespoke testimonial slider; Swiper = fast full-featured product gallery. |
| Smooth scroll | **Lenis** (rebranded under Darkroom Engineering, now at `lenis.dev`) | `npm i lenis` | Pairs with GSAP ScrollTrigger or Motion's `useScroll`. Always respect `prefers-reduced-motion` and test on trackpad/touch — smooth-scroll libraries hijack native scroll and can hurt perceived performance on low-end devices if overused. |
| Split text / character stagger | `SplitType` (framework-agnostic), or GSAP's `SplitText` plugin — **now free** for everyone since GSAP went 100% free (Webflow acquisition, all Club GreenSock plugins included) | `npm i split-type` / `npm i gsap` | Used for headline reveal-on-scroll animations. |
| Animated numbers | **Number Flow** — dependency-free, Web-Components-based, `Intl`-aware (currency, compact notation), accessible/reduced-motion-aware; current best-in-class, supersedes `react-countup` | `npm i @number-flow/react` (also has a framework-agnostic core) | Great for pricing/stat counters. |
| Drawer / bottom sheet | **Vaul** (Emil Kowalski) | `npm i vaul` | React only; mobile-style sheet, shadcn's default drawer recommendation. |
| Toasts | **Sonner** (Emil Kowalski) | `npm i sonner` | React only; shadcn's default toast, replacing the old shadcn Toast primitive. |
| Particles / confetti / starfield | **tsParticles** | `npm i @tsparticles/engine @tsparticles/slim` | Huge framework coverage (React, Vue 2/3, Angular, Svelte, Astro, Solid, Qwik...). Heavier bundle — load as a `client:visible`/`client:idle` island, never blocking. |
| Hand-drawn annotations | **Rough Notation** (Preet Shihn) | `npm i rough-notation` | Underline, box, circle, highlight, strike-through, crossed-off, bracket — SVG, sketchy aesthetic, cheap way to add a human, editorial feel to a headline. |
| Page transitions | Astro's native **View Transitions** wrapper; Next.js 16 / React 19.2's native `<ViewTransition>`; Motion's `AnimatePresence` for SPA route transitions | Built into Astro/Next | Prefer the framework-native option before reaching for a separate library. |
| SVG filter tricks | `feTurbulence` + `feDisplacementMap` (liquid/glass distortion on hover), `feGaussianBlur` + `feColorMatrix` (classic "gooey" blob-merge effect), `feColorMatrix` alone (duotone, §5.4) | Inline SVG, no package | Codrops (§3.4) is the best source of working examples to adapt. |
| Lottie animation | `lottie-web`, or the much smaller **dotLottie** format via `@lottiefiles/dotlottie-web` | `npm i @lottiefiles/dotlottie-web` | After-Effects-authored JSON/dotLottie, best for autoplay/loop decorative animation. |
| Interactive vector animation | **Rive** (`.riv` format, state-machine driven) | `npm i @rive-app/react-canvas` (or `@rive-app/webgl2`) | Better than Lottie whenever the animation needs to *react* to state — hover, scroll position, form input — rather than just autoplay. |
| SVG-as-component | **`@svgr`** | `npm i -D @svgr/webpack` or Vite's `vite-plugin-svgr` | Import an `.svg` as a real React component (`import Logo from './logo.svg?react'`) so paths are stylable/animatable via props instead of an opaque `<img>`. |
| Icon sprite (non-React) | `svgo` + a `<symbol>`/`<use>` sprite sheet | `npm i -D svgo` | One HTTP request, cacheable, zero per-icon JS — the right call for the no-build/Vite-vanilla presets. |

### 6.1 Icon sets compared

| Set | Icon count | License | Signature |
|---|---|---|---|
| **Lucide** | 1,847 | ISC | Fork of Feather Icons; 24×24 grid, 2px stroke; the default pairing with shadcn/ui |
| **Phosphor** | 1,248+ | MIT | **6 weights** (Thin/Light/Regular/Bold/Fill/Duotone) — most flexible for matching a brand's exact line weight |
| **Iconoir** | ~1,600 | MIT | Similar niche to Lucide, slightly different geometric personality |
| **Remix Icon** | ~2,800+ | MIT | Line + fill pairs for every icon; popular in Chinese and global design systems |
| **Tabler Icons** | 6,184 (5,130 outline + 1,054 filled) | MIT | Largest raw count, 24×24 grid, 2px stroke |
| **Radix Icons** | ~300 | MIT | Small curated 15×15 set — pixel-matches the Radix/shadcn aesthetic exactly; best when every other primitive in the project is Radix/shadcn |

**When a custom-drawn icon set is required instead**: when the brand's visual identity depends on a distinctive geometric signature that no open set replicates (a specific corner-radius language that's part of the logo system, an unusual single-weight stroke), or when a top-tier agency deliverable needs full ownership of every mark. The common middle ground: one open set (Lucide/Phosphor) for utility chrome (nav, form icons) plus a small hand-drawn set (15–30 icons, locked grid, exported as one SVG sprite) for hero/feature icons that need to feel bespoke.

---

## 7. Images & media

### 7.1 Formats and responsive markup

- **AVIF** compresses 20–50% smaller than WebP at equivalent visual quality for photographic content, at the cost of slower encoding; **WebP** is the safe universal fallback (broad support, faster encode). Both are Baseline-supported across evergreen browsers in 2026. Serve both, oldest-format-last:

```html
<picture>
  <source type="image/avif" srcset="/img/hero-800.avif 800w, /img/hero-1600.avif 1600w" sizes="(min-width: 1024px) 50vw, 100vw">
  <source type="image/webp" srcset="/img/hero-800.webp 800w, /img/hero-1600.webp 1600w" sizes="(min-width: 1024px) 50vw, 100vw">
  <img src="/img/hero-800.jpg" alt="…" width="1600" height="900" fetchpriority="high" decoding="async">
</picture>
```

- **`srcset` + `sizes`** handle *resolution switching* (same crop, different pixel densities/viewport widths); **`<picture>`** with multiple `<source>` handles *art direction* (different crops per breakpoint) as well as format negotiation.
- **`aspect-ratio`** on the container/`img` (`aspect-ratio: 16/9`) prevents layout shift — always pair with explicit `width`/`height` attributes so the browser can compute the box before the image loads.
- **Blur-up placeholders**: a tiny (~20px) blurred version inlined as a base64 background or stacked `<img filter: blur(...)>` swapped on load. Framework-automated: Next's `next/image` `placeholder="blur"`, Astro's built-in `astro:assets` image optimization, or manual via `plaiceholder`/BlurHash for the Vite-vanilla preset.
- **`fetchpriority="high"`** on the LCP hero image only — confirmed browser support Chrome/Edge 102+, Firefox 132+, Safari 17.2+. It's a *hint*, not a directive, but Google's own case study showed Google Flights' LCP improve 2.6s → 1.9s from this one attribute. Use `fetchpriority="low"` to deprioritize below-fold carousel images competing for bandwidth with the hero.
- **`loading="lazy"`** on everything below the fold; never on the LCP image (which should be `loading="eager"` — the default — plus `fetchpriority="high"`).

### 7.2 Video

- **Containers**: MP4 (H.264) as the universal baseline `<source>`, WebM (VP9/AV1) as a lighter alternative source for browsers that support it.
- Attributes for an ambient hero background video: `muted autoplay loop playsinline` — autoplay only fires unmuted-blocked-so-always-mute, and `playsinline` is required for autoplay to work at all on iOS Safari. Always provide a `poster` frame so there's something to paint before the video buffers.
- **Transparent (alpha-channel) video**: two real options —
  - **HEVC with alpha** (`.mov`) — Safari/WebKit only, the format Apple's own marketing site uses.
  - **WebM with VP9 alpha channel** — Chrome/Firefox support, true transparency without a chroma-key hack.
  
  Serve both via two `<source>` tags and let the browser pick; when cross-browser alpha really matters and file size is a concern, a **Lottie or Rive** vector animation (§6) is often a better choice than transparent video entirely.

### 7.3 Where imagery actually comes from

- **Unsplash**: free for commercial use, no attribution required (though encouraged); may not resell raw photos or build a competing photo-library product from them.
- **Pexels**: equivalent free-commercial-use license, similar restrictions.
- **Cosmos** (mymind): a curated design-reference/moodboard search engine — treat as an *inspiration* source, not a licensed-asset source.
- **AI-generated art** (Gemini's image models, Flux, Midjourney): now routine at agencies in 2026 for hero illustrations, abstract textures, and backgrounds where stock photography would look generic — full usage rights depend on the specific plan/ToS, and disclosure requirements (EU AI Act transparency rules and platform-level policies) are increasingly a real compliance question, not just an ethical nicety, for client work.
- **How top studios actually source imagery in 2026**: commissioned photography or 3D renders (Blender/Cinema 4D) for the hero visual remains the single biggest differentiator from an "AI slop" look — stock is relegated to secondary/background use, and AI generation is treated as a *texture/mood layer* (grain, gradient blobs, abstract 3D) rather than as the hero "product shot," which still reads as synthetic to a trained eye even in 2026.

---

## 8. Practical constraint: running a Vite/Astro dev server from WSL2 with only Windows Node installed

Your situation: Windows 11 + WSL2, **no Linux-side Node**, but Windows `node.exe v24` (LTS, codename "Krypton") + `npm 11` at `/mnt/c/Program Files/nodejs/`.

### Option A — invoke Windows `node.exe`/`npm.cmd` directly from WSL bash, project stays on `/mnt/c`

This works with zero installation because WSL auto-appends the Windows `PATH` into the Linux shell's `PATH` by default (`interop.appendWindowsPath=true` in `/etc/wsl.conf`), and WSL's binfmt interop lets you exec a `.exe` directly.

```bash
# sanity check the binaries are reachable
ls "/mnt/c/Program Files/nodejs/"
"/mnt/c/Program Files/nodejs/node.exe" --version   # v24.x.x
"/mnt/c/Program Files/nodejs/npm.cmd" --version    # 11.x.x

# pin them onto PATH explicitly for this shell rather than relying on interop config
export PATH="/mnt/c/Program Files/nodejs:$PATH"

# project MUST live under /mnt/c for this option to make sense — e.g. current cwd:
cd /mnt/c/Users/ihoro/dev/my-landing   # NB: mkdir it first if it doesn't exist
npm.cmd install
npm.cmd run dev
npx.cmd create-vite@latest my-app -- --template vanilla-ts
```

Why this is *not* as bad as the usual "don't touch /mnt/c from WSL" advice implies: the 9P-protocol slowdown that WSL is infamous for (Microsoft's own tracker shows **~40 MB/s writes to `/mnt/c` from a *Linux* process vs ~442 MB/s natively** — [github.com/microsoft/WSL#4197](https://github.com/microsoft/WSL/issues/4197)) applies to *Linux processes* reading/writing through the 9P server. `node.exe` here is a **native Windows process** — once launched, it talks to `C:\Users\ihoro\...` through normal NTFS APIs, not through 9P, so file I/O and file-watching (`chokidar`/Vite's watcher use native `ReadDirectoryChangesW`) both run at full native speed with no `usePolling` workaround needed. The performance penalty of this option is *not* the WSL filesystem bridge — it's Windows Defender's real-time scanner churning through `node_modules` (tens of thousands of small files), which is worth excluding your dev folder from regardless of which option you pick.

Caveats:
- Must call the **`.cmd`/`.exe` suffix explicitly** (`npm.cmd`, `npx.cmd`, `node.exe`) — bare `npm`/`node` in bash may or may not resolve depending on WSL interop settings and whether a same-named Linux binary shadows it; explicit suffixes are the reliable path. You can alias them in `~/.bashrc` if you'll use this option regularly:
  ```bash
  alias node='"/mnt/c/Program Files/nodejs/node.exe"'
  alias npm='"/mnt/c/Program Files/nodejs/npm.cmd"'
  alias npx='"/mnt/c/Program Files/nodejs/npx.cmd"'
  ```
- npm's global packages/cache live under the **Windows** user profile (`%AppData%\npm`, `%LocalAppData%\npm-cache`) — fully separate from anything in the WSL side.
- **`localhost` access**: with WSL2's mirrored networking mode (the current default on up-to-date Windows 11), Windows and WSL share the same network namespace, so this is moot either way — but it's especially trivially true here since the dev server process *is* a Windows process, visible on `localhost:5173` from any Windows browser exactly like any other native Windows dev server, no port-forwarding involved. (You've already hit the mirrored-mode Hyper-V firewall quirk for inbound SSH per your notes — that's an *inbound-from-another-machine* issue and doesn't apply to same-machine `localhost` traffic.)
- Watch for path-quoting pain: `Program Files` has a space in it, so every reference needs quotes; if you write scripts that shell out to `cmd.exe`/PowerShell for anything, escaping between bash → cmd.exe adds a second layer of quoting to get wrong.

### Option B — install Node inside WSL (nvm or fnm), keep the project on the Linux filesystem

This is what Microsoft's own docs recommend as the general rule ("store your files in the WSL file system if you are working in a Linux command line" — [learn.microsoft.com/windows/wsl/filesystems](https://learn.microsoft.com/en-us/windows/wsl/filesystems)), and it's the option with no cross-boundary gotchas at all.

```bash
# nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.bashrc
nvm install 24        # match the Windows side's major version (v24 LTS "Krypton")
nvm use 24
nvm alias default 24
node --version         # v24.x
npm --version

# or fnm (Rust-based, faster shell startup than nvm)
curl -fsSL https://fnm.vercel.app/install | bash
source ~/.bashrc
fnm install 24
fnm use 24
fnm default 24

# keep the project OFF /mnt/c entirely
mkdir -p ~/dev
cd ~/dev
npm create vite@latest my-app -- --template vanilla-ts
cd my-app && npm install && npm run dev
# or: npm create astro@latest my-landing
```

Why this is the generally-safer long-term default:
- **File watching just works** — native `inotify` on ext4, no polling, no missed-change bugs.
- **npm install throughput** is native ext4 speed, not gated by anything cross-boundary — meaningfully faster on a `node_modules` tree with tens of thousands of files.
- `localhost:5173` from a Windows browser reaches the WSL2 dev server automatically under mirrored networking (same reasoning as Option A, just the other direction) — no extra config needed on a reasonably current Windows 11.
- **VS Code** users get the best of both worlds with the WSL Remote extension: `code .` from inside `~/dev/my-app` opens the Windows VS Code UI with a Linux-side backend/terminal — no manual `\\wsl$` UNC-path juggling required day to day (though `\\wsl$\Ubuntu\home\ihoro\dev\my-app` still works from Explorer if needed).
- **Trade-off specific to you**: per your own notes on `ext4.vhdx` growth, every byte written inside the Linux filesystem grows that VHDX file, which Windows won't auto-shrink — you're already managing this with a periodic `diskpart`/`wsl-compact.ps1` pass, so this is a known, already-mitigated cost, not a surprise.

### Recommendation

**Use Option B for anything beyond a five-minute experiment.** Install Node 24 in WSL via `nvm` (or `fnm` if you want faster shell startup — you're already running enough shell tooling that the difference is noticeable), keep the project under `~/dev/...`, and get native file-watching and native I/O with zero cross-boundary caveats — this is also the path that keeps your toolchain consistent with the rest of your WSL-centric setup (git, ssh, Vivado, per your other project notes). **Reach for Option A only** for a genuine one-off — testing that `npx create-vite@latest` scaffolds correctly, or a quick throwaway script — where you don't want to spend even the two minutes `nvm install` takes, or specifically when you want to avoid growing the `ext4.vhdx` further before your next compaction pass.

---

## 9. Three recommended stack presets

### Preset 1 — "No-build single file"

**When**: one page, ships once, or needs to be portable as a literal single `.html` file (email attachment, client preview, Claude Artifact-style delivery).

**Dependencies**: none installed locally. Everything either hand-written or loaded from a CDN `<script>`/`<link>` tag at runtime.

```
my-landing/
├── index.html          # everything: markup + <style> + <script type="module">
├── fonts/
│   ├── manrope-var.woff2
│   └── playfair-display.woff2
└── img/
    ├── hero-1600.avif
    ├── hero-1600.webp
    └── hero-1600.jpg
```

CDN references used inside `index.html` if any motion/animation is needed (no bundler, so import straight from a CDN as an ES module):

```html
<link rel="preload" as="font" type="font/woff2" href="/fonts/manrope-var.woff2" crossorigin>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital@0;1&display=swap">
<script type="module">
  import { animate, scroll } from "https://cdn.jsdelivr.net/npm/motion@13.3.0/+esm";
  animate("h1", { opacity: [0, 1], y: [20, 0] }, { duration: 0.8 });
</script>
```

Do **not** use `https://cdn.tailwindcss.com` (the Tailwind Play CDN) for anything real — it's explicitly a prototyping-only, unminified runtime build; hand-write CSS with custom properties + `@layer` (§2.2) instead. Deploy: drop the folder on literally any static host, or attach `index.html` directly.

### Preset 2 — "Vite + vanilla TS"

**When**: one scroll-driven page (or a couple, no real routing) that needs genuine interactivity — magnetic buttons, a scroll-driven reveal system, a carousel — with HMR and type safety, no framework opinions.

**Dependencies**:

```bash
npm create vite@latest my-landing -- --template vanilla-ts
cd my-landing
npm install tailwindcss @tailwindcss/vite motion lenis embla-carousel rough-notation
```

`package.json` dependencies block, pinned to what this research verified as current:

```json
{
  "devDependencies": {
    "vite": "^8.3.0",
    "typescript": "^5.7.0",
    "@tailwindcss/vite": "^4.3.3"
  },
  "dependencies": {
    "tailwindcss": "^4.3.3",
    "motion": "^13.3.0",
    "lenis": "^1.1.0",
    "embla-carousel": "^9.0.0",
    "rough-notation": "^1.0.5"
  }
}
```

```
my-landing/
├── index.html
├── vite.config.ts          # tailwindcss() plugin registered
├── tsconfig.json
├── public/
│   └── fonts/…
└── src/
    ├── main.ts              # entry: Lenis init, Motion reveals, carousel wiring
    ├── style.css            # @import "tailwindcss"; @theme { … }
    ├── cursor.ts            # custom cursor / magnetic-button logic
    └── sections/
        ├── hero.ts
        ├── testimonials.ts
        └── pricing.ts
```

```ts
// vite.config.ts
import { defineConfig } from "vite";
import tailwindcss from "@tailwindcss/vite";
export default defineConfig({ plugins: [tailwindcss()] });
```

Build/deploy: `npm run build` → static `dist/`, ship to Cloudflare Workers static assets / Netlify / GitHub Pages.

### Preset 3 — "Astro + React islands"

**When**: this is the actual "award-winning landing page" default — multi-section, SEO-critical, content that will be edited after launch (case studies, testimonials), but only a handful of things need to actually be interactive.

**Dependencies**:

```bash
npm create astro@latest my-landing -- --template minimal
cd my-landing
npx astro add react tailwind
npm install motion @number-flow/react embla-carousel-react vaul sonner lucide-react
```

`package.json` dependencies block:

```json
{
  "dependencies": {
    "astro": "^7.3.2",
    "@astrojs/react": "^4.0.0",
    "react": "^19.2.0",
    "react-dom": "^19.2.0",
    "@tailwindcss/vite": "^4.3.3",
    "tailwindcss": "^4.3.3",
    "motion": "^13.3.0",
    "@number-flow/react": "^0.5.0",
    "embla-carousel-react": "^9.0.0",
    "vaul": "^1.1.0",
    "sonner": "^2.0.0",
    "lucide-react": "^0.460.0"
  }
}
```

```
my-landing/
├── astro.config.mjs         # react() + tailwindcss() (via @tailwindcss/vite) integrations
├── src/
│   ├── content.config.ts    # defineCollection() for case studies / testimonials
│   ├── content/
│   │   ├── testimonials/*.md
│   │   └── case-studies/*.md
│   ├── layouts/
│   │   └── Layout.astro     # <head>, font preloads, view-transitions
│   ├── styles/
│   │   └── global.css       # @import "tailwindcss"; @theme { … }
│   ├── components/
│   │   ├── Hero.astro           # static
│   │   ├── LogoMarquee.astro    # static, pure-CSS marquee
│   │   ├── Footer.astro         # static
│   │   └── islands/
│   │       ├── PricingToggle.tsx     # client:visible
│   │       ├── TestimonialCarousel.tsx  # client:visible (Embla)
│   │       ├── StatCounter.tsx       # client:visible (Number Flow)
│   │       └── ContactDrawer.tsx     # client:idle (Vaul + Sonner)
│   └── pages/
│       ├── index.astro
│       ├── pricing.astro
│       └── about.astro
└── public/
    └── fonts/…
```

```astro
---
// src/pages/index.astro (excerpt)
import Layout from "../layouts/Layout.astro";
import Hero from "../components/Hero.astro";
import TestimonialCarousel from "../components/islands/TestimonialCarousel.tsx";
---
<Layout title="…">
  <Hero />
  <TestimonialCarousel client:visible />
</Layout>
```

Build/deploy: `astro build` for a fully static site (deploy anywhere in §1.8), or add `@astrojs/vercel` / `@astrojs/netlify` / `@astrojs/cloudflare` for SSR-specific features (Server Islands, personalization) if the brief ever needs them.

---

## Sources consulted (primary)

- nextjs.org/blog/next-16, nextjs.org/docs/app/guides/upgrading/version-16
- astro.build/blog/astro-7, astro.build/blog/astro-6-beta, docs.astro.build/en/concepts/islands, docs.astro.build/en/guides/content-collections
- vite.dev/blog/announcing-vite7, vite.dev/blog/announcing-vite8, voidzero.dev/posts/announcing-vite-8-beta
- nuxt.com/blog/v4, nuxt.com/blog/v4-4
- remix.run/blog/react-router-v8, remix.run/blog/react-router-v7
- tailwindcss.com/blog/tailwindcss-v4
- ui.shadcn.com/docs/registry, ui.shadcn.com/docs/registry/mcp, ui.shadcn.com/docs/installation, ui.shadcn.com/docs/changelog/2026-01-base-ui
- base-ui.com, github.com/mui/base-ui
- developers.cloudflare.com/pages, developers.cloudflare.com/workers/static-assets
- learn.microsoft.com/windows/wsl/filesystems, learn.microsoft.com/windows/wsl/interop, github.com/microsoft/WSL/issues/4197
- raw.githubusercontent.com/google/fonts (per-family METADATA.pb — Manrope, Bricolage Grotesque, Instrument Serif, Fraunces, Unbounded, Space Grotesk, Playfair Display, Syne, Newsreader, Golos Text, Onest, Inter Tight, Geist, DM Serif Display, Libre Caslon Text)
- npm registry (`next`, `astro`, `vite`, `tailwindcss`, `@sveltejs/kit`, `nuxt`, `react-router` — `latest` dist-tags)
- oklch.com, radix-ui.com/colors, klim.co.nz
- web.dev/articles/fetch-priority, web.dev/articles/compress-images
- unsplash.com/license
- motion.dev, lucide.dev, embla-carousel.com, number-flow.barvian.me, vaul.emilkowal.ski, github.com/emilkowalski/sonner, github.com/tsparticles/tsparticles, github.com/tabler/tabler-icons, github.com/phosphor-icons/homepage
