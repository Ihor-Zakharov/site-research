# Harvesting the open-source ecosystem for award-winning landing pages

Research date: 2026-09-15. Mirrors the method behind `skills/deck-workflow/library/`: clone a curated set of
third-party repos into a `library/` folder, write an `INDEX.md`, and instruct Claude to mine them every run —
but for websites/landing pages instead of decks. All star counts, licenses, and push dates below are live-fetched
from the GitHub API on 2026-09-15 (not estimates). WebSearch quota was exhausted mid-task (200/200 used on broad
landscape mapping); everything after that point was verified via `api.github.com`, `raw.githubusercontent.com`,
and WebFetch directly against primary sources — arguably higher-fidelity than search snippets anyway.

---

# HALF 1 — THE VENDOR LIST

## A. Anthropic official

**`anthropics/skills`** — github.com/anthropics/skills — 176,437★ (huge outlier: this is the public Agent Skills
repo Anthropic ships to everyone) — no repo-level license badge, but **every individual skill folder ships its own
`LICENSE.txt` = Apache-2.0** (verified by fetching `skills/frontend-design/LICENSE.txt`) — pushed 2026-09-10 — 4.7MB.
Contains 19 official skills at `skills/<name>/`: `academy-guide`, `algorithmic-art`, `brand-guidelines`,
`canvas-design`, `claude-api`, `discernment-nudge`, `doc-coauthoring`, `docx`, **`frontend-design`**,
`internal-comms`, `mcp-builder`, `pdf`, `pptx`, **`skill-creator`**, `slack-gif-creator`, **`theme-factory`**,
**`web-artifacts-builder`**, **`webapp-testing`**, `xlsx`. Also has `spec/` (the Skill file-format spec) and
`template/` (skeleton for new skills).
**Extract:**
- `skills/frontend-design/SKILL.md` — Anthropic's own design-taste instructions (full text captured below in Half 2 — this is the single highest-value file in the whole harvest).
- `skills/web-artifacts-builder/SKILL.md` + `scripts/init-artifact.sh` + `scripts/bundle-artifact.sh` — a working React18+Vite+Tailwind+shadcn/ui scaffold-and-bundle-to-single-HTML pipeline (40+ shadcn components pre-wired). Directly reusable for a "build a real React landing page, ship as one HTML file" mode.
- `skills/theme-factory/` — 10 preset color+font themes (`themes/*.md` + `theme-showcase.pdf`); its own description explicitly says it applies to "HTML landing pages." Good as a fast fallback when there's no time for full custom design-token derivation.
- `skills/brand-guidelines/SKILL.md` — not useful content-wise (it's Anthropic's own brand), but it's the **template to copy** for the user's own "extract my brand, generate a one-page brand book" pattern (already halfway built as the user's own `design-system` skill).
- `skills/canvas-design/SKILL.md` — the "write a design philosophy manifesto first, then express it visually, then do a self-critique pass" two-step method. Same underlying pattern is reusable for hero sections/generative backgrounds.
- `skills/algorithmic-art/SKILL.md` — same two-pass philosophy-then-code method, but for p5.js generative art (flow fields, particle systems, noise). Quarry for landing-page hero backgrounds/canvas art.
- `skills/skill-creator/SKILL.md` (485 lines) — meta-skill for writing more skills; useful if the user wants Claude to auto-generate new library entries.

**`anthropics/claude-code`** — github.com/anthropics/claude-code — 145,134★ — pushed 2026-09-15 (today) —
30.5MB. The CLI itself. Also ships `plugins/plugin-dev/skills/skill-development/SKILL.md` (the internal skill
Anthropic uses to build its own skills) and `.claude-plugin/marketplace.json` (a live, real-world marketplace
manifest to copy the shape of).

**`anthropics/claude-plugins-official`** — github.com/anthropics/claude-plugins-official — 36,312★ — **Apache-2.0**
— pushed 2026-09-15 — 11.4MB. The curated official plugin directory. `marketplace.json` lists plugins with a
`category` field; `category: "design"` includes e.g. `adobe-for-creativity` (Adobe's own Creative Cloud AI tools
plugin, sourced from `github.com/adobe/skills`). Worth grepping `marketplace.json` for `"category": "design"` /
`"frontend"` periodically as it grows — Anthropic adds new vendor plugins continuously.
**Note:** there's also `anthropics/claude-plugins-community` (community submissions, nightly-synced, pinned to
commit SHAs) — install via `/plugin marketplace add anthropics/claude-plugins-community`.

## B. Community skill/agent collections

**`hesreallyhim/awesome-claude-code`** — github.com/hesreallyhim/awesome-claude-code — 54,096★ — the canonical
awesome-list (not the many forks/clones that dominate naive search results). Curated links out to skills, agents,
statuslines, hooks, plugins across the whole ecosystem — use as an index, not a content source.

**`VoltAgent/awesome-claude-code-subagents`** — github.com/VoltAgent/awesome-claude-code-subagents — 25,087★ —
MIT — pushed 2026-09-14 — 870KB. 100+ ready `.md` subagents with proper frontmatter, organized by domain.
**`VoltAgent/awesome-agent-skills`** — github.com/VoltAgent/awesome-agent-skills — 34,350★ — MIT — pushed
2026-09-15 (today) — 1000+ skills aggregated from official vendors and community, cross-tool (Claude Code, Codex,
Gemini CLI, Cursor). Both are good **indexes** — browse, cherry-pick, don't bulk-vendor.

**`contains-studio/agents`** — github.com/contains-studio/agents — 12,415★ — **no license field** (repo says
"sharing current agents in use"; treat as reference/inspiration, ask before verbatim redistribution) — **stale,
last pushed 2025-07-28** — 143KB. A real product studio's actual subagent roster, organized by function:
`design/`, `engineering/`, `marketing/`, `product/`, `project-management/`, `studio-operations/`, `testing/`,
`bonus/`. `design/` has exactly 5 agents: `brand-guardian.md`, `ui-designer.md`, `ux-researcher.md`,
`visual-storyteller.md`, **`whimsy-injector.md`**.
**Extract:** `whimsy-injector.md` (full text captured in Half 2 — an excellent, complete subagent example: turns
loading/error/empty states into memorable moments, has a concrete "Animation Principles" + "Common Whimsy
Patterns" + "Anti-Patterns to Avoid" checklist) and `ui-designer.md`. These are genuinely useful as literal
subagents to drop into a landing-page-building plugin, not just examples.

**`davila7/claude-code-templates`** — github.com/davila7/claude-code-templates — 30,746★ — MIT — pushed
2026-09-15 (today) — 237MB (huge repo overall, but the relevant subtree is tiny — clone shallow or sparse-checkout
`cli-tool/components/skills/creative-design/`). This is **the single richest quarry found for this mission**.
`cli-tool/components/skills/creative-design/` contains 32 skill folders, most purpose-built for exactly
"award-winning landing pages":
- **`premium-web-design/SKILL.md`** (746 lines) — "Create premium, Awwwards-quality website designs... that look
  like they were built by a top-tier agency charging $50k+." Contains a concrete **"Blacklist — Never Do These"**
  (typography/color/layout/motion/imagery sins, verbatim captured in Half 2), a "Structural DNA Catalog", and a
  detailed protocol for sourcing/embedding topic-relevant Spline 3D scenes without generic stock-3D clichés. **The
  single best negative-constraints document found in this entire research pass.**
- **`scroll-experience/SKILL.md`** (263 lines) — parallax storytelling, GSAP ScrollTrigger + Framer Motion scroll +
  native CSS scroll-driven-animations code patterns, sticky-section and horizontal-scroll recipes, an
  "Anti-Patterns" section (scroll hijacking, animation overload, desktop-only). Frontmatter says
  `source: vibeship-spawner-skills (Apache 2.0)` — a further upstream repo worth a follow-up look.
- **`web-design-guidelines/SKILL.md`** (36 lines) — thin wrapper that live-fetches Vercel's own
  `vercel-labs/web-interface-guidelines` on every run (see below) and audits code against it in `file:line` format.
- **`ui-design-system/SKILL.md`** (32 lines) + `scripts/design_token_generator.py` — generates a full design-token
  set (color ramp, modular type scale, 8pt spacing grid, shadows, breakpoints) from one brand hex, in
  modern/classic/playful styles, exported as json/css/scss. Directly runnable design-token-first tool.
- **`tailwind-patterns/SKILL.md`** (269 lines) — Tailwind v4 CSS-first config, container queries, dark-mode
  patterns — current (v4, "2025").
- **`ui-ux-pro-max/SKILL.md`** (351 lines) — searchable database: 50 styles (glassmorphism/brutalism/bento
  grid/etc.), 21 color palettes, 50 font pairings, 20 chart types, across 9 stacks; integrates with a shadcn/ui MCP.
- Also present: `figma/`, `figma-implement-design/`, `3d-web-experience/`, `interactive-portfolio/`,
  `mobile-design/`, `accessibility-auditor/`, `imagegen/`, `luma-imagegen/`, `draw-io/`, `excalidraw/`,
  `mermaid-diagrams/`, `remotion-best-practices/`, `ux-researcher-designer/`, `executing-marketing-campaigns/`,
  plus mirrors of Anthropic's own `canvas-design`, `algorithmic-art`, `frontend-design`, `theme-factory`.
- Separately, `cli-tool/components/skills/design-to-code/SKILL.md` (top-level, not under creative-design/) is a
  screenshot/Figma → code conversion skill.
**Extract:** the whole `creative-design/` subtree (sparse-checkout or `svn export`/`degit`-style single-dir pull).

**`wshobson/agents`** — github.com/wshobson/agents — 39,680★ — MIT — pushed 2026-09-14 — 6.5MB. Restructured as a
multi-harness plugin marketplace (Claude Code, Codex, Cursor, OpenCode, Copilot, Antigravity). 92 plugins at
`plugins/<name>/`; relevant ones: **`brand-landingpage`**, `ui-design`, `frontend-mobile-development`,
`frontend-mobile-security`, `meigen-ai-design`, `database-design`. Each plugin has its own `skills/` dir plus
`.claude-plugin/` and `.codex-plugin/` manifests — good reference for **multi-harness plugin packaging** (one
plugin source, several agent-tool targets).

## C. Design AI tooling & leaked system prompts

**`superdesigndev/superdesign`** — github.com/superdesigndev/superdesign — 6,980★ — license **NOASSERTION**
(verify `LICENSE` file by hand before redistributing code) — pushed 2026-06-29 — 1.3MB. The VS Code
extension/"Cursor for design" product itself — an infinite-canvas UI generation agent. Less useful to vendor
(it's an app, not a corpus) than its sibling:
**`superdesigndev/superdesign-skill`** — github.com/superdesigndev/superdesign-skill — 560★ — **MIT** — pushed
2026-08-21 — 1.1MB. Installable via `npx skills add superdesigndev/superdesign-skill`. Explicitly positioned as
"Stop shipping AI-slop UI: turn it into shippable, tasteful frontend" — small, focused, actively maintained.

**`x1xhlol/system-prompts-and-models-of-ai-tools`** — github.com/x1xhlol/system-prompts-and-models-of-ai-tools —
143,640★ — **GPL-3.0** (note: copyleft — safe to *read and paraphrase the instruction patterns*, be careful about
verbatim redistribution inside a proprietary skill bundle) — pushed 2026-08-11 — 1.5MB. Raw, leaked/published
system prompts + tool schemas for 28+ tools: v0, Bolt, Lovable, Cursor, Devin AI, Replit Agent, Windsurf, Same.dev,
Manus, Warp.dev, Trae, and more, e.g. `Open Source prompts/Bolt/Prompt.txt`.
**Extract:** the design/aesthetic-instruction sections of the **v0** and **Lovable** prompts specifically — these
two products are the closest competitive benchmark for "AI that generates good-looking web UI," so their exact
instruction phrasing (how v0 tells itself to pick fonts, avoid generic Tailwind defaults, structure components) is
directly transferable craft language.

**`asgeirtj/system_prompts_leaks`** — github.com/asgeirtj/system_prompts_leaks — 67,107★ — **CC0-1.0** (public
domain — safest license in this whole list) — pushed 2026-09-13 — 12.1MB. Extracted prompts from Anthropic, OpenAI,
Google, xAI, and more, organized by vendor (`Anthropic/`, `OpenAI/`, `Google/`, `xAI/`, `Cursor/`, `Notion/`...).
`Anthropic/` alone has `claude-code/`, `claude-cowork/`, **`claude-design/`** (142KB `claude-design.md` — the
system prompt + tool definitions behind Claude's own "Design" canvas product, i.e. the same product family as the
`design` skill available in this session), plus per-model prompts (`claude-opus-5.md`, `claude-sonnet-5.md`,
`claude-fable-5.1.md`, etc.) and `claude-in-chrome.md`.
**Extract:** `Anthropic/claude-design/claude-design.md` — read Claude's own internal design-tool instructions
directly (large file, sample rather than ingest whole) for aesthetic-instruction language consistent with — but
more detailed than — the public `frontend-design` skill.

**`vercel-labs/web-interface-guidelines`** — github.com/vercel-labs/web-interface-guidelines — 871★ — **MIT** —
pushed 2026-08-18 — tiny (single `command.md`). Vercel's own official, terse, checklist-style interface-quality
rulebook: Accessibility, Focus States, Forms, Animation, Typography, Content Handling, Images, Performance,
Navigation & State, Touch & Interaction, Safe Areas, Dark Mode, i18n, Hydration Safety, Hover States, Content &
Copy, and a concrete **Anti-patterns** list — ends with a `file:line` audit output format. Full text captured
below (Half 2). This is designed to be fetched live and used as a **review pass** after building — pairs perfectly
with a `webapp-testing`/screenshot loop.

## D. Pattern/effect corpora (quarry — browse and cherry-pick, don't bulk-clone)

**`codrops` GitHub org** (github.com/codrops) — not one repo but 100+ small, standalone demo repos, each one
self-contained vanilla-JS/CSS/WebGL implementation of a single effect. Confirmed still active (newest repos pushed
2026). Sample of what's there with star counts (proxy for "battle-tested/copied a lot"): `PageTransitions` (2310★),
`RainEffect` (1777★), `HoverEffectIdeas` (1642★), `ParticleEffectsButtons` (1260★), `DistortedButtonEffects`
(1121★), `ElasticProgress` (876★), `CreativeButtons` (730★), `CreativeLinkEffects` (724★), `AnimatedSVGIcons`
(540★), `AnimatedGridLayout` (411★), `GradientTopographyAnimation` (352★). **Use as:** a per-need reference —
clone the one specific repo whose effect a landing page actually needs (git clone is instant, each is <1MB), read
the source, adapt. Do not bulk-vendor the whole org.

**`raunofreiberg/interfaces`** — github.com/raunofreiberg/interfaces — 1,943★ — no license listed — "A
non-exhaustive list of details that make a good web interface." Short, concrete craft checklist (e.g. "clicking
input labels should focus the input," "buttons disable after submission"). Same author's `ui-playbook` project
(uiplaybook.dev) documents common components with pitfalls; the GitHub repo for it appears effectively retired —
the live site is the more current reference.

**`argyleink/open-props`** — github.com/argyleink/open-props — 5,517★ — **MIT** — pushed 2026-08-11 — 55.6MB
(mostly docs/examples; the actual CSS is small). CSS custom-property tokens for spacing, color ramps, shadows,
easing, animation durations, gradients, aspect-ratios — an `@import`-able foundational token layer, actively
maintained.

**`trys/utopia-core`** — github.com/trys/utopia-core — 138★ — no license listed — the JS/TS calculation engine
behind utopia.fyi's fluid type/space scale interpolation (`clamp()` generation between a min and max viewport).
Small, focused, embeddable.

**`AxiomeCG/awesome-threejs`** — github.com/AxiomeCG/awesome-threejs — 983★ — CC0-1.0 — pushed 2026-07-28 — a
curated index into the WebGL/Three.js ecosystem (good jumping-off point, not content itself).
**`pmndrs/threejs-journey`** — github.com/pmndrs/threejs-journey — 810★ — no license listed — pushed 2022 (stale
but complete) — React-ported lesson code from Bruno Simon's Three.js Journey course; useful as *learning material
to extract patterns from*, not a drop-in library.

**Small, single-purpose MIT animation primitives — all real, all worth vendoring as literal dependencies:**
- `darkroomengineering/lenis` — 15,833★ — MIT — pushed 2026-09-09 (today-ish, actively maintained) — smooth-scroll
  engine; syncs with GSAP ScrollTrigger/WebGL scroll scenes. This is the modern successor to the now-retired
  `studio-freight/lenis` (Studio Freight → Darkroom Engineering rename).
- `rough-stuff/rough-notation` — 9,694★ — MIT — pushed 2024-03 (stable/finished, not actively changing) —
  hand-drawn-style annotate/highlight/underline animations.
- `shshaw/Splitting` — 1,758★ — MIT — pushed 2024-06 — splits text into words/chars/lines as CSS custom properties
  for animation; 1.5KB gzipped.
- `tsparticles/tsparticles` — 8,982★ — MIT — particle-field backgrounds, huge config surface.
- `greensock/GSAP` — 28,424★ — license field is `null` on GitHub (GSAP's own terms: **as of 2024 essentially all
  of GSAP including ScrollTrigger/SplitText is free, including commercial use** — verify current terms at
  gsap.com/licensing before shipping, don't rely solely on the GitHub license badge).
- `animate-css/animate.css` (82,789★) and `IanLunn/Hover` (29,395★) both show **NOASSERTION** on GitHub — both are
  long-standing, widely-used, historically MIT-style libraries, but confirm the actual `LICENSE` file content
  before vendoring verbatim.
- `uiverse-io/galaxy` — 12,885★ — the actual open-source repo behind uiverse.io, "largest open-source UI element
  library," community-contributed CSS/Tailwind components (buttons, cards, loaders, forms) — good for rapid
  micro-interaction prototyping, quality is uneven (community-submitted) so treat as a rough-draft source, not
  final craft.

## E. MCP servers for web design

All of the following are **Node/npx-based stdio MCP servers** except `browser-use` (Python). This session's
environment note applies directly: **WSL here has no Linux-side node; only Windows has node v24.** To use any
npx-based server from this WSL shell, either (a) call `npx.cmd`/the Windows node binary via a `cmd.exe /c npx ...`
wrapper in the MCP's `command`, (b) install a Linux nvm/node inside WSL just for this purpose, or (c) run Claude
Code itself from the Windows side for sessions that need these servers. `browser-use` needs only Python + pip and
would work natively in this WSL environment today.

| Server | Clone URL | Stars | License | Installs via | Notes |
|---|---|---|---|---|---|
| **chrome-devtools-mcp** (official, Google Chrome team) | github.com/ChromeDevTools/chrome-devtools-mcp | 52,034★ | Apache-2.0 | `npx chrome-devtools-mcp@latest` | Full CDP access: performance traces, console, network, screenshots. Best for **debugging/inspecting** a built page. |
| **playwright-mcp** (official, Microsoft) | github.com/microsoft/playwright-mcp | 37,134★ | Apache-2.0 | `npx @playwright/mcp@latest` | Cross-browser (Chromium/Firefox/WebKit) via accessibility-tree snapshots, not pixels. Best for **repeatable automation/testing**, not one-off debugging. |
| **Figma-Context-MCP** / "Framelink" | github.com/GLips/Figma-Context-MCP | 15,861★ | MIT | `npx figma-developer-mcp` + Figma personal access token | Fetches Figma file/node data, simplifies to layout+style JSON for a coding agent. Design handoff. |
| **magic-mcp** (21st.dev, "v0 in your editor") | github.com/21st-dev/magic-mcp | 5,868★ | ISC | `npx @21st-dev/magic-mcp` | Searches 10,000+ React/Tailwind components, generates new UI, publishes back to 21st.dev. Superseded by "21st MCP" — package name kept for back-compat configs. |
| **design-extract** | github.com/Manavarya09/design-extract | 4,101★ | MIT | Node 20+, uses Playwright | Extracts a **live website's entire design system** in one command: DTCG design tokens, Tailwind v4 config, Figma variables, multi-platform emitters (SwiftUI/Compose/Flutter/WordPress), CSS health audit, WCAG remediation. Directly complements/could upgrade the user's own `design-system` skill. |
| **shadcn-ui-mcp-server** | github.com/Jpisnice/shadcn-ui-mcp-server | 2,991★ | MIT | `npx shadcn-ui-mcp-server` | shadcn/ui component structure/usage/install context; React, Svelte 5, Vue, React Native. |
| **better-design** | github.com/marvkr/better-design | 233★ | MIT | npx | Design MCP + shadcn registry with 31 "brand-grade" cloned themes (Linear, Stripe, Vercel, Notion, Apple, Supabase, Figma…) + design tokens + WCAG rules. |
| **browser-use** | github.com/browser-use/browser-use | 114,698★ | MIT | `pip install browser-use` | Python agent-driven browser control — the one entry here that needs **no node at all**, runs natively in this WSL box today. |

---

## Prioritized "clone these" table (max 15, ranked by genuine value-add to this mission)

| # | Name | Clone URL | License | Size | What to extract |
|---|---|---|---|---|---|
| 1 | **anthropics/skills** | `https://github.com/anthropics/skills.git` | Apache-2.0 (per-skill) | 4.7MB | `frontend-design/SKILL.md` (the taste doctrine), `web-artifacts-builder` (React/Tailwind/shadcn scaffold+bundle scripts), `theme-factory` (10 preset themes), `algorithmic-art`+`canvas-design` (philosophy-then-express method for hero art) |
| 2 | **davila7/claude-code-templates** | `https://github.com/davila7/claude-code-templates.git` (sparse-checkout `cli-tool/components/skills/creative-design/`) | MIT | ~237MB full repo, subtree is tiny | `premium-web-design` (Awwwards Blacklist + Structural DNA Catalog), `scroll-experience`, `ui-design-system` (token generator script), `tailwind-patterns` (v4), `ui-ux-pro-max`, `web-design-guidelines` |
| 3 | **superdesigndev/superdesign-skill** | `https://github.com/superdesigndev/superdesign-skill.git` | MIT | 1.1MB | Whole skill — small, focused, purpose-built anti-AI-slop skill, actively maintained |
| 4 | **vercel-labs/web-interface-guidelines** | `https://github.com/vercel-labs/web-interface-guidelines.git` | MIT | tiny | `command.md` in full — official checklist + `file:line` audit-output convention, ideal as a post-build review pass |
| 5 | **asgeirtj/system_prompts_leaks** | `https://github.com/asgeirtj/system_prompts_leaks.git` | CC0-1.0 | 12.1MB | `Anthropic/claude-design/claude-design.md` — Claude's own internal design-tool system prompt |
| 6 | **x1xhlol/system-prompts-and-models-of-ai-tools** | `https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools.git` | GPL-3.0 (caution) | 1.5MB | v0 and Lovable prompt directories — the aesthetic-instruction language of Claude's closest competitors |
| 7 | **contains-studio/agents** | `https://github.com/contains-studio/agents.git` | none stated (ask first) | 143KB | `design/whimsy-injector.md`, `design/ui-designer.md`, `design/brand-guardian.md`, `design/visual-storyteller.md` |
| 8 | **wshobson/agents** | `https://github.com/wshobson/agents.git` | MIT | 6.5MB | `plugins/brand-landingpage/`, `plugins/ui-design/` |
| 9 | **VoltAgent/awesome-claude-code-subagents** | `https://github.com/VoltAgent/awesome-claude-code-subagents.git` | MIT | 870KB | Browse for any remaining design/frontend/QA subagents not already covered above |
| 10 | **darkroomengineering/lenis** | `https://github.com/darkroomengineering/lenis.git` | MIT | 10.2MB | The library itself — real runtime smooth-scroll dependency, not just reference |
| 11 | **tsparticles/tsparticles** + **rough-stuff/rough-notation** + **shshaw/Splitting** | (3 separate `git clone`s) | all MIT | small | Vendor as literal small dependencies for particle backgrounds / hand-drawn annotations / text-split animation |
| 12 | **argyleink/open-props** | `https://github.com/argyleink/open-props.git` | MIT | 55.6MB (mostly docs) | The CSS token files themselves — spacing/color/shadow/easing custom properties |
| 13 | **Manavarya09/design-extract** | `https://github.com/Manavarya09/design-extract.git` | MIT | — | MCP server + CLI for extracting a live site's full design-token system; pairs with the user's own `design-system` skill |
| 14 | **GLips/Figma-Context-MCP** | `https://github.com/GLips/Figma-Context-MCP.git` | MIT | 4.5MB | MCP server for Figma→code handoff when a client supplies Figma files |
| 15 | **raunofreiberg/interfaces** | `https://github.com/raunofreiberg/interfaces.git` | none stated | 421KB | Short craft checklist, cross-check against `premium-web-design`'s Blacklist for overlap/gaps |

Not table-worthy but worth remembering: the **codrops GitHub org** (100+ tiny single-effect repos) is a live quarry
to clone *individual* repos from on demand, never in bulk; `uiverse-io/galaxy` similarly is better mined per-need
than bulk-vendored (quality is uneven).

---

# HALF 2 — MAKING CLAUDE PRODUCE AWARD-GRADE DESIGN

## A. Anthropic's own published guidance

### The `frontend-design` SKILL.md (verbatim, Apache-2.0, from `anthropics/skills`)

This is Anthropic's actual shipped instruction set for design taste. Full text, because every sentence is
load-bearing:

> Approach this as the design lead at a design studio known for giving every client a distinct visual identity
> that is not mistaken for anyone else's. This client has already rejected proposals that felt cliché or
> templated, and is paying for a distinctive point of view: make deliberate, opinionated choices about palette,
> typography, and layout that are specific to this brief, and take aesthetic risk if justified.

Its **"AI-generated design right now clusters around some traits"** calibration section is the most concrete,
citable statement of what "AI slop" looks like from Anthropic itself — worth quoting exactly since a caller may
want to check new output against it:

1. a warm cream background (near `#F4F1EA`) with a high-contrast serif display and a terracotta/warm-clay accent
   (often near `#D97757` — **Anthropic's own Claude-interaction accent color**, so on a user's brief it reads as a
   tell);
2. a near-black background with a single bright acid-green or vermilion accent;
3. a broadsheet-style layout: hairline rules, zero border-radius, dense newspaper-like columns;
4. the "SaaS-card kit": content chopped into identical rounded cards, one border-radius on everything, the same
   soft grey shadow (`rgba(0,0,0,.1)`) under each, gradient washes as decoration;
5. template chrome regardless of subject: tracked-out ALL-CAPS eyebrow labels above every heading; meta strings
   joined with middle dots (`A · B · C`); labels as `WORD — fragment` with spaced em dash; tinted near-black
   (`#0B0B0B`, `#111`) standing in for true black; monospace for small data labels; a `→` appended to every link.

It also names specific **typographic tells** to avoid (accenting a single word in a headline with italic/color;
all-caps labels; unnecessary eyebrow labels above content) and a concrete **process**: (1) brainstorm a compact
token system — Color (4-6 named hex values), Type (families + roles), Layout (prose + ASCII wireframes,
alignment), Principles — (2) *review that plan against the brief* and explicitly revise anything that reads as
the generic default, only then (3) build, then (4) self-critique with actual screenshots ("a picture is worth
1000 tokens"), quoting Chanel: *"before leaving the house, take a look in the mirror and remove one accessory."*

Anthropic's two blog posts on this (`claude.com/blog/improving-frontend-design-through-skills` and
`claude.com/blog/lessons-from-building-claude-code-how-we-use-skills`) explain the *method*, not just the result:
the skill was built by **iterating with customers** on what reads as generic, and the internal principle is
*"identify convergent defaults, provide concrete alternatives, structure guidance at the right altitude, and make
it reusable through Skills."* Anthropic's internal skill taxonomy (9 categories they've observed cluster
naturally across hundreds of internal skills) rates **"product verification" — testing/verification tooling — as
the category with the single highest impact on output quality**, ahead of scaffolding or reference-doc skills.
That's a direct argument for pairing any design skill with a screenshot/`webapp-testing`-style verification loop,
not shipping the design skill alone.

### Skill-authoring guidance (`platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices`)

Full best-practices doc was fetched and is condensed into the SKILL.md example at the end of this section. Headline
rules: **assume Claude is already smart** (cut any paragraph that doesn't justify its token cost), match
**"degrees of freedom" to task fragility** (prose guidance for open-ended judgment calls, exact scripts for fragile
deterministic steps), **build evaluations before writing extensive docs**, and develop skills iteratively with
"Claude A" (skill author) + "Claude B" (skill user) in a loop.

## B. Community-proven prompting patterns for beating "AI slop" (with evidence, not folklore)

**Negative constraints / blacklists work better than positive vibes.** The single most concrete piece of evidence
in this whole research pass is `premium-web-design`'s "Blacklist — Never Do These," organized by category
(verbatim, from `davila7/claude-code-templates`):

- *Typography sins:* "Inter, Poppins, Montserrat, Raleway, Space Grotesk, Outfit as primary fonts — these scream
  'AI template'" (with the caveat that *Inter Tight* is a distinct font family and allowed); one font family for
  everything; uniform predictable hierarchy (64→32→18→14px); centered multi-line paragraph blocks.
- *Color sins:* purple-to-blue gradients ("the single biggest AI design cliché"); indigo/violet as unmotivated
  brand color; "white background + one accent + gray text (the SaaS starter kit)"; gradients on buttons/cards "for
  no reason"; neon-on-dark ("the developer portfolio look").
- *Layout sins:* "Hero → 3-column feature grid → testimonials → CTA → footer (the default SaaS landing page)" —
  and, tellingly, it also blacklists the *next-level* cliché that replaced it: "Hero → stats bar → work grid →
  about split → CTA → footer (the 'premium AI' landing page — just as formulaic)." The general rule stated:
  *"Every site must have its own unique structural DNA — different number of sections, different ordering logic."*
- *Motion sins:* fade-up-on-scroll everywhere ("the overused AOS effect"); identical transitions on everything;
  hover = scale-up-plus-shadow and nothing else.
- *Imagery sins:* blob shapes; floating geometric decoration; generic gradient-mesh backgrounds; uniform stock
  photo grids.

Independently, Vercel's `web-interface-guidelines` repo encodes the same instinct as a terse engineering checklist
rather than an aesthetic one (its own explicit **Anti-patterns** list: `user-scalable=no`, `transition: all`,
`outline-none` with no replacement, `<div onClick>` instead of `<button>`, images without dimensions, hardcoded
date/number formats instead of `Intl.*`) — the same "name the cliché explicitly so the model can avoid it" pattern
applied to code quality instead of visuals.

**Plan → critique-against-the-brief → build → self-critique is the repeated structural pattern**, appearing near-
identically in three independent sources: Anthropic's `frontend-design` (token-plan → revise-if-generic → build →
screenshot self-critique), Anthropic's `canvas-design`/`algorithmic-art` (write a named "philosophy" manifesto
first, *then* express it, with an explicit final instruction to "refine what's already there" rather than add
more), and `premium-web-design`'s explicit two-step "Blacklist check, then build." This is strong convergent
evidence that a **plan-first, explicitly-checked-against-negative-examples, then verify-with-screenshots** loop
outperforms one-shot generation — not a single write-up's opinion but the same structure independently arrived at
across Anthropic's own team and at least one large third-party skill author.

**"Make it feel like X studio" / role-anchoring is Anthropic's own opening move**, not just community folklore —
`frontend-design`'s very first sentence is the role anchor ("design lead at a design studio known for giving every
client a distinct visual identity"), and `premium-web-design` does the same thing by naming reference brands
directly in its trigger conditions (Apple, Aesop, Bottega Veneta, Stripe) so the model has concrete taste anchors
rather than the word "premium" alone.

**Design-token-first prompting has a working reference implementation**, not just a slogan: `ui-design-system`'s
`design_token_generator.py` takes one brand hex color + a style keyword (modern/classic/playful) and emits a full
color ramp, modular type scale, 8pt spacing grid, shadow/animation tokens, and breakpoints as JSON/CSS/SCSS —
i.e., "derive the whole system from a small number of committed decisions" implemented as an actual script, not
just advice.

**Multi-variant generation-then-selection** is Anthropic's own shipped pattern in `theme-factory`: generate/show
10 distinct theme options as a single PDF showcase, get an explicit human choice, *then* apply — the
generate-many/pick-one loop as a literal built-in workflow rather than aspirational advice.

*(Caveat on scope: the specific phrase "pick 3 unusual constraints" that the brief asked about was not confirmed
against a citable primary source in this pass — WebSearch quota ran out before it could be traced to an original
write-up, so it is deliberately omitted above rather than asserted on guesswork. Everything stated above is
traceable to a fetched primary source.)*

## C. Claude Code mechanics, verified against current docs (2026-09-15)

### Skills: SKILL.md, progressive disclosure, `allowed-tools`

Docs: `platform.claude.com/docs/en/agents-and-tools/agent-skills/{overview,best-practices}`,
`code.claude.com/docs/en/skills`.

- **Frontmatter required fields:** `name` (≤64 chars, lowercase+digits+hyphens only, no XML tags, can't contain
  "anthropic"/"claude"), `description` (non-empty, ≤1,024 chars, third person, states both *what* and *when*).
- **Progressive disclosure, 3 levels:** (1) at startup only `name`+`description` for *every* skill load into the
  system prompt; (2) when Claude judges a skill relevant, it reads the full `SKILL.md` body; (3) any file the body
  references is read only on demand. Bundled scripts execute via Bash without their source ever entering context.
- **Size limit:** keep `SKILL.md` body **under 500 lines**; split overflow into reference files linked **one level
  deep only** from `SKILL.md` (Claude may `head -100` a file it reaches through a second-level link and silently
  miss content — this is an explicit documented failure mode, not speculation).
- **Other frontmatter Claude Code itself recognizes** (beyond the two required ones): `license`,
  `disable-model-invocation` (user-only, for side-effecting actions), `user-invocable: false` (Claude-only,
  background knowledge), `allowed-tools` / `disallowed-tools`, `argument-hint`, `arguments` (named positional
  args), `model`, `effort`, `context: fork` (run in an isolated subagent), `agent` (which subagent type).
- **Arguments:** `$ARGUMENTS` = full string; `$0`/`$1`/… positional; named args via `arguments: [a, b]` frontmatter
  give `$a`/`$b`; dynamic context injection via `` !`shell command` `` lines that execute at invocation time.
- **File locations:** personal `~/.claude/skills/<name>/SKILL.md`, project `.claude/skills/<name>/SKILL.md`,
  plugin `plugin/skills/<name>/SKILL.md` (namespaced `/plugin:name`), legacy flat `.claude/commands/<name>.md`.

### Slash commands — now unified with Skills

As of the current docs (2026-09-15), **"slash commands" are Skills**: a plain Markdown file under
`.claude/commands/` or `.claude/skills/<name>/SKILL.md` with frontmatter, invoked as `/<name>`. The `commands/`
directory ("flat Markdown files") is explicitly documented as the **legacy** form — official guidance is *"Use
`skills/` for new plugins."* Both forms share the exact same `$ARGUMENTS`/`$0`/named-arg mechanics described above.
A minimal real example straight from the docs:

```markdown
---
name: commit
description: Stage and commit the current changes
disable-model-invocation: true
allowed-tools: Bash(git add *) Bash(git commit *) Bash(git status *)
---
```

### Subagents

Docs: `code.claude.com/docs/en/sub-agents`.

- File = YAML frontmatter + Markdown system prompt. Required: `name`, `description`. Optional (partial list,
  verified from current docs): `tools` (allowlist; omit to inherit all), `disallowedTools`, `model`
  (`sonnet`/`opus`/`haiku`/`fable`/full ID/`inherit`), `permissionMode`, `maxTurns`, `skills` (preload), `hooks`,
  `memory` (`user`/`project`/`local` — persistent `MEMORY.md` across sessions), `background`, `effort`,
  `isolation: worktree`, `color`.
- **Locations, priority high→low:** managed/org settings → `--agents` CLI flag → `.claude/agents/` (project,
  team-shared, check into VCS) → `~/.claude/agents/` (personal) → plugin `agents/`.
- **What a subagent does NOT get:** conversation history, main session's output style, previously-invoked skills,
  main session's auto-memory (unless it's a `fork`, which inherits everything).
- **When to use:** verbose/self-contained output that needs summarizing (test runs, search results), needing
  different tool restrictions or a cheaper model, parallel independent research. **When not to:** frequent
  back-and-forth, work that needs full conversation history, a quick targeted edit (main-thread is faster).

### Hooks — all event names + real JSON example

Docs: `code.claude.com/docs/en/hooks`. The full documented event list (broader than the brief's 9 — Claude Code
has grown many more): per-session `SessionStart`, `SessionEnd`, `Setup`; per-turn `UserPromptSubmit`,
`UserPromptExpansion`, `Stop`, `StopFailure`; tool-call `PreToolUse`, `PostToolUse`, `PostToolUseFailure`,
`PostToolBatch`, `PermissionRequest`, `PermissionDenied`; subagent/team `SubagentStart`, `SubagentStop`,
`TeammateIdle`, `TaskCreated`, `TaskCompleted`; context/env `InstructionsLoaded`, `ConfigChange`, `CwdChanged`,
`DirectoryAdded`, `FileChanged`; worktree `WorktreeCreate`, `WorktreeRemove`; model/context `PreModelSwitch`,
`PostModelSwitch`, `PreCompact`, `PostCompact`; display `Notification`, `MessageDisplay`; MCP `Elicitation`,
`ElicitationResult`.

**Matcher syntax:** `"*"`/empty/omitted = match all; plain letters/digits/`_`/`-`/spaces/`,`/`|` = exact string or
list (`Bash`, `Edit|Write`); anything else = unanchored JS regex (`^Notebook`, `mcp__memory__.*`). Event-specific
targets differ: tool events match tool name, `SessionStart` matches `startup`/`resume`/`clear`/`compact`/`fork`,
`Notification` matches notification type, etc.

**Real settings.json example** (condensed from the fetched doc — a block-destructive-`rm` guard):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "if": "Bash(rm *)",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/block-rm.sh",
            "timeout": 30,
            "statusMessage": "Checking for destructive commands..."
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{ "type": "command", "command": "/usr/local/bin/lint-check.sh", "timeout": 60 }]
      }
    ]
  }
}
```

`block-rm.sh`:

```bash
#!/bin/bash
COMMAND=$(jq -r '.tool_input.command')
if echo "$COMMAND" | grep -q 'rm -rf'; then
  jq -n '{hookSpecificOutput: {hookEventName: "PreToolUse", permissionDecision: "deny",
    permissionDecisionReason: "Destructive command blocked by hook"}}'
else
  exit 0
fi
```

**Exit codes:** `0` = success (stdout JSON parsed if it starts with `{`/ends with `}`); `2` = blocking (blocks the
action on blockable events — `PreToolUse`, `UserPromptSubmit`, `Stop`, `SubagentStop`, `PreModelSwitch` — stderr
becomes the reason); other codes = non-blocking error, action proceeds. Five handler `type`s exist:
`command` (shell), `http` (POST to a URL), `mcp_tool`, `prompt` (ask a fast model), `agent` (spawn a subagent to
verify). Locations: `~/.claude/settings.json` (user), `.claude/settings.json` (project, commit it),
`.claude/settings.local.json` (gitignored personal), managed policy, plugin `hooks/hooks.json`, skill/subagent
frontmatter (scoped to that invocation only).

### Plan mode

Docs: `code.claude.com/docs/en/permission-modes`. Enter with `Shift+Tab` (cycles `auto → default → acceptEdits →
plan → default`), prefix one prompt with `/plan`, or launch with `claude --permission-mode plan`. In plan mode
Claude reads files and runs read-only/classifier-approved exploration commands but **cannot edit files**; edits
stay blocked until a plan is explicitly approved. On approval you choose "Yes, and use auto mode," "Yes, manually
approve edits," or "No, keep planning." `Ctrl+G` opens the proposed plan in your editor for direct editing before
Claude proceeds. Set `"defaultMode": "plan"` in `.claude/settings.json` to make it a project's default.

### Output styles

Docs: `code.claude.com/docs/en/output-styles`. Built-in: **Default**, **Proactive** (act immediately, skip
routine-decision pauses), **Concise** (lead with result, skip preamble, full detail on request), **Explanatory**
(adds educational "Insights"), **Learning** (adds `TODO(human)` markers for the user to fill in). Change via
`/config` → Output style (writes `outputStyle` to `.claude/settings.local.json`) or set the field directly:
`{"outputStyle": "Explanatory"}`. Custom styles are Markdown files at `~/.claude/output-styles/` (user) or
`.claude/output-styles/` (project); frontmatter: `name`, `description`, `keep-coding-instructions` (default
`false` — set `true` to keep Claude Code's engineering behavior while changing tone/format), `force-for-plugin`
(plugin-only). A style applies to the main thread and to `fork` subagents only — ordinary subagents run their own
system prompt regardless of the active style.

### `/loop`

Built-in skill (available in this session's own skill list): runs a prompt or slash command on a recurring
interval, e.g. `/loop 5m /foo`; omit the interval to let the model self-pace the cadence.

### CLAUDE.md conventions

Docs: `code.claude.com/docs/en/memory`. Two complementary systems: **CLAUDE.md** (you write it — rules, standards,
architecture) and **auto memory** (Claude writes it — corrections, preferences, project context it can't derive
from code). Locations in load order (broadest→narrowest, later = read last = highest effective priority):
managed-policy path → `~/.claude/CLAUDE.md` (user) → `./CLAUDE.md` or `./.claude/CLAUDE.md` (project, VCS-shared)
→ `./CLAUDE.local.md` (personal, gitignore it). All discovered files are **concatenated**, not override-replaced;
within a directory, `CLAUDE.local.md` is appended after `CLAUDE.md`. **Target under 200 lines**; longer files
measurably reduce instruction adherence. Use `@path/to/file` import syntax (max 4 hops recursive; wrap in
backticks to cite a path *without* importing it) — `.claude/rules/*.md` for modular, optionally path-scoped
(`paths: ["src/api/**/*.ts"]` frontmatter) instructions that load only when matching files are touched. `/init`
auto-generates a starting CLAUDE.md from the codebase. Root-level project CLAUDE.md **survives `/compact`** (Claude
re-reads it from disk after compaction); nested/path-scoped rules reload only as matching files are read again.
Use a `PreToolUse` **hook**, not CLAUDE.md, for anything that must be *enforced* rather than merely *requested* —
CLAUDE.md is context, not configuration, so compliance isn't guaranteed.

### Context management — `/clear` vs `/compact`

`/compact` summarizes the conversation and continues in the same session (root CLAUDE.md and any freshly-touched
path-scoped rules reload automatically afterward; anything that was only ever stated in conversation, or lives in
a nested CLAUDE.md that hasn't reloaded yet, can be lost — re-state important instructions in CLAUDE.md to survive
this). `/clear` discards the session entirely and starts fresh context — use it between genuinely unrelated tasks
rather than `/compact`-ing across a topic change, since a full reset avoids carrying over stale/irrelevant framing.
(`code.claude.com/docs/en/context-window` renders as an interactive token-budget visualizer rather than prose —
useful to open interactively, not usefully excerptable as text here.)

### Plugin format

Docs: `code.claude.com/docs/en/plugins`. A plugin is any self-contained directory with `skills/`, `agents/`,
`hooks/`, `.mcp.json`, `.lsp.json`, `monitors/`, `bin/`, `settings.json` at its **root** — only `plugin.json`
itself goes inside `.claude-plugin/` (putting `commands/`/`agents/`/`skills/`/`hooks/` inside `.claude-plugin/` is
called out as the single most common mistake). Minimal `plugin.json`:

```json
{
  "name": "my-first-plugin",
  "description": "A greeting plugin to learn the basics",
  "version": "1.0.0",
  "author": { "name": "Your Name" }
}
```

A **marketplace** is a separate repo with `.claude-plugin/marketplace.json` at its root listing plugins by name +
`source` (a git URL, optionally `git-subdir` with `path`/`ref`/`sha` for a plugin living in a subdirectory of a
bigger repo — this is exactly the shape `anthropics/claude-plugins-official` uses to reference `adobe/skills`).
Test locally with `claude --plugin-dir ./my-plugin` (also accepts a `.zip`, or a folder-of-plugins as of v2.1.265);
apply live edits with `/reload-plugins`. Two public marketplaces exist: `claude-plugins-official` (Anthropic-
curated, auto-registered on first interactive launch) and `claude-plugins-community` (public submissions, reviewed
then pinned to a commit SHA, synced nightly).

---

## Copy-pasteable examples (all verified against current docs/live repos above)

### 1. `SKILL.md`

```markdown
---
name: landing-page-craft
description: Design-quality checklist and negative-constraints library for building award-grade HTML landing pages. Use when building, reviewing, or critiquing a marketing/landing page, portfolio, or product page — especially when the user says "make it look expensive", "not generic", or expresses dissatisfaction with a typical AI-generated look.
---

# Landing Page Craft

Approach this as the design lead at a studio that never ships the same layout twice. See `references/blacklist.md`
for the specific patterns to avoid before you start, and `references/tokens.md` for how to derive a design-token
plan from the brief.

## Workflow

1. Read `references/blacklist.md`. Note anything the brief explicitly asks for that overlaps the blacklist — the
   brief always wins.
2. Draft a compact token plan (color, type, layout, one-line principle) per `references/tokens.md`.
3. Check the plan against the blacklist. Revise anything that reads as a generic default.
4. Build.
5. Screenshot the result and self-critique against `references/blacklist.md` one more time before calling it done.

## Reference files

- **Negative constraints**: see [references/blacklist.md](references/blacklist.md)
- **Design-token derivation**: see [references/tokens.md](references/tokens.md)
```

### 2. Subagent (`.claude/agents/design-critic.md`)

```markdown
---
name: design-critic
description: Use PROACTIVELY after any landing-page HTML/CSS is written or edited, to screenshot it and critique it against the AI-slop blacklist before calling the work done.
tools: Read, Bash, Glob
model: sonnet
color: purple
---

You are a ruthless design critic at a studio that has seen every AI-generated landing page cliché. Given a path to
HTML/CSS, you:

1. Take a screenshot (desktop and mobile widths) using the project's available screenshot tooling.
2. Check it against the blacklist: generic fonts (Inter/Poppins/Montserrat/Raleway/Space Grotesk/Outfit as
   primary), purple-to-blue gradients, the SaaS-card kit (identical rounded cards + one shadow everywhere),
   predictable hero→3-col-grid→testimonials→CTA→footer structure, fade-up-on-scroll on everything, blob/floating-
   geometry decoration.
3. Report findings as `file:line — issue`, one line per finding, terse. End with a one-line verdict: SHIP or
   REVISE, and if REVISE, the single highest-priority fix.

Never rewrite the code yourself — report only. Never pass on style opinions not grounded in a specific rule above.
```

### 3. Hook (`.claude/settings.json` block)

Runs a lint/format pass automatically after every HTML/CSS edit while building a landing page, and blocks any
`rm -rf` the model attempts along the way — combines the two real patterns captured from the official hooks docs
above into one project-scoped example:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "if": "Bash(rm *)",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/block-rm.sh",
            "timeout": 30,
            "statusMessage": "Checking for destructive commands..."
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "if": "Edit(*.html) Edit(*.css) Write(*.html) Write(*.css)",
            "command": "echo 'Reminder: screenshot and check against references/blacklist.md before calling this done.' >&2",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

`.claude/hooks/block-rm.sh` (referenced above, must be `chmod +x`):

```bash
#!/bin/bash
COMMAND=$(jq -r '.tool_input.command')
if echo "$COMMAND" | grep -q 'rm -rf'; then
  jq -n '{hookSpecificOutput: {hookEventName: "PreToolUse", permissionDecision: "deny",
    permissionDecisionReason: "Destructive command blocked by hook"}}'
else
  exit 0
fi
```

### 4. Slash command (`.claude/commands/design-review.md`)

Uses the modern unified skill-as-command form (frontmatter + `$ARGUMENTS`), matching the real `web-design-
guidelines` pattern found in `davila7/claude-code-templates` (which live-fetches Vercel's guidelines on every run)
combined with the official `allowed-tools`/`argument-hint` fields:

```markdown
---
description: Review a landing page file against the AI-slop blacklist and Vercel's Web Interface Guidelines
argument-hint: [file-or-pattern]
allowed-tools: Read, Grep, Glob, WebFetch
disable-model-invocation: true
---

# Design review

Review these files for compliance: $ARGUMENTS

1. Fetch the latest guidelines: `https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md`
2. Read the specified file(s).
3. Check against every rule in the fetched guidelines, plus the local blacklist in
   `.claude/skills/landing-page-craft/references/blacklist.md`.
4. Output findings grouped by file, terse `file:line — issue` format, high signal-to-noise. If a file passes
   everything, print `✓ pass` for it instead of an empty section.
```

Usage: `/design-review src/index.html` → `$ARGUMENTS` = `src/index.html`.
