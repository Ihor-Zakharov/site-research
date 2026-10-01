# Landing page content architecture: award-winning AND converting

Research compiled 2026-09-15. Sources are live-fetched pages (marked with fetch date where the content is clearly time-sensitive) and search-aggregated advice; every claim below is attributed. Where a fetch failed (Cash App, Perplexity, OpenAI all returned HTTP 403 during research) it is noted rather than papered over with invented copy.

---

## 0. The thesis: award sites still have to say something

The tension in the brief — "wins design awards" vs. "actually converts" — is usually framed as a tradeoff. The research doesn't support that framing. It supports a different one: **award-winning sites move the ask later and prove it with craft; conversion sites move the ask earlier and prove it with repetition.** Both still have to say something specific, or the craft (or the CTA) has nothing to hang on.

Linear's own design team put this precisely in a 2026 essay about AI-generated design work:

> "Design is the search for a good fit between a form and its context." AI tools "generate plausible outputs quickly, but they do not necessarily help you understand the underlying problem." Products built this way "begin to unravel the moment you actually use them" because the fit was never worked through. "The risk is mistaking generated form for solved problems."
> — Karri Saarinen, ["Output Isn't Design"](https://linear.app/now/output-isn-t-design), Linear, Apr 2026

Same team, describing their own 2026 interface refresh:

> "Software rarely gets worse all at once. More often, it contorts out of shape one useful feature at a time." The fix was subtraction, not addition: a dimmed sidebar so "the main content area — where users work — [could] take precedence," softened borders because "structure should be felt not seen." If the changes go unnoticed, "that's probably a good sign."
> — ["A calmer interface for a product in motion"](https://linear.app/now/behind-the-latest-design-refresh), Linear, Mar 2026

That's the throughline for this whole document: restraint and specificity are the same skill in visual design and in copy. A page that "shows craft" through motion but says nothing specific in its words is not actually an award-winning page with a conversion problem — it's an unfinished page. The best Awwwards sites researched here (Obys, basement.studio, PX PUSH, MERSI, Cerebrium) all had a governing idea stated in one sentence before a single shader was written. The best conversion pages (Stripe, Linear, Attio) all have a visual identity distinctive enough to be recognized with the logo removed. The two disciplines converge; they don't trade off.

---

## 1. Section anatomy

### 1.1 The canonical inventory

Aggregating structural advice from [Web Anatomy's section taxonomy](https://www.webanatomy.ai/best-landing-pages/sections), [Involve.me's structure guide](https://www.involve.me/blog/landing-page-structure), and B2B-specific guidance, a landing page is a fixed sequence of jobs, not a fixed sequence of sections — the jobs are constant, the section that does each job varies by page type. The 19-type taxonomy Web Anatomy uses: Hero, Value Proposition, Navbar, CTA, Features, Testimonial, Pricing, Pricing Table, Problem, How It Works, Trust Signal, FAQ, Comparison, Use Case, Contact, About, Resources, Integrations, Footer.

Involve.me's recommended default sequence, and the one most conversion-page audits converge on: **Hero → USP → Benefits/Features → Social Proof → Primary CTA/Form → Objection-busters (FAQ) → Additional detail → Final CTA → Footer**, summarized as "promise → proof → action," with the explicit rule: "One page. One job." Minimize navigation and secondary links to reduce "escape hatches." ([Involve.me](https://www.involve.me/blog/landing-page-structure))

A separate synthesis (Thrive Themes / RampStack / LTL Creative aggregate) converges on 9 sections for a "strong" conversion page: hero, trust bar, problem, solution, social proof, features, secondary trust, FAQ, final CTA — with **3 to 5 primary CTAs** (same button, repeated placements), not one.

### 1.2 Section-by-section playbook

For each canonical section: its job, three layouts that work, the failure modes, and how award-winning sites subvert or skip it.

#### Hero

**Job:** confirm in the first few seconds that the visitor is in the right place, for the right reason, and give them one obvious next action (or, on narrative sites, one obvious reason to keep scrolling).

**Three layouts that work:**
1. *Headline + subhead + CTA + product screenshot* — the SaaS default (Linear, Stripe, Attio, Notion all use it in 2026).
2. *Headline as a found quote* — Arc's hero headline is literally a user testimonial ("Arc is the Chrome replacement I've been waiting for") doing double duty as proof and promise. ([arc.net](https://arc.net))
3. *No headline at all — the work is the hero* — Obys Agency's homepage opens directly on a scrolling portfolio grid with no hero copy whatsoever; craft is the pitch. ([obys.agency](https://obys.agency))

**Failure modes:** headline states a category instead of an outcome ("Sales automation software" vs. "Close deals 40% faster" — outcome headlines measurably outperform); hero visual is decorative stock photography instead of the product; CTA is invisible against a busy background; video hero causes the real content (the poster/LCP element) to load after the visitor has already decided to leave.

**How award sites subvert it:** PX PUSH replaces the headline with a device — visitors view the whole site through a simulated 1988 CRT monitor, so the "hero" is an environment, not a sentence. ([Codrops](https://tympanus.net/codrops/2026/08/07/the-department-is-open-building-the-px-push-website/)) Cerebrium replaces explanatory hero copy with an interactive 3D network visualization because the team "wanted people to *feel* how it works" rather than read a diagram. ([Codrops](https://tympanus.net/codrops/2026/07/23/building-cerebrium-making-serverless-infrastructure-tangible/))

#### Problem

**Job:** name the cost of the status quo in the visitor's own language, so the solution feels inevitable rather than sold.

**Three layouts:** (1) direct pain-point list ("delays, inefficiencies, compliance risk, revenue leakage"); (2) narrated scenario/before-state, the format testimonials also use for the same rhetorical job; (3) implicit — shown, not stated, via a "before" screenshot or metric.

**Failure modes:** manufactured problems nobody has; agitating pain without pairing it to a credible fix (reads as manipulative); skipping this section on a page where the audience doesn't yet know they have the problem (education-stage products need it most and skip it most).

**How award sites subvert it:** Portfolio and brand-campaign sites routinely skip an explicit Problem section entirely — the case study format (Obys, basement.studio) implies the "problem" (a dated brand, a flat launch) inside the *result*, never stating it as a section. Nod Coding Bootcamp's redesign is the inverse case: the *old site itself* was the problem, and the redesign's results section functions as an implicit problem statement — "traffic 6,000 → 60,000 visitors (10x), daily users 100 → 800 (8x), leads 2.9x." ([Codrops](https://tympanus.net/codrops/2024/11/26/case-study-nod-coding-bootcamp/))

#### Solution / how it works

**Job:** show the mechanism, not just the outcome — the step between "problem" and "trust me" that makes the claim credible.

**Three layouts:** (1) numbered steps (1-2-3); (2) product screenshot walkthrough / auto-playing UI loop; (3) one physical metaphor carried through the whole page (PX PUSH's CRT; Yestalgia's cassette-menu).

**Failure modes:** explaining the mechanism in engineering language instead of outcome language; a "how it works" that's actually still a feature list in disguise; too many steps (past 3-4, comprehension drops).

**How award sites subvert it:** Yestalgia (Decathlon's 90s campaign) turns the *navigation itself* into the mechanism demonstration — the menu is styled as a cassette player where "each tape represents a different destination," making the nostalgia concept "tangible without compromising simplicity" rather than explained in copy. ([Codrops](https://tympanus.net/codrops/2026/09/12/yestalgia-bringing-decathlons-90s-spirit-to-life-through-a-playful-digital-experience/))

#### Proof / logos

**Job:** transfer trust from a name the visitor already trusts to the product they don't yet.

**Three layouts:** (1) static logo strip; (2) logo strip that moves/marquees (higher engagement, lower scannability); (3) logos embedded *inside* case-study proof rather than isolated in a strip (basement.studio pairs "Selected Work" with a separate "Client Roster" — two proof layers, not one). ([basement.studio](https://basement.studio))

**Failure modes:** a logo wall is not proof of a relationship, it's proof of contact — and buyers have caught on. A dedicated argument on this: "Every sophisticated buyer has now accumulated the experience of seeing logo walls everywhere... the pattern has been so overused that it has lost its semantic weight entirely. When a trust signal becomes universally expected, it stops being a signal at all. It becomes visual wallpaper." ([Webifii](https://webifii.co.in/social-proof-strategy-logo-wall-conversions/))

**How award sites subvert it:** they skip the strip and go straight to named, dated case studies with numbers — see §5.1 for the five-level replacement model.

#### Features

**Job:** map capabilities to outcomes for the specific buyer reading right now, without becoming an inventory.

**Three layouts:** (1) 3-column icon grid (the default, also the most forgettable); (2) alternating image/text "feature story" rows, one per scroll beat; (3) tabbed/interactive feature switcher (Attio: "SDK. API. MCP. Build anything on Attio").

**Failure modes:** every feature gets equal visual weight, so none reads as the reason to buy; features are named by their internal engineering name, not the outcome; the grid becomes an unreadable wall past 6-9 items.

**How award sites subvert it:** they rename the section entirely around a persona or a verb instead of "Features" — see §3.4.

#### Testimonials

**Job:** let someone other than the brand make the claim.

**Three layouts:** (1) card grid with headshots; (2) single large rotating quote with name/title/company; (3) embedded inside a numbered case study rather than isolated (see §5.2).

**Failure modes:** stock-photo faces; quotes so generic they could be swapped between any two competitors; quotes with no attribution past a first name.

**How award sites subvert it:** Oura frames its proof as narrative rather than quote-card: "Real members, real stories of impact," organized by health outcome (Heart Health, Symptom Radar, Women's Health) rather than by customer name. ([ouraring.com](https://ouraring.com))

#### Pricing

**Job:** let the visitor self-select a tier without leaving to do math, and make the tier you want chosen feel obviously correct.

**Three layouts:** (1) 3-column comparison table (converts best — "three-tier pricing pages convert 31% better than pages with 4+ tiers," per [Kinde](https://www.kinde.com/learn/billing/pricing/building-a-pricing-table-that-converts-best-practices-for-saas/) and cross-referenced by [Digital Applied](https://www.digitalapplied.com/blog/subscription-pricing-page-psychology-decision-framework-2026)); (2) slider/calculator for usage-based pricing; (3) single "talk to sales" gate for enterprise-only products (Vercel, Attio both keep "Talk to sales" as a parallel CTA next to self-serve).

**Psychology at work:** *anchoring* — a deliberately expensive top tier makes the middle tier look reasonable even when almost nobody buys the top tier; the *decoy effect* — a deliberately worse option added purely to redirect choice toward the target tier. These are distinct mechanisms, often conflated. ([Digital Applied](https://www.digitalapplied.com/blog/subscription-pricing-page-psychology-decision-framework-2026))

**Failure modes:** hidden pricing on a page that could show it (erodes trust, doesn't protect margin); feature-matrix rows so dense that comparison becomes work; anchoring so obvious it reads as manipulation.

**How award sites subvert it:** PX PUSH's pricing section is modeled as a physical floppy disk — "literally referencing physical software packages" — folding the page's governing 1988-computer concept into the one section every other SaaS site treats as pure utility. ([Codrops](https://tympanus.net/codrops/2026/08/07/the-department-is-open-building-the-px-push-website/))

#### FAQ

**Job:** pre-answer the objections that would otherwise become support tickets or silent bounces.

**Three layouts:** (1) accordion list; (2) FAQ folded into the pricing table as footnotes; (3) FAQ as a searchable/chat entry point.

**Failure modes:** FAQs that are actually marketing copy in question format ("Why is [product] the best choice?"); FAQs that dodge the real objection (price, lock-in, data ownership) in favor of easy softballs.

**How award sites subvert it:** most narrative/portfolio sites skip FAQ entirely — Obys and basement.studio have none, because the "purchase decision" (hiring an agency) doesn't happen self-serve on the page. FAQ is a self-serve-commerce section; it appears on Family (crypto wallet, self-serve, real objections about security) and disappears on studio sites (relationship-sold, no self-serve step to de-risk).

#### CTA

**Job:** give the single next action in words that describe the visitor's result, not the system's action.

**Three layouts:** (1) repeated identical button at 3-5 scroll points (the B2B-recommended default — one dominant action, secondary actions for different readiness levels); (2) two parallel CTAs for two buyer types (Attio: "Start for free" / "Talk to sales" / "Send me a demo" — three, actually, split by intent); (3) CTA disguised as content, e.g., Raycast's CTA button literally reads "The Raycast Keyboard" — a product name, not an instruction.

**Failure modes:** "Submit" (describes the system, not the person); a single generic "Learn More" repeated everywhere with no variation for context; CTA copy that doesn't match what happens after the click (mismatch spikes bounce on the next screen).

**Real CTA copy that isn't "Get Started":** "Deploy now" / "Talk to sales" (Vercel); "Start for free" / "Send me a demo" (Attio); "Get Notion free" / "Request a demo" (Notion); "Download Arc for Windows" (Arc — platform-specific, removes a decision); "Try Claude" (Anthropic); "Explore" (Oura); "Get Superhuman" (Superhuman — branded verb).

#### Footer

**Job:** the last trust check and the last utility layer — legal, contact, sitemap — for the visitor who scrolled all the way down precisely because they weren't convinced yet.

**Three layouts:** (1) maximalist/regulated — Monzo's footer carries site information, an "Authorised push payment (APP) fraud rankings" disclosure, and an independent service-quality survey result, because UK banking marketing is legally required to disclose comparative fraud/quality data; (2) minimalist link list; (3) footer-as-brand-moment — Obys's footer shows a live clock in CET next to the mission line "The studio is shaped by people who care deeply about design and the process behind..." — turning the most templated section on the internet into one more craft signal. ([obys.agency](https://obys.agency), [monzo.com](https://monzo.com))

**How award sites subvert it:** PX PUSH turns the *error* pages adjacent to the footer layer into a brand moment instead — 404/500 pages recreate the Windows "Blue Screen of Death," in the exact hex `#03049C`, rather than treating them as dead ends. ([Codrops](https://tympanus.net/codrops/2026/08/07/the-department-is-open-building-the-px-push-website/))

### 1.3 Landing type variations

| Type | Typical section order | Real example |
|---|---|---|
| **Product/SaaS** | Hero (headline+CTA+screenshot) → Trust bar → Problem → Solution/how it works → Feature stories → Proof/case studies → Pricing → FAQ → Final CTA → Footer | Attio, Linear, Notion — see §3.5 |
| **Agency/portfolio** | Cold-open hero or none → Work grid (proof and hero fused) → Capabilities/services → Client roster → About/studio context → Contact | Obys ("work" opens the page); basement.studio (value prop → selected work → services → client roster → about → contact) |
| **Launch/teaser** | Cryptic headline → single visual/video → email capture → countdown (optional) → social links | General pattern per [Landingi](https://landingi.com/landing-page/coming-soon-examples/) and [KickoffLabs](https://kickofflabs.com/blog/6-great-coming-soon-pages-for-next-product-launch/): "just enough taste of what's coming," cryptic messaging over full disclosure |
| **App (mobile)** | Hero with one real device screenshot → store badges → 3-5 benefit sections → short walkthrough → proof/reviews → FAQ → support/privacy links | Family (iOS wallet): explore → five one-word benefit heads (Easy/Secure/Fast/Powerful/Fun) → FAQ → download |
| **Event/conference** | Hero (event name + what it's for) → schedule/speakers → registration CTA → past-year recap/social proof → footer | Figma Config: "Figma's conference for people who build products" — thin outside the live event window, recap-only post-event |
| **Physical product** | Hero (product beauty shot, minimal copy) → category benefit sections → proof/reviews with numbers → comparison/spec → payment options → newsletter | Oura: "Subtle. Power." → "Understand your body. Own your health." → per-category sections (Sleep, Heart, Women's Health) → "86% of Oura Members see their health improve" → payment options |
| **AI startup** | Hero (agent/outcome framing) → "how the agent works" demo → integrations/ecosystem → proof (logos or usage stat) → pricing/API → changelog | Cursor: "Cursor is your coding agent for building ambitious software" → "Agents turn ideas into code" → "Works autonomously, runs in parallel" → changelog |
| **B2B/enterprise** | Hero (who it's for + problem it solves) → context/problem in operational language → solution → social proof with outcomes → multi-stakeholder proof → long-form detail → gated demo CTA | Datadog: "AI-Powered Observability and Security" → product-line grid → Gartner Magic Quadrant banner → "Thousands of customers love & trust Datadog" |

AI-startup sites in particular converge on a narrow visual grammar right now: "clean layouts, subtle motion, clear product storytelling... restraint throughout most of the page and one deliberate moment of visual impact, usually in the hero or in a product demonstration section," frequently dark-mode-only. ([ebaqdesign.com](https://www.ebaqdesign.com/blog/ai-startup-landing-page-examples)) That convergence is itself a trap worth naming to a client: it reads as "current" today and generic within 18 months, the same fate that befell the 2021 gradient-blob SaaS look.

### 1.4 The one-page Awwwards narrative vs. the conversion-optimised SaaS page

The brief's five-beat Awwwards structure — **intro → reveal → demonstration → proof → invitation** — maps cleanly onto what the teardowns actually do:

- **Intro**: a cold open that sets mood before it sets expectations. PX PUSH's boot sequence, Goodgrowth's literal boot-up animation, Noomo's shader/3D-phoenix hero. No CTA yet, often no words yet.
- **Reveal**: the governing concept declares itself. PX PUSH reveals "you're looking at this through a CRT"; Cerebrium reveals the shader-driven network paths represent live data flow.
- **Demonstration**: the visitor is handed the interaction. Cerebrium's mouse-tracked security shield; Trionn's "hold to touch the lines" hero prompt — note the copy is one line: "Dare ⚡ to touch the lines," description subordinated entirely to interaction.
- **Proof**: portfolio or case-study evidence, usually the first place real client names or numbers appear — Obys's 19-project grid, basement.studio's "Selected Work."
- **Invitation**: a single low-pressure contact point, not a repeated CTA. No pricing table, usually no FAQ.

A conversion-optimised SaaS page inverts the pacing entirely: **ask first, prove continuously.** The CTA appears in the hero, not at the end; it repeats 3-5 times rather than once; proof is distributed throughout rather than saved for one beat; FAQ and pricing — sections the narrative structure treats as unnecessary — are load-bearing because the visitor is expected to *transact*, not just be *impressed*. The B2B research above puts the mechanism plainly: "One primary CTA should dominate, with secondary actions supporting different readiness levels" — repetition of the ask, not a single reveal of it.

The practical rule for a page that wants both awards and conversions: **use the five-beat pacing for mood and trust-building, but don't withhold the ask as long as a pure portfolio site can.** Award judges reward the intro/reveal/demonstration craft; buyers need the ask restated near every proof point. Concretely: keep the cold-open hero (intro/reveal), but put a real CTA in it too; keep one interactive demonstration beat; compress "proof" and "invitation" so the final CTA arrives no later than a portfolio site's proof section would.

---

## 2. Hero craft

### 2.1 The first three seconds

The visitor decides three things almost simultaneously: *what is this, is it for me, do I trust it enough to keep looking.* The mechanism that most often fails this test isn't copy — it's performance. "Autoplay hero videos are the #1 cause of LCP [Largest Contentful Paint] failures on visually-rich websites... The poster image should be the LCP element (always), with video as progressive enhancement." A hero that hasn't rendered by the time the visitor has formed a judgment has already lost regardless of what the headline says. ([Mintec](https://mintec.co/blog/video-lcp-hero-performance-2026/))

Once it has rendered, the job is to answer *what/for-whom/why-now* without making the visitor work. B2B guidance frames it as: "who the offer is for, what business problem it solves, and why the solution is practical now" — three answers, one headline+subhead pair.

### 2.2 Headline formulas that are not clichés

Generic-sounding advice ("How to X without Y," "[Who] + [what they get] + [how fast]") is directionally correct but the real lesson from the research is sharper: **specificity beats cleverness at a rate that should embarrass most creative headline writing.** After testing over 150,000 opt-in headlines, straightforward, specific headlines out-performed creative alternatives 88% of the time. CityCliq's rewrite from "Businesses Grow Faster Online" (abstract, could be any product) to "Create a Webpage for Your Business" (literal, one action) lifted conversion 90%. ([Woobox](https://woobox.com/articles/landing-page-headline-formulas), [Unbounce](https://unbounce.com/landing-pages/5-headline-formulas/), [KlientBoost](https://www.klientboost.com/landing-pages/landing-page-headlines/))

The rule that generalizes: **name the outcome, not the category.** "Close deals 40% faster" beats "Sales automation software." The live-fetched headlines below (§3.5) are almost all outcome-or-identity statements, not category labels — nobody's hero says "Project management software" anymore; Basecamp's says "the refreshingly straightforward project management system that's rock-solid and easy to use," which is still a category name, but modified by three specific, contrarian adjectives that argue against the category's reputation.

### 2.3 The words that scream "AI wrote this"

This list is compiled from four independent word-list audits plus what actually showed up (or conspicuously didn't) in the sites fetched for this research: [ContentBeta's 300+ list](https://www.contentbeta.com/blog/list-of-words-overused-by-ai/), [FOMO's prompt-based list](https://fomo.ai/ai-resources/the-ultimate-copy-paste-prompt-add-on-to-avoid-overused-words-and-phrases-in-ai-generated-content/), [Humanaizer's cliché breakdown](https://humanaizer.io/blog/ai-writing-clichs-to-avoid-and-what-to-write-instead), and a landing-page-specific pass from [Microcopy Examples](https://microcopyexamples.substack.com/p/avoid-landing-page-words).

**Single words:** unleash, unlock, elevate, supercharge, turbocharge, amplify, revolutionize/revolutionary, seamless, robust, cutting-edge, future-ready, game-changer, empower, harness, navigate, embark, skyrocket, catapult, bombard, arsenal.

**Phrases:** "in the fast-paced world of," "in today's digital landscape," "in the age of AI," "take it to the next level," "drive impact," "chaos into clarity."

**Sentence-level tics (more diagnostic than any single word):**
- The false-contrast pair: *"It's not about X, it's about Y"* / *"That's not X, that's Y."*
- The double negation: *"Not because X. But because Y."*
- The triple staccato: *"No X. No Y. Just Z."*
- The rhetorical-question pivot: *"The result? Higher engagement."*
- The em dash used as connective tissue between two clauses that don't actually need one, repeated more than once on a page.
- The tricolon that adds no new information across its three beats ("faster, smarter, better" — three words describing the same claim).

**The important nuance the word-lists miss:** none of these patterns are disqualifying in isolation — they're diagnostic of *unearned* abstraction, not of the words themselves. Arc's real, human-written, award-recognized hero subhead is exactly the "not X, it's Y" contrast pattern: "A browser that doesn't just meet your needs — it anticipates them." ([arc.net](https://arc.net)) It works because "anticipates" is a specific, falsifiable claim about the product's actual mechanism (Arc's Spaces/Boosts), not a placeholder for one. The test isn't "does this sentence match a banned pattern," it's "does the second half of the sentence contain information the first half didn't already give you." Monzo's real hero subhead uses "Unlock smarter ways to save" — "unlock" appears on every banned-word list — and it still reads as fine, because it's one clause inside a three-clause list that's otherwise concrete ("Get clear on your spending... Invest with confidence"). One soft verb surrounded by specifics is invisible; three soft verbs in a row is the tell.

**2026's emerging cliché cluster, worth flagging pre-emptively:** "agentic" and "agent" now appear as the load-bearing word in hero copy across categories that have nothing to do with each other — Vercel's live 2026 hero headline is literally "Agentic Infrastructure"; Attio's is "Welcome to agentic revenue." Both are precise today (both companies ship literal AI agents), which is exactly why the word is about to follow "seamless" and "revolutionize" down the same path: real differentiator → category descriptor → meaningless intensifier, in roughly 18-24 months. Anyone briefing copy in late 2026 should treat "agentic" the way a 2015 brief should have treated "disruptive."

**What to write instead:** replace the intensifier with the mechanism. Not "supercharge your workflow" but the actual thing that happens ("cuts review time from three days to one"). See Artefact B for the full replace-with table.

### 2.4 Length, subhead, CTA copy

**Length:** six to twelve words for the headline — long enough to carry one real claim, short enough to read in a single glance. Sentence case (only the first word and proper nouns capitalized) now reads as the conversational default rather than Title Case. ([Woobox](https://woobox.com/articles/landing-page-headline-formulas), [SeedProd](https://www.seedprod.com/landing-page-headline-formulas/))

**Subhead's job:** the headline is the claim; the subhead is the proof of who, how, or when. Stripe's pairing is a clean example: headline "Financial infrastructure to grow your revenue" (claim) → subhead "Accept payments, offer financial services, and implement custom revenue models — from your first transaction to your billionth" (the how, plus a specificity flourish at the end that turns an abstract scaling claim into an image).

**CTA copy:** button copy should reflect the person's result, not the system's action — "Send my invite," not "Submit." Concrete CTAs beat "Learn More" reliably. See the full real-example list in §1.2's CTA subsection.

### 2.5 Show the product in the hero

Cross-referenced across three independent SaaS-structure sources: showing a real screenshot of the actual dashboard or interface "does more conversion work than illustrations or 3D graphics" for self-serve software. The mechanism is trust through specificity — an illustration could belong to any product; a real interface can't. This is why Linear, Notion, Attio, and Cursor all put product UI directly in or immediately below the hero rather than an abstract visual.

The counter-case is deliberate: when the product itself is not visual (infrastructure, APIs, abstract services), "showing the product" becomes "showing the *effect* of the product." Cerebrium (serverless AI infra — nothing to screenshot) built an interactive 3D visualization instead, explicitly reasoning that abstract infrastructure needed to be *felt*, not diagrammed. ([Codrops](https://tympanus.net/codrops/2026/07/23/building-cerebrium-making-serverless-infrastructure-tangible/)) Same principle, different medium, because the underlying product differs.

### 2.6 Hero video vs. 3D tradeoffs

**Video:** highest emotional bandwidth, highest performance risk. "Heavy video files and complex JavaScript libraries can significantly degrade page speed, particularly on mobile" and a slow-loading hero "defeats its own purpose as users will bounce before the visual even renders." ([HostArmada](https://www.hostarmada.com/blog/video-hero-section/), [Mintec](https://mintec.co/blog/video-lcp-hero-performance-2026/)) Non-negotiable technical floor: static poster frame loads first and *is* the LCP element; video plays as enhancement on top of it, never as a blocking dependency.

**3D/WebGL:** highest craft ceiling, highest engineering cost, real device/accessibility exclusion risk — "WebGL isn't always accessible or compatible with all devices, is resource-intensive." The Cerebrium team's own post-mortem is the most honest account of this cost found in the research: they built the entire hero scene in Three.js's newer WebGPURenderer for cleaner code, discovered shader compilation took "close to twenty seconds" on load — unshippable — and had to port the whole scene back to WebGLRenderer, relearning that "two shaders implementing exactly the same idea rarely produce identical images," which meant recalibrating every bloom, fog, and material value by hand. ([Codrops](https://tympanus.net/codrops/2026/07/23/building-cerebrium-making-serverless-infrastructure-tangible/)) Budget for this tax whenever "3D hero" is scoped — it is never a drop-in.

**Static image:** lowest risk, lowest ceiling. "Static hero images generally provide faster load times and improved stability across devices," and for most self-serve SaaS, a real product screenshot beats both video and 3D on conversion per §2.5 — the fancier medium only wins when the product itself is either physically beautiful (Oura, physical goods generally) or is not visual at all (infra, Cerebrium).

---

## 3. Copywriting

### 3.1 Voice and tone systems

The clearest pattern across every live-fetched site: **voice is a constraint system stated once, then applied everywhere**, not a paragraph of adjectives in a brand deck.

- **Basecamp/37signals** — plainspoken, first-person, faintly combative toward the category itself: "most project management systems are bloated, complicated, and confusing," signed off by name: "Thanks for checking us out. We invite you to give Basecamp a try — Jason Fried." ([basecamp.com](https://basecamp.com)) The tone constraint: never sound like software marketing; sound like a founder who's annoyed on your behalf.
- **Raycast** — the tagline system itself *is* the voice system: one-word claim, one-sentence proof, repeated as a design pattern: "Fast. Think in milliseconds." / "Ergonomic. Keyboard First." / "Personal. Your tools, your way." / "Reliable. 99.8% crash-free rate." ([raycast.com](https://raycast.com)) Every claim is immediately taxed with a number or a mechanism.
- **PX PUSH** — a governing bit, not a tone descriptor: "bureaucratic language" and "deadpan," reverse-engineered from the question "What would this look like on a screen from 1988?" ([Codrops](https://tympanus.net/codrops/2026/08/07/the-department-is-open-building-the-px-push-website/)) The tone isn't chosen from a list of adjectives, it's derived from the concept.
- **MERSI** — restraint as the entire voice: "quiet luxury: subtle, precise, tactile and confident without being demonstrative." ([Codrops](https://tympanus.net/codrops/2026/07/27/between-print-and-digital-the-making-of-mersis-website/)) Notably, this is a voice defined by what it withholds.
- **Notion** — literary borrowing as brand voice: closing the homepage on a McLuhan paraphrase, "We shape our tools, and thereafter our tools shape us," functioning simultaneously as tagline, proof-of-taste, and testimonial-shaped closer. ([notion.com](https://www.notion.com))

The operating lesson for a briefing document: a voice system should be expressible as *one governing constraint plus a small number of enforced patterns* (Raycast's claim+proof pairing; Basecamp's founder-signs-every-page), not as a list of adjectives ("friendly, bold, human") that any two competitors could both claim.

### 3.2 Specificity over adjectives

Every strong proof point gathered in this research replaces an adjective with a number and a source:

| Vague version | Specific version actually used | Source |
|---|---|---|
| "Massive traffic growth" | "Traffic increased 10x (6,000 → 60,000 visitors), daily users 8x (100 → 800), leads 2.9x" | Nod Coding Bootcamp / [Codrops](https://tympanus.net/codrops/2024/11/26/case-study-nod-coding-bootcamp/) |
| "Most people see results" | "86% of Oura Members see their health improve" | [ouraring.com](https://ouraring.com) |
| "Extremely reliable" | "99.8% crash-free rate" | [raycast.com](https://raycast.com) |
| "Great for teams" | "16 million personal and business customers have changed the way they bank" | [monzo.com](https://monzo.com) |
| "Widely used" | "Products built with Rive reach over 2 billion users worldwide" | [rive.app](https://rive.app) |
| "Traffic improved" | "Organic impressions grew 112%" | [Webifii](https://webifii.co.in/social-proof-strategy-logo-wall-conversions/), cited as the model good-testimonial sentence |

The pattern holds even at the level of individual clauses: Stripe doesn't say "for businesses of every size," it says "from your first transaction to your billionth" — same claim, but the second version is checkable, imaginable, and therefore believed.

### 3.3 Microcopy for buttons, forms, empty states

Microcopy is "the short, functional phrases inside an interface (buttons, errors, empty states)"; UX writing is the broader discipline including naming and voice/tone. ([Kompassify](https://kompassify.com/blog/ux-microcopy-guide))

**Buttons:** reflect the person's result, not the system's action — "Send my invite" outperforms "Submit"; "Start My Free Trial" outperforms "Submit" on a form. ([Leo9 Studio](https://leo9studio.com/blog/ux-writing-examples-for-better-microcopy-the-ultimate-guide/))

**Empty states:** treat as onboarding, not as dead ends. "A blank screen with a good sentence and one button converts, while a blank screen with 'No data' is a dead end" — the difference is whether the copy proposes the visitor's *next action*, not whether it apologizes for being empty.

**Errors:** explain how to fix the problem, not just that one occurred. PX PUSH's error pages are the extreme, on-brand version of this principle taken as a design opportunity rather than a liability — recreating the Windows Blue Screen of Death for 404/500 pages rather than a generic "Page not found."

**Discipline for a working team:** document every button label, error, and empty state in one shared list to catch voice drift — the same Raycast-style claim+proof discipline applies at microcopy scale, not just headline scale.

### 3.4 Naming sections instead of labeling them

"Overview" and "Features" are labels — they describe the section's *function in the CMS*, not anything a reader would want to know. A useful test: **if a sentence would survive on a competitor's site with the logo swapped, it's too generic to earn attention** — the same test applies to section headers, not just headlines.

What live sites actually name instead of "Features":

- Stripe: "Building the economic infrastructure for AI" / "The backbone of global commerce" (not "Enterprise Features")
- Raycast: "It's not about saving time." / "Take the short way." (not "Benefits")
- Attio: "Agents dig. You close." / "The move's ready. You make the call." (not "How It Works")
- Cursor: "Stay on the frontier" (not "Roadmap")
- Linear: "Intake and integrations" / "Planning and monitoring" / "Build, review, and ship" — named by *workflow stage*, not by feature category (not "Product Features")
- Monzo: "Tell me about..." (an actual conversational fragment, used as a section header, not "FAQ")
- Basecamp: "Tell me if this sounds about right." (a rhetorical question standing in for "Value Proposition")

The mechanism: naming a section as a sentence forces the writer to commit to a specific claim; naming it as a label lets every section be interchangeable filler. This is the copy-level equivalent of the "structure should be felt not seen" design principle from §0 — the taxonomy (features, proof, pricing) should be legible from context and layout, not spelled out in the header.

### 3.5 Twenty-plus real hero headlines, quoted and annotated

All quotes below were live-fetched September 2026 unless marked otherwise. Three fetches failed with HTTP 403 (Cash App, Perplexity, OpenAI) and are omitted rather than reconstructed from memory.

| Company | Hero headline (verbatim) | Why it works |
|---|---|---|
| [Linear](https://linear.app) | "The product development system for teams **and agents**" | Names category (rare, usually a failure mode — see §2.2) but earns it by appending the one modifier that makes it 2026-specific and true only of Linear right now. |
| [Stripe](https://stripe.com) | "Financial infrastructure to grow your revenue." | Verb ("grow") ties an abstract noun ("infrastructure") to a business outcome in six words. |
| [Vercel](https://vercel.com) | "Agentic Infrastructure" | Two words, category-defining for the moment (see §2.3 for the cliché-risk caveat) — bets the whole hero on being early to a term rather than explaining it. |
| [Raycast](https://raycast.com) | "Your shortcut to everything." | Product-literal (Raycast is a launcher) and metaphor at once; earns the double meaning instead of forcing it. |
| [Framer](https://framer.com) | "Framer is the AI design agent for every step from idea to launch" | States the full user journey (idea → launch) inside the headline instead of deferring it to a subhead. |
| [Rive](https://rive.app) | "The Interactive experience engine" | Category name, but a category Rive effectively created — the "outcome vs. category" rule bends when there's no existing category to outperform. |
| [Anthropic](https://www.anthropic.com) | "AI research and products that put safety at the frontier" | Leads with the constraint (safety) before the capability — unusual and deliberate positioning against category norms. |
| [Arc](https://arc.net) | "Arc is the Chrome replacement I've been waiting for." | Headline is a user quote, not brand copy — proof and promise fused into one sentence, first person. |
| [Family](https://family.co) | "Your favorite crypto wallet." | "Favorite" is a claim of affection, not utility — deliberately breaks from the security/utility framing every other crypto product uses. |
| [Basecamp](https://basecamp.com) | "The refreshingly straightforward project management system that's rock-solid and easy to use." | Three adjectives that each argue against a specific category reputation (bloated, fragile, complicated) — the adjectives are doing rebuttal work, not decoration. |
| [Monzo](https://monzo.com) | "Monzo for all your money" | Compresses "we're not just a current account anymore" (their actual 2026 positioning shift) into five words. |
| [Notion](https://www.notion.com) | "Where teams and agents Think together." | Mid-sentence capital "Think" is a typographic flourish that slows the reader down on the one word that matters. |
| [Cursor](https://cursor.com) | "Cursor is your coding agent for building ambitious software." | "Ambitious" does double duty — flatters the reader's project while implying the tool is for serious use, not toys. |
| [Attio](https://attio.com) | "Welcome to agentic revenue." | "Welcome to" frames the category itself as a new place, not just a new tool — high-risk, high-reward framing (see §2.3 cliché-risk note on "agentic"). |
| [Superhuman](https://superhuman.com) | "Superpowers, everywhere you work" | Brand name literalized as the claim (Superhuman → superpowers) without restating the product name redundantly. |
| [Oura](https://ouraring.com) | "Subtle. Power." | Two words, a sentence fragment each — mimics the physical object's own minimalism typographically. |
| [Figma Config](https://config.figma.com) | "Figma's conference for people who build products" | Defines the audience ("people who build products"), not the event format — answers "is this for me" before "what happens there." |
| [Datadog](https://www.datadoghq.com) | "AI-Powered Observability and Security" | Pure category compression for a bottom-of-funnel enterprise buyer who arrived already knowing the category — correctly skips persuasion. |
| [basement.studio](https://basement.studio) | "A digital studio & branding powerhouse making cool shit that performs" | Profanity as a filter — actively de-selects clients who'd be a bad culture fit, which is the point for an agency. |
| [Cerebrium](https://tympanus.net/codrops/2026/07/23/building-cerebrium-making-serverless-infrastructure-tangible/) (positioning line, via case study) | "We wanted people to *feel* how it works." | Not hero copy verbatim but the design brief distilled to one sentence — included because it's the clearest articulation in the whole research set of "show don't tell" as an operating principle rather than a slogan. |
| [Teenage Engineering](https://teenage.engineering) | Product-name-as-poetry: "K.O.–SIDEKICK," "field system™," "pocket operator®" | No hero sentence at all — the product *names* are written with the same design density as slogans elsewhere, so naming absorbs the persuasive work. |

Supporting proof-lines and section headers worth studying alongside the headlines proper (pushing well past 20 total quoted pieces of copy): Raycast's tagline system ("Fast. Think in milliseconds." / "Ergonomic. Keyboard First." / "Personal. Your tools, your way." / "Reliable. 99.8% crash-free rate."); Family's one-word section heads ("Easy," "Secure," "Fast," "Powerful," "Fun"); Attio's "Agents dig. You close."; Cursor's "Stay on the frontier"; Linear's "Built for the future. Available today."; Basecamp's "And there's more…" as an ending device; Obys's footer line, "The studio is shaped by people who care deeply about design and the process behind..."

---

## 4. Storytelling & pacing for scroll

### 4.1 Scroll as timeline / act structure

Scroll position is the only "time" a static page has, so pacing decisions are structural, not decorative. Modern award-winning sites increasingly break from pure vertical scroll as the only timeline device: "recent award-winning examples include sites using comic panels advanced by holding rather than scrolling, looping short-film arcs, game-style unlocks with XP counters, portfolios explored as playable worlds" ([Utsubo](https://www.utsubo.com/blog/immersive-storytelling-websites-guide)) — Studio375's "Ten Years Away" (a scroll-driven interactive comic for a 10th anniversary) is a live example of the comic-panel variant. ([Codrops](https://tympanus.net/codrops/2026/07/08/ten-years-away-designing-an-interactive-comic-for-studio375s-tenth-anniversary/))

### 4.2 Rhythm: dense, quiet, peak

The clearest documented example of deliberate pacing-as-rhythm in this research is Yestalgia (Decathlon), which alternates two section types on purpose: dense product sequences, then "purely graphic sections where typography, shapes and movement take over" as a deliberate pacing break — "closer to flipping through a campaign than navigating a traditional product catalogue." ([Codrops](https://tympanus.net/codrops/2026/09/12/yestalgia-bringing-decathlons-90s-spirit-to-life-through-a-playful-digital-experience/)) Goodgrowth demonstrates the same rhythm at the level of prose: its case-study section headers are deliberately short and punchy ("It starts with a sketch," "Boot it up," "Feeling alive") specifically to "create rhythm and prevent technical density from overwhelming readers." ([Codrops](https://tympanus.net/codrops/2026/08/27/goodgrowth-boot-sequences-spinning-discs-and-the-art-of-the-portfolio/))

The generalizable rule: **never stack two dense/technical sections back to back.** Insert a low-information, high-mood beat (a full-bleed image, a single big statement, a graphic interlude) between any two sections that both require reading.

### 4.3 One idea per screen

Cerebrium's team engineered toward this literally — every scene (network visualization, security shield, environment) maps to exactly one product attribute, never two attributes competing in the same viewport. ([Codrops](https://tympanus.net/codrops/2026/07/23/building-cerebrium-making-serverless-infrastructure-tangible/)) On conversion pages the same rule shows up as "don't put the feature grid and the testimonial carousel in the same fold" — each scroll stop should have one job, matching the "one page, one job" principle from §1.1 applied at section granularity instead of page granularity.

### 4.4 How many sections before fold-to-CTA

There's a real split between page types, not a single number:

- **Narrative/portfolio pages** intentionally withhold the ask through the entire intro → reveal → demonstration → proof arc — the CTA (a single "get in touch") doesn't appear until the *invitation* beat, often 5+ scroll-screens in.
- **Conversion SaaS pages** put a CTA in the hero (screen one) and repeat it 3-5 times total, per the B2B research in §1.1 — "One primary CTA should dominate, with secondary actions supporting different readiness levels."

A page trying to do both (the brief's actual ask) should treat the *first* CTA as non-negotiable in the hero, then allow itself one long uninterrupted narrative stretch (the demonstration beat) before the next ask — that's the compromise structure recommended at the end of §1.4.

### 4.5 Where to put the signature moment

Every teardown in this research has exactly one scene the whole build is organized around, and it's placed either in the hero or immediately after it — never buried mid-page:

- Cerebrium: the interactive security shield.
- PX PUSH: the CRT-monitor framing device, established immediately.
- Goodgrowth: the boot sequence, first thing loaded.
- Trionn: the hold-to-blast hero interaction ("Dare ⚡ to touch the lines").
- MERSI: the split-screen slider, "a strong sense of tension between images," anchoring the whole visual system from the first screen.

The pattern generalizes: **the signature moment establishes the visitor's expectations for everything after it, so it has to come before their attention budget is spent, not as a reward for scrolling to the end.** Saving the best interaction for the bottom of the page assumes visitors that don't exist — most won't get there.

### 4.6 Preloaders: when justified

The clearest verdict found: "The only acceptable use case for a loading animation is a complex web application — a dashboard, booking system, or data-heavy tool — where actual processing time is unavoidable... for standard business websites... there is no justification for a loading screen." ([Lollypop Design](https://lollypop.design/blog/2025/july/preloader-design/)) The purpose, when it is justified, is narrowly to communicate "something is happening" — not to be a branding opportunity that adds its own delay.

PX PUSH's solution is the more sophisticated real-world answer to the same underlying problem (when a scene genuinely needs to finish loading before reveal): a pub/sub lifecycle-coordination system where "the page waits until every registered task has settled, or until a hard timeout is reached" — meaning the *hard timeout* is the actual design decision, not the preloader animation itself. Any preloader briefed without a hard timeout is a liability, not a craft choice. ([Codrops](https://tympanus.net/codrops/2026/08/07/the-department-is-open-building-the-px-push-website/))

### 4.7 Horizontal / pinned sections: when they earn their place

Horizontal scroll is "ideal for visual storytelling... if you're detailing a sequence of events — especially in a timeline format." ([Vev](https://www.vev.design/blog/horizontal-scrolling-website/)) Pinned/sticky elements work "for charts, product mockups, and other focal visuals you want to remain in frame as the surrounding text updates" ([Lovable](https://lovable.dev/guides/scrolling-designs-patterns-when-to-use)) — the pin only earns its place when something *changes* around a fixed anchor; a pinned section with static surrounding content is just a scroll-jacking tax with no payoff.

Two real, contrasting implementations:
- **Jitter's homepage** uses a large horizontal-scroll hero specifically because Antinomy Studio needed smooth cross-device performance without brittle per-breakpoint logic — solved by driving everything off "progress values as CSS variables" rather than screen-width-dependent timelines, a technique choice that exists entirely to make horizontal scroll *cheap enough to justify*. ([Codrops](https://tympanus.net/codrops/2025/09/02/a-behind-the-scenes-look-at-the-new-jitter-website/))
- **MERSI** uses horizontal scroll only *inside* individual case studies, to "mimic visual spreads from architectural publications" — a print metaphor earning a specific interaction, not a default template choice. ([Codrops](https://tympanus.net/codrops/2026/07/27/between-print-and-digital-the-making-of-mersis-website/))

The shared rule: horizontal/pinned scroll should be traceable to one specific content reason (a real sequence, a real print metaphor, a real comparison), stated in one sentence. If that sentence can't be written, the section is scroll-jacking for its own sake and should ship as normal vertical scroll.

### 4.8 How to end a page

Five distinct, real closing patterns found:
1. **Reopen the conversation** — Basecamp's "And there's more…" refuses to let the page feel finished, implying depth beyond the fold rather than closure.
2. **Capture, don't convert** — Rive and Family both end on newsletter signup, a lower-commitment ask than the primary CTA, positioned only after the primary ask has already been made higher up.
3. **Humanize with a detail** — Obys's footer live clock (CET) is a small, functionally useless, purely trust-building detail — it proves a real studio, in a real timezone, is behind the page.
4. **Regulatory close** — Monzo's dense footer (fraud-ranking disclosures, service-quality survey results) is the correct ending for a regulated category where the *absence* of this section would itself be a red flag.
5. **Break the fourth wall** — PX PUSH's Blue-Screen-of-Death error pages function as an alternate ending for visitors who leave the "happy path," turning an inevitable failure state (404/500) into one more brand moment rather than treating it as out of scope.

---

## 5. Proof and trust

### 5.1 Logo walls that don't look fake

The core finding, worth repeating because it should change how these sections get briefed: logos have become "visual wallpaper" through overuse, and B2B buyers now complete "57-70% of their decision-making independently before contacting vendors" — during that research phase they are actively looking for "narrative evidence of transformation," which a logo cannot provide. ([Webifii](https://webifii.co.in/social-proof-strategy-logo-wall-conversions/))

The same source's five-level social-proof hierarchy, low to high impact — useful as a literal checklist when auditing a page's existing proof section:

1. **Brand association** (logo, badge) — weakest.
2. **Sentiment snippet** (generic praise, unattributed or thinly attributed).
3. **Outcome statement** (a number with no context — "improved traffic").
4. **Contextual testimonial** (named person, specific challenge, measurable result).
5. **Deep-dive case study** (full before/strategy/execution/verified-outcome narrative) — strongest.

Most pages max out at level 2 or 3 and call it "social proof"; the sites that actually convert skeptical buyers operate at 4-5. If a logo wall must exist (it's still a fast trust signal for top-of-funnel visitors skimming in under two seconds), pair it with at least one level-4/5 element nearby rather than letting it stand alone.

Authenticity is non-negotiable and fast to police: "fake proof includes invented or inflated numbers, stock-photo testimonials, borrowed 'trusted by' logos for companies that aren't real customers, fake live-activity counters, and vague unverifiable claims... once caught it discredits everything else on the page." ([landingdoctors.com](https://landingdoctors.com/guides/what-is-social-proof))

### 5.2 Testimonials with faces and specifics

The construction of a genuinely high-converting testimonial has three components, per the same five-level model: **before-state** ("describing the client's situation before your engagement does several things simultaneously"), **intervention logic** (explain *why* specific choices were made — this is what signals real expertise rather than a nice compliment), and **verified outcome** (a specific number, not an adjective — "organic impressions grew 112%" beats "traffic improved"). A quote missing any of the three reads as filler regardless of how enthusiastic it sounds.

Oura's approach reframes testimonials as health narratives organized by *outcome category* rather than by customer identity ("Heart Health," "Symptom Radar," "Women's Health") — proof organized around what the reader is looking for, not around who's speaking. ([ouraring.com](https://ouraring.com))

### 5.3 Metrics with sources

Every strong metric found in this research carries three things: a specific number, a comparison baseline, and (implicitly or explicitly) a source that makes it checkable — see the table in §3.2. The absence of a baseline is the most common way a real number still reads as fake: "10x traffic" is meaningless without "6,000 → 60,000 visitors" attached; the raw multiplier alone invites suspicion, the absolute numbers resolve it.

### 5.4 Case studies

The structural difference between a "portfolio item" and a genuine case study, evident across every agency site fetched: a portfolio item shows the *artifact* (screenshots, a link); a case study narrates the *decision* (what was chosen and why) and ends on a *verified result*. basement.studio's homepage explicitly separates these into two different proof layers — "Selected Work" (artifacts, outcome-oriented framing: "leaves a mark," "impossible to ignore") sits above a separate "Client Roster" (pure brand-association logos) — deliberately not conflating the two, because they do different trust work. ([basement.studio](https://basement.studio))

### 5.5 Security/compliance strips

These exist to neutralize a specific, nameable objection, and the best examples name the objection precisely rather than gesturing at "security" generally: Family's wallet-security section reads "Secure - Relentless protection. Restful ease." (paired opposites: protection is active, ease is passive — answers "will this be a burden to use safely," not just "is it safe"); Datadog leads its enterprise trust signal with "Datadog named a Leader in the Gartner® Magic Quadrant™ for Observability Platforms" (third-party analyst validation, the correct register for an enterprise buying committee); Monzo's regulatory disclosures (APP fraud rankings, independent service-quality survey) exist because UK financial marketing rules require exactly this kind of comparative disclosure — a compliance strip that is legally mandatory, not optional trust decoration.

### 5.6 Pricing table patterns

See the full treatment in §1.2. Summary of the mechanisms, restated because they're easy to blur together: **anchoring** sets a reference point so the target tier looks cheap by comparison (works even when nobody buys the anchor tier); the **decoy effect** adds a deliberately inferior option purely to redirect choice toward a specific tier — a different lever aimed at the same outcome. "Three-tier pricing pages convert 31% better than pages with 4+ tiers" — choice overload is measurable, not just a UX aphorism. ([Digital Applied](https://www.digitalapplied.com/blog/subscription-pricing-page-psychology-decision-framework-2026))

### 5.7 Legal/footer essentials

From the maximalist real example (Monzo) and the taxonomy definition ("final page element with links and secondary CTAs for conversion optimization," per [Web Anatomy](https://www.webanatomy.ai/best-landing-pages/sections)), the footer's non-negotiable inventory: company/legal entity name, privacy policy, terms of service, any category-specific regulatory disclosure (financial, medical, data-protection), support/contact path, sitemap for pages not in primary nav. Everything past that inventory is brand opportunity, not obligation — Obys's live clock and PX PUSH's BSOD 404 both prove the "boring, legally required" layer and the "craft flourish" layer can coexist in the same section without either undermining the other.

---

## 6. Teardowns: twelve real, substantive case studies

**1. Jitter homepage redesign** — Antinomy Studio × Jitter. [tympanus.net/codrops/2025/09/02](https://tympanus.net/codrops/2025/09/02/a-behind-the-scenes-look-at-the-new-jitter-website/)
Decided: shift messaging from feature list to four value pillars (ease, creativity, speed, collaboration), reflecting a real audience pivot from solo designers to teams, based on customer interviews across "dozens of creative teams, studios, and design leaders" through 2024. Why: "start with benefits, not features" — users need value before mechanism. Technical decision: drove the homepage's horizontal-scroll hero off CSS-variable progress values instead of screen-width-dependent timelines, specifically to make the effect affordable across devices.

**2. Nod Coding Bootcamp redesign** — Waaark. [tympanus.net/codrops/2024/11/26](https://tympanus.net/codrops/2024/11/26/case-study-nod-coding-bootcamp/)
Decided: a full rebrand triggered by a prospective student doubting the bootcamp's legitimacy because of the old site's quality — credibility was the actual product being sold, not just the curriculum. Chose "Scandinavian minimalism" plus "Bauhaus-inspired graphics" as the resolved direction from discovery keywords (node, exclusive, elegant, Scandinavian, challenging, adventurous). Why: "small, digestible sections enhanced with engaging visual elements" rather than dense blocks, matching a bootcamp-shopper's actual reading behavior. Result: traffic 10x, daily users 8x, leads 2.9x — plus Awwwards SOTD, Developer Award, and FWA honors.

**3. Cerebrium: making serverless infrastructure tangible.** [tympanus.net/codrops/2026/07/23](https://tympanus.net/codrops/2026/07/23/building-cerebrium-making-serverless-infrastructure-tangible/)
Decided: replace explanatory diagrams/copy with interactive 3D scenes the visitor can manipulate, because the product (serverless AI infra) has no natural visual form. Why: "we wanted people to feel how it works," not read about it. Notable failure-and-recovery: built the hero in WebGPU for cleaner code, found ~20-second shader compile times unshippable, rebuilt in WebGL, and had to hand-recalibrate visual parity because "two shaders implementing exactly the same idea rarely produce identical images."

**4. MERSI website** — FLOT NOIR. [tympanus.net/codrops/2026/07/27](https://tympanus.net/codrops/2026/07/27/between-print-and-digital-the-making-of-mersis-website/)
Decided: an editorial (not grid) content architecture for an architecture portfolio, built around portrait-oriented imagery as a compositional strength rather than a constraint to fight. Why: the brand line was "quiet luxury: subtle, precise, tactile and confident without being demonstrative" — the site's restraint had to match the client's actual positioning, not the studio's default style. Kept Webflow's CMS backbone but rejected its template defaults for a hybrid, client-editable-but-custom system.

**5. PX PUSH website** — The Department. [tympanus.net/codrops/2026/08/07](https://tympanus.net/codrops/2026/08/07/the-department-is-open-building-the-px-push-website/)
Decided: one governing metaphor (viewing the whole site through a simulated 1988 CRT monitor) drives every subsequent decision — color (Blue Screen of Death blue, `#03049C`), tone ("bureaucratic," "deadpan"), even the pricing section (styled as a physical floppy disk) and error pages (styled as Windows crash screens). Why: asking "what would this look like on a screen from 1988?" gave the whole team one falsifiable test for every new decision, rather than relitigating taste each time.

**6. Yestalgia** — Decathlon 90s campaign. [tympanus.net/codrops/2026/09/12](https://tympanus.net/codrops/2026/09/12/yestalgia-bringing-decathlons-90s-spirit-to-life-through-a-playful-digital-experience/)
Decided: alternate dense product sequences with "purely graphic sections where typography, shapes and movement take over" as a deliberate pacing device, and turn the navigation itself into a nostalgia object (a cassette-player menu) instead of decorating a standard menu with retro styling. Why: the brief was explicitly a "campaign," meant to browse "closer to flipping through a campaign than navigating a traditional product catalogue" — the pacing decision follows from the format decision (campaign, not catalogue).

**7. Goodgrowth portfolio** — Matt Stone. [tympanus.net/codrops/2026/08/27](https://tympanus.net/codrops/2026/08/27/goodgrowth-boot-sequences-spinning-discs-and-the-art-of-the-portfolio/)
Decided: short, punchy, almost conversational section headers ("It starts with a sketch," "Boot it up," "Feeling alive") specifically to pace a technically dense build log. Why, in the author's own diagnostic language: a real debugging story where he initially blamed the wrong system ("I was optimizing for something I understood instead of what I could measure") — included here because it's a rare, honest account of *wrong* creative-technical reasoning being caught and corrected, not just a highlight reel.

**8. Trionn** — unified interaction/animation architecture. [tympanus.net/codrops/2026/07/15](https://tympanus.net/codrops/2026/07/15/the-architecture-behind-trionn-coordinating-gsap-three-js-lenis-and-web-audio/)
Decided: treat animation, WebGL, and interaction as "one unified experience" driven by a single shared scroll-progress value, rather than as separately coded systems that happen to sit on the same page. Why: the payoff came "less from individual effects and more from carefully synchronizing every layer" — a direct, quotable argument for content-architecture coherence over isolated craft moments. Acknowledged regret: no shared `canvasManager` from day one, meaning multiple WebGL scenes had to be reconciled after the fact.

**9. Linear's 2026 interface refresh.** [linear.app/now/behind-the-latest-design-refresh](https://linear.app/now/behind-the-latest-design-refresh)
Decided: dim the navigation sidebar, soften borders, warm the gray palette — subtractive changes, not new features. Why: "software rarely gets worse all at once... it contorts out of shape one useful feature at a time," and the fix was making "structure... felt not seen." Notable as the one teardown in this set from a company describing its own *product* UI rather than a marketing site — the same restraint principle applied one layer deeper than a landing page.

**10. Linear's "Output Isn't Design."** [linear.app/now/output-isn-t-design](https://linear.app/now/output-isn-t-design)
Not a build teardown but essential context for this whole document: an explicit argument that AI-assisted design/writing tools produce "plausible outputs" without forcing the "understanding" that actually solves the problem, and that products built this way "unravel the moment you actually use them." Directly supports the brief's framing that award-worthy and converting pages both require an underlying specific idea, not just fluent output.

**11. "The Power of Storytelling"** — Noomo Agency, Awwwards Site of the Day. [awwwards.com/sites/the-power-of-storytelling](https://www.awwwards.com/sites/the-power-of-storytelling)
Decided: an agency portfolio where "craft itself becomes the story" — a shader/3D hero (a glass phoenix), case studies with cinematic page transitions (including a Coinbase Warriors project), a contact page with water-reflection mouse interaction, and a themed 404. Jury scored it 8.37/10 for creativity (one juror gave 10/10) and 8.60/10 for animation/transitions, but only 6.40/10 for accessibility — the honest tradeoff of an interaction-maximalist build, worth citing precisely because it's a real cost, not a hypothetical one.

**12. basement.studio homepage** (live structural analysis, not a written teardown post). [basement.studio](https://basement.studio)
Decided: split proof into two distinct layers on the same page — "Selected Work" (four flagship projects, outcome-driven language: "leaves a mark," "impossible to ignore") and a separate "Client Roster" (pure logo recognition) — rather than merging portfolio and logo-wall into one section, because the two do different trust work (§5.1's level-5 vs. level-1 proof, side by side rather than substituting for each other).

---

## 7. Localisation note: building in Russian and Ukrainian

Everything above assumes English typographic and rhetorical defaults. Three consequences for Cyrillic builds, each with real mechanical effects on a design system, not just "translate the copy."

**Quotation marks.** Both Russian and Ukrainian use outward-pointing guillemets, **« »**, as the primary quotation mark, per [Wikipedia's guillemet usage survey](https://en.wikipedia.org/wiki/Guillemet) and a dedicated breakdown: "the typographer's choice for quotation marks in Russian are the French «Guillemets»." For a nested quote inside a guillemet quote, Russian convention switches to German-style low-high curly quotes, „…" (opening on the baseline, closing at the top) — not the English "smart quotes" a CMS will insert by default. ([Pimp My Type](https://pimpmytype.com/russian-typography/)) Practical implication: any component library or CMS that auto-generates quotation marks from straight `"` characters (most do, defaulting to English curly quotes `“ ”`) needs an explicit locale override for RU/UA builds, or every pull-quote and testimonial on the page will be visibly, obviously wrong to a native reader — the kind of detail that undermines trust exactly the way a stock-photo testimonial does (§5.1).

**Dashes.** This is the point with the sharpest crossover risk against §2.3's "em-dash-as-AI-tic" warning, and it needs to be stated carefully so it isn't misapplied: in **English** copywriting, a page dense with em dashes reads as an AI-writing tell because the pattern is usually decorative, standing in for a comma or period with no added precision. In **Russian typography**, the em dash is not decorative — it's grammatically load-bearing. It's the standard way to punctuate dialogue when direct speech opens a new paragraph, it frequently substitutes for the omitted verb "to be" (a real grammatical function English doesn't have), and — critically — **it is set with spaces on both sides**, unlike the tight, unspaced em dash of American typesetting. ([Pimp My Type](https://pimpmytype.com/russian-typography/)) The rule for a bilingual/trilingual design system: don't port the "reduce em dash frequency" instinct from an English style guide into Russian or Ukrainian copy review — a Russian dash usage that would look excessive in English is often just correct Russian grammar. Judge Cyrillic copy by Cyrillic punctuation norms, not by pattern-matching against the English source.

**Cyrillic display type.** The failure mode is specific and mechanical, not aesthetic: most "cool" Latin display faces have Cyrillic support bolted on afterward, and the two scripts have different structural rules that a naive font-pairing exercise will miss. Documented, recurring problems in weak Cyrillic extensions: "Cyrillic and Latin use different approaches to letter spacing; they differ even for characters with identical shapes," incorrect proportions (straight-sided letters rendered too wide relative to round ones), wrong breve shape on characters that need it, and structural mismatches on the letters shared visually with Latin (**к, о, с**) plus descender problems on **ц, щ, д** (letters with no Latin equivalent, so no inherited care in the original design). ([type.today](https://type.today/en/journal/display)) The same source names specific fonts with strong Cyrillic support (Kablammo, Oi — "fully consistent with the Latin set stylistically") against specific fonts to avoid for Cyrillic display use (Dela Gothic One, Poiret One, Rampart One, Kelly Slab, Reggae One, Stick, Train One — "significant Cyrillic deficiencies"). **Practical rule:** never choose a display face for a RU/UA project from its Latin specimen alone — pull the actual Cyrillic glyph set and check it at the target display size before it enters a design system, not after.

**Headline length.** Russian and Ukrainian both run longer than English for the same meaning — a well-established localization-industry rule of thumb (not independently re-verified by a fresh source in this research pass, flagged as such) puts Cyrillic Slavic-language expansion at roughly 15-25% more characters than the English source, driven by grammatical case endings (no equivalent economy to English's fixed word order + minimal inflection) and the absence of contractions as a compression tool. **Practical consequence for the hero-headline discipline in §2.4:** a "six to twelve words" English headline rule doesn't translate to a word-count rule in Cyrillic — translate for the six-to-twelve-word *reading time*, then let the word count and character count land wherever the language actually needs, and re-test the display type at the resulting length rather than shrinking type to force a fit. This is also why headline translation should never be a literal pass — it should be a fresh six-to-twelve-word (by reading time, not count) Russian or Ukrainian sentence written from the same brief, the same discipline recommended for the English headline in the first place.

**Hyphenation.** Browser-native automatic hyphenation (`hyphens: auto`) has materially weaker, less consistent dictionaries for Cyrillic than for English/Latin scripts across engines — a risk multiplied in display type, where a bad break is far more visible than in body copy. Combined with hanging punctuation (positioning punctuation slightly outside the text block edge "so that they do not disrupt the flow of text or break the margin alignment," per [Wikipedia](https://en.wikipedia.org/wiki/Hanging_punctuation)), the safest practical default for RU/UA display headlines is: **disable automatic hyphenation entirely on headline-level type** (`hyphens: none`), control line breaks manually with `<br>` or non-breaking spaces at chosen points instead, and treat hanging punctuation as a manual per-headline check rather than a CSS default, since guillemets and em dashes (both wider and structurally different from English quote marks and hyphens) hang differently than the Latin punctuation most hanging-punctuation CSS recipes were written for.

---

## Artefact A — Storyline template (fill in per brief, section by section)

Use this as the working document for any new landing brief. Answer every question before writing a word of copy; leave a question blank rather than filling it with a placeholder, and treat any section where three or more questions stay blank as not yet briefed.

### 0. Governing idea (fill first, revisit last)
- In one sentence, what is the *one* thing this page has to make someone believe?
- What's the page's pacing model: narrative (Awwwards five-beat, ask deferred) or conversion (ask in hero, repeated), or the hybrid from §1.4?
- What would a competitor's site NOT be able to say if you swapped the logo? (If nothing survives this test, the brief isn't specific enough yet.)

### 1. Hero
- What does the visitor believe about themselves/their problem in the three seconds before they arrive? (Where are they coming from — an ad, a search, a referral — and what do they already know?)
- What's the outcome-not-category headline? (Test: does it name a result, or a product type?)
- What's the one proof-of-how in the subhead that the headline's claim needs to be believable?
- Is the product/service visual? If yes: what's the real screenshot/photo in the hero? If no: what's the *felt effect* that stands in for it (see Cerebrium, §2.5)?
- What's the first CTA, in the visitor's words for their result, not the system's word for the action?
- Video, 3D, static, or none — and can you name the one content reason for whichever you picked (not "it looks cool")?

### 2. Problem (if used — narrative/portfolio pages may skip this entirely, see §1.2)
- What's the cost of the status quo, in the visitor's operational language, not yours?
- Is this problem being stated, or implied through a case study/before-state? Which is right for this audience?
- Does agitating this problem pair immediately with a credible fix, or does it risk reading as manipulation on its own?

### 3. Solution / how it works
- What's the mechanism — the specific step between "problem" and "trust me" — and can it be shown in 3 steps or fewer?
- Is there one physical metaphor that could carry the whole page (see PX PUSH's CRT, Yestalgia's cassette menu)? Would committing to it clarify or constrain the brief?
- What's being explained in outcome language vs. engineering language — audit every sentence in this section for the swap.

### 4. Proof
- Where does this page sit on the five-level proof ladder (§5.1): brand association, sentiment snippet, outcome statement, contextual testimonial, or deep-dive case study? Can it move up one level?
- For every testimonial: is there a before-state, an explanation of *why* a specific choice was made, and a verified, sourced number? If any of the three is missing, is it fixable or should the quote be cut?
- Do the logos (if used) pair with at least one level-4/5 element nearby, or are they standing alone as wallpaper?

### 5. Features (only if the page needs a persuasion layer beyond proof)
- What's this section named as a sentence, not a label — and does the name make a specific claim (§3.4)?
- Which 3-5 capabilities actually change the buying decision? (Anything else belongs in docs, not the landing page.)
- Is every feature mapped to an outcome for the specific reader, or just described?

### 6. Pricing (if self-serve)
- Three tiers or fewer, or is there a specific reason for more?
- What's the anchor tier, and is the tier you want chosen the one that looks best by comparison?
- Is there a "talk to sales" escape valve for the buyer this page's self-serve pricing doesn't fit?

### 7. FAQ / objections
- What are the three to five real objections (price, lock-in, data, timeline, credibility) — not the softball questions that are actually more marketing copy?
- Does this page's purchase model even need a self-serve FAQ, or is trust built through a relationship (agency/enterprise), in which case this section may not belong here at all?

### 8. Final CTA
- Is this the same CTA as the hero's, or a different, lower-commitment one (newsletter, contact) for visitors not ready to convert?
- How many times has the primary CTA appeared by this point — 3-5 for a conversion page, or is this genuinely the first and only ask (narrative page)?

### 9. Footer
- What's legally/categorically required here (privacy, terms, regulatory disclosure) — is anything missing that this category requires (see Monzo, §5.7)?
- Is there room for one small, functionally unnecessary, trust-building detail (Obys's clock) without undermining the section's real job?
- What happens on 404/500 — a dead end, or one more brand moment?

### 10. Localisation pass (if RU/UA/multilingual)
- Has every headline been rewritten from the brief for reading-time parity, not translated word-for-word (§7)?
- Does the CMS default to English smart quotes anywhere copy will render in Cyrillic? Has it been overridden to « » / „ "?
- Has the chosen display typeface's actual Cyrillic glyph set been checked at display size — not just its Latin specimen?
- Is automatic hyphenation disabled on headline-level Cyrillic type?

---

## Artefact B — Banned words and phrases, with replacements

| Banned | Why it's a tell | Replace with |
|---|---|---|
| Unleash / Unlock | Implies hidden potential with no mechanism named | Name the specific thing that becomes possible, and how |
| Elevate | Vague upward gesture, no direction specified | The specific before → after state |
| Seamless | Claims an absence of friction without proving it | The specific step that was removed or automated |
| Revolutionize / revolutionary | Self-assessed superlative, not a claim a reader can check | A specific, falsifiable comparison to the old way |
| Empower | Abstract transfer of agency, no object | What the person can now do that they couldn't before |
| Supercharge / turbocharge / amplify | Intensity with no quantity attached | The actual number (time saved, % increase, etc.) |
| Robust | Meaningless without a stress condition | What it withstands, specifically (load, downtime, edge case) |
| Cutting-edge / future-ready | Claims novelty without evidence of it | The specific capability that's actually new, dated if possible |
| Game-changer | Overclaims impact before the reader has any context to judge it | Let the reader reach that conclusion from a specific result |
| Harness | Implies taming something wild and abstract | Name the "wild" thing precisely, or cut the metaphor |
| Navigate (as in "navigate complexity") | Filler verb for "deal with" | The actual action taken |
| Embark (as in "embark on a journey") | Borrowed travel metaphor, adds no information | Just start the sentence with the actual first step |
| Deep dive | Promises depth instead of providing it | The specific subtopic, named |
| Skyrocket / catapult / bombard | Violent/aerospace metaphors for ordinary growth | The real growth number and timeframe |
| "In today's fast-paced world" / "in today's digital landscape" | Generic scene-setting that applies to any product, any year | Cut it; open on the specific claim instead |
| "In the age of AI" | Same scene-setting problem, 2026-flavored | Name the specific shift that matters to this specific reader |
| "Drive impact" / "unlock value" | Corporate-jargon substitute for a real outcome | The real outcome, in the customer's own metric |
| "Agentic" (as of late 2026) | Real and precise today only for products with literal agents; on trajectory to become the next "seamless" within ~2 years — see §2.3 | Name what the agent actually does, autonomously, that a human previously had to do |
| "It's not about X, it's about Y" / "That's not X, that's Y" | AI-pattern false contrast; fine only when Y contains genuinely new information (see the Arc counter-example, §2.3) | State Y directly; only keep the contrast if X is a real, specific misconception being corrected |
| "Not because X. But because Y." | Same false-contrast pattern, doubled | Same fix — collapse to the real reason, stated once |
| "No X. No Y. Just Z." | Staccato triple with usually-empty first two beats | Check whether X and Y are real objections being pre-empted; if not, cut to Z |
| "The result? [Claim]." | Rhetorical-question pivot, a filler transition | Just state the result as a sentence |
| Tricolon with no new information per beat ("faster, smarter, better") | Three words restating one claim, sounds decisive, says nothing extra | One specific claim, or three beats that are each actually different |
| Em dash used as default connective tissue, repeated 3+ times on one page (English copy only — does not apply to Russian/Ukrainian, see §7) | Diagnostic of unearned abstraction more than of the punctuation itself | A period, or a colon when it's actually a definition |
| "Whether you're X or Y" | Fake-inclusive hedge that usually adds no targeting information | Pick the actual primary audience and speak to them |
| "At the end of the day" | Filler throat-clearing before the real point | Cut it; start with the real point |
| Generic section labels: "Overview," "Features," "About," "Solutions" | Function-of-the-CMS labels, not claims — interchangeable across any competitor's site (see §3.4) | A one-line claim specific to this section's actual content |
| "Get Started" (as the only CTA, everywhere, with no variation) | Not a lie, just maximally uninformative about what happens next | The visitor's actual next action and result ("Start for free," "Talk to sales," "Download for Windows") |

---

*Research method note: 24 web searches and 44 web fetches were used (3 fetches — Cash App, Perplexity, OpenAI — returned HTTP 403 and are omitted rather than reconstructed from memory). All quoted copy is either live-fetched in September 2026 (marked by company/product name linking to the live URL) or explicitly sourced to a named article. The one exception, flagged inline in §7, is the 15-25% Cyrillic headline-expansion figure, which reflects standard localization-industry practice rather than a fresh citation, since the WebSearch budget was exhausted before that specific figure could be independently re-verified against a live source.*
