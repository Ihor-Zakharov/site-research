# site-research — offline web research for building award-grade websites

A snapshot of web research gathered on **2026-09-15** for the `site-workflow` skill (landing pages,
portfolios, institution sites at Awwwards level). It exists so an agent can answer these questions
**locally, without searching the internet**. Every claim carries its source URL.

## For an agent: how to use this

1. **Look here before you search.** If the question is covered below, answer from these files.
   Go to the web only for what is missing, or to re-check something version-sensitive.
2. **Never read a report whole** — they are 300–1600 lines each. Find the section with
   `grep -n "^#" reports/C-webgl-3d.md`, then read only that range (`sed -n '400,520p' …`).
3. **Freshness.** This is a snapshot from 15.09.2026. Concepts, patterns, jury criteria and
   techniques stay valid; **library versions, prices, star counts and "current" award winners
   go stale**. Before pinning a version in code, check npm / the official repo.
4. Cite the URL that is in the file, not the file itself, when a source matters.
5. Some notes mention `webgl.md`, `motion-system.md`, `quality-gates.md`, `taste.md` etc. — those
   are files of the private skill and are not in this repo. The facts here stand on their own.

## Index

### reports/ — nine dense reports

| File | Lines | What it answers |
|---|---|---|
| `A-awards-landscape.md` | 558 | jury criteria and weights (Awwwards, CSSDA, FWA), 86 named winning sites, 20 winning patterns and their cheap-imitation failure modes, studios, 2026 trends, 20 sites to study first |
| `B-motion-stack.md` | 1059 | GSAP 3.15 (all plugins free) with verified snippets, Motion 13, Lenis, native scroll-driven CSS, View Transitions, craft numbers from named motion designers, a working starter |
| `C-webgl-3d.md` | 1640 | three.js r186 (module-only), WebGPU/TSL, R3F/drei, a 20-entry effect catalogue with shader code, asset pipeline, performance, a no-build starter |
| `D-stack-ui.md` | 861 | framework decision matrix, Tailwind 4, 20+ component libraries judged for landing work, **which Google Fonts really have Cyrillic**, colour/texture tools, Windows/WSL + node notes, three stack presets |
| `E-claude-craft-vendor.md` | 694 | open-source Claude-design ecosystem ranked with licences; how to make a model produce non-generic design; SKILL/agent/hook examples |
| `F-quality-gates.md` | 1288 | Core Web Vitals 2026 thresholds, a pure-stdlib CDP (headless Chrome) script, a11y for motion-heavy sites, the `<head>` block, an 86-item submission checklist |
| `G-landing-anatomy-copy.md` | 586 | section anatomy per site type, hero craft, 20+ real hero headlines analysed, 12 teardowns, proof ladder, storyline template, banned words |
| `I-image-prompts.md` | 1024 | image generators compared (2026), an 11-slot prompt formula, 55 unusual image directions with templates, post-processing, archives |
| `J-opensource-toolbox.md` | 302 | creative coding, SVG, dataviz, audio, textures, CSS systems, icons, 3D assets, archives, colour, Cyrillic fonts, micro-libraries, a11y tooling + a verified 25-item shortlist |

### notes/ — three shorter follow-ups (same date)

| File | What it answers |
|---|---|
| `motion-2026.md` | scroll choreography 2025–26: sticky "hold then change state", scroll-linked counters, self-drawing lines, mobile pitfalls — each with named live examples |
| `ux-2026.md` | jury criteria precisely, what ordinary users need on a one-pager (NN/g, Baymard), CWV 2026, custom cursor / scroll-hijack verdicts |
| `field-and-antenna-gpu.md` | two GPU techniques in depth: interference field drawn as anti-aliased ink lines in a fragment shader, and a scroll-grown antenna radiation pattern |

## Quick facts that are easy to get wrong (verified 15.09.2026)

- Awwwards weights: Design 40 / Usability 30 / Creativity 20 / Content 10.
- GSAP is fully free incl. all former Club plugins (SplitText, ScrollSmoother, MorphSVG…).
- Lenis is now the plain `lenis` package. three.js is module-only (no `build/three.js`).
- Fontshare is 100 % Latin — no Cyrillic. pmndrs market is gone. Met Open Access API needs no key.
- CWV "good": LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1 (75th percentile of real visits).
