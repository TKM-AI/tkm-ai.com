# Migrate TKM-AI to Astro + Cloudflare Pages, re-skinned to the DESIGN.md editorial system

- **Date:** 2026-06-06
- **Status:** Approved (design) — ready for implementation planning
- **Scope of this spec:** Port the existing single-page site into Astro **and** re-skin it to the `DESIGN.md` editorial design system (cream + dark + serif), with one substitution — TKM-AI's own **deep teal** accent in place of Anthropic's coral. Plus a typed blog content collection with one sample post to prove the content-growth path. Docs/changelog and other content types are deferred to follow-on specs.
- **Design source of truth:** `DESIGN.md` (repo root). This spec records how we adopt and adapt it; token values and component specs live there.

## 1. Context / current state

The site is a single self-contained static page: `index.html` (598 lines, ~35 KB) holds all markup, one `<style>` block, and one `<script>`. There is no build step, framework, package manager, or tests. It deploys to **GitHub Pages** (repo `TKM-AI/tkm-ai.com`, `main` branch root) with a custom domain `tkm-ai.com` (`CNAME` + `.nojekyll` in repo root). **DNS is already on Cloudflare** (DNS-only / grey cloud).

The page has 9 sections — nav, hero, trust marquee, stats band, features, how-it-works (dark), about, CTA, footer — a currently **monochrome / grayscale** theme driven by CSS custom properties in `:root`, and 3 vanilla-JS interactions:

1. **Scroll-reveal** — `IntersectionObserver` adds `.in` to `.reveal` elements.
2. **Contact form → `mailto:`** — `#contactForm` builds a `mailto:hello@tkm-ai.com` link (no backend).
3. **Savings chart** (`#rchart`) — a ~80-line IIFE: workload tabs (chat/rag/agents), a volume slider, bars animated via CSS `transform: scaleX()`. Numbers come from a `WORK` object and are **explicitly labeled illustrative**.

Assets: 8 logo PNGs in `assets/logos/`. The logo set is **hand-duplicated** in the HTML so the marquee's `-50%` translate loops seamlessly.

### Why we're migrating, and why now re-skinning

Growth is **content-driven** — a blog, docs, a changelog, more marketing pages. That needs reusable layout/components, file-based routing, and a structured content model. Separately, the team wants the site to adopt the **`DESIGN.md` editorial system** (a warmer, more distinctive, magazine-like brand surface) rather than the current monochrome look. The two changes ship together: rebuild in Astro **and** re-skin to DESIGN.md.

## 2. Goals / non-goals

**Goals**

- Rebuild the existing page as an Astro site with reusable layouts/components and file-based routing.
- **Re-skin to the `DESIGN.md` editorial system** — cream canvas, dark product surfaces, serif display + humanist sans, the DESIGN.md spacing/radius/elevation tokens and component patterns.
- Apply **one deliberate substitution**: TKM-AI's deep-teal accent (`#2F6E5F`) wherever DESIGN.md uses coral, and TKM-AI's own diamond logo/name (never Anthropic's spike-mark).
- Preserve all 3 interactions' **behavior** (their visual styling is re-skinned).
- Stand up a typed blog content collection (with one working sample post).
- Deploy on Cloudflare Pages with per-PR preview deploys, keeping `tkm-ai.com`.
- Keep the existing **honest / genericized-content** discipline.

**Non-goals (this spec)**

- No new content sections invented from DESIGN.md that TKM-AI doesn't have (no pricing tiers, connector tiles, model-comparison cards).
- No docs or changelog collections yet.
- No MDX (interactive components in posts) yet.
- No serverless contact form — the `mailto:` form stays.
- No sharing of components with a separate product app.
- **Pixel-parity with the *current* monochrome page is explicitly NOT a goal** — the visual target is DESIGN.md, not today's look.

## 3. Locked decisions

| Decision | Choice |
|---|---|
| Growth shape | Content-driven (blog, docs, more pages) |
| Framework | **Astro** (static output) |
| Hosting | **Cloudflare Pages** (DNS already on Cloudflare) |
| First-migration scope | Structure port + DESIGN.md re-skin + typed `blog` collection w/ one sample post |
| Visual target | **DESIGN.md editorial system** (supersedes the monochrome look) |
| Brand adaptation | Full DESIGN.md system, but **accent = deep teal `#2F6E5F`** (not coral); keep **TKM-AI diamond logo + name** |
| Fonts | **Cormorant Garamond** (display) + **Inter** (body) + **JetBrains Mono** (mono) |
| Chart interactivity | Vanilla (TypeScript) inside an Astro `<script>` — **not** a framework island |

**Accepted defaults from brainstorming:**

- **(a) One global stylesheet** (token layer + component CSS), not per-component scoped CSS, for the first pass. Componentizing styles is a later refactor.
- **(b) Markdown-only blog**, MDX deferred.
- **(c) Retire GitHub Pages.** Domain moves to Cloudflare Pages; `CNAME` and `.nojekyll` become unnecessary; the CLAUDE.md Pages/DNS notes get rewritten.

## 4. Approaches considered (framework)

- **Astro — chosen.** Purpose-built for content + marketing; ships zero JS by default (only the chart sends any); `.astro` ≈ HTML + components, so the port is "paste, then componentize"; typed content collections give a blog/docs model for free; static output deploys to Cloudflare Pages and keeps the domain. DESIGN.md is itself component-oriented, which **reinforces** an Astro component architecture.
- **Next.js (static export) — rejected.** Ships a React runtime + hydration on every static page that doesn't need it; more config to stay static; only justified by a React product app, which isn't in play.
- **Eleventy / light static — runner-up, rejected.** Lightest, zero-JS, but a weaker component/island story than Astro.
- **Chart — vanilla over island.** One self-contained widget; a framework runtime isn't worth it. Revisit only if interactivity multiplies.

## 5. Design system adoption (DESIGN.md)

We adopt DESIGN.md's tokens **as-is** for surfaces, text, spacing, radius, and typography scale, with the single coral→teal accent substitution and the open-source font substitutes DESIGN.md itself names. Concretely, the implementation builds this `:root` token layer (one global stylesheet):

```css
:root {
  /* Surfaces — DESIGN.md, adopted as-is */
  --canvas:#faf9f5; --surface-soft:#f5f0e8; --surface-card:#efe9de; --surface-cream-strong:#e8e0d2;
  --surface-dark:#181715; --surface-dark-elevated:#252320; --surface-dark-soft:#1f1e1b;
  --hairline:#e6dfd8; --hairline-soft:#ebe6df;

  /* Text — DESIGN.md, adopted as-is */
  --ink:#141413; --body-strong:#252523; --body:#3d3d3a; --muted:#6c6a64; --muted-soft:#8e8b82;
  --on-primary:#ffffff; --on-dark:#faf9f5; --on-dark-soft:#a09d96;

  /* Accent — the ONE substitution: coral → TKM-AI deep teal */
  --primary:#2f6e5f; --primary-active:#245446; --primary-disabled:#d7e0db;
  --accent-amber:#e8a55a;   /* small warm companion kept from DESIGN.md (badges, inline highlights) */

  /* Semantic — DESIGN.md */
  --success:#5db872; --warning:#d4a017; --error:#c64545;

  /* Type — DESIGN.md substitutes (Copernicus/StyreneB are licensed, not web-available) */
  --font-display:"Cormorant Garamond","EB Garamond",Garamond,"Times New Roman",serif;
  --font-body:"Inter",-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,sans-serif;
  --font-mono:"JetBrains Mono",ui-monospace,monospace;

  /* Radius — DESIGN.md */
  --r-sm:6px; --r-md:8px; --r-lg:12px; --r-xl:16px; --r-pill:9999px;

  /* Spacing — DESIGN.md (4px base) */
  --s-xxs:4px; --s-xs:8px; --s-sm:12px; --s-md:16px; --s-lg:24px; --s-xl:32px; --s-xxl:48px; --s-section:96px;
}
```

Notes:
- DESIGN.md's secondary `accent-teal (#5db8a6)` is **dropped** — teal is now the primary; keeping a second teal would be redundant. `accent-amber` is retained as the small warm companion (teal + amber on cream is a coherent editorial pairing).
- `--primary-disabled` is a desaturated teal-cream tint (DESIGN.md's coral-disabled was cream-tinted; we mirror that for teal).
- Fonts load from Google Fonts: **Cormorant Garamond** (display, weight 500, letter-spacing ≈ -0.02em per DESIGN.md's substitute note), **Inter** (body, 400/500), **JetBrains Mono** (already loaded). EB Garamond is the serif fallback.

### Typography mapping (current heading → DESIGN.md token → face)

| TKM element | DESIGN.md token | Face / weight |
|---|---|---|
| Hero `h1` | `display-xl` (64 / 1.05 / -1.5px) | Cormorant serif, 500 |
| Section `h2` | `display-lg` (48) or `display-md` (36) | Cormorant serif, 500 |
| Pillar words (Cost/Speed/Scale) | `display-md` (36) | Cormorant serif, 500 — **not** the current 800 sans |
| Feature/value/step titles | `title-md` (18 / 500) | Inter |
| Eyebrows, category tags | `caption-uppercase` (12 / 500 / 1.5px) | Inter |
| Body copy | `body-md` (16 / 1.55) | Inter 400 |
| Footer/fine print | `body-sm` (14) | Inter |
| Buttons | `button` (14 / 500) | Inter |
| Nav links | `nav-link` (14 / 500) | Inter |
| Chart numbers, feature tags | `code` (14) | JetBrains Mono |

**Display headlines stay weight ~500, never bold**, with negative tracking — DESIGN.md's defining typographic rule.

### Elevation & interaction states (DESIGN.md discipline)

DESIGN.md is "color-block first, shadow rare," and documents **default + active/pressed only — never hover elaboration**. So we align away from the current playful hovers:

- **Drop** the current `transform: translateY` lifts and hover box-shadows on buttons/feature cards.
- **Primary buttons** darken to `--primary-active` on press (no transform).
- **Inputs**: focus state per `text-input-focused` — border shifts to `--primary` (teal) with a 3px teal-at-15%-alpha ring.
- **Cards** are flat: cream fill (`--surface-card`) or hairline border, no shadow.
- **Keep** the marquee pause-on-hover (it's behavior, not decorative styling) and all `prefers-reduced-motion` handling.

## 6. Project structure

```
/ (repo root)
├─ src/
│  ├─ layouts/
│  │  └─ BaseLayout.astro        # <head> (meta, OG, fonts, favicon), global CSS import, <slot/>
│  ├─ components/
│  │  ├─ Nav.astro               # → DESIGN.md top-nav (cream, diamond mark + wordmark, teal CTA)
│  │  ├─ Hero.astro              # → hero-band (6-6 grid); right artifact = SavingsChart on dark card
│  │  ├─ SavingsChart.astro      # → product-mockup-card-dark; ported IIFE in a typed <script>
│  │  ├─ TrustMarquee.astro      # logos from a data array, rendered doubled in a loop
│  │  ├─ StatsBand.astro         # cream-strong band, serif pillar words
│  │  ├─ Features.astro          # → feature-card grid (cream cards)
│  │  ├─ HowItWorks.astro        # → dark band (surface-dark)
│  │  ├─ About.astro             # cream section + about-card + value cards
│  │  ├─ CTA.astro               # → callout-card (full-bleed TEAL); contact form → mailto
│  │  └─ Footer.astro            # → footer (dark, surface-dark)
│  ├─ pages/
│  │  ├─ index.astro             # composes the homepage inside BaseLayout
│  │  └─ blog/
│  │     ├─ index.astro          # blog listing (non-draft, by date)
│  │     └─ [...slug].astro      # single post via getStaticPaths()
│  ├─ content/
│  │  ├─ config.ts               # `blog` collection + zod schema
│  │  └─ blog/hello-world.md     # one sample post
│  └─ styles/
│     └─ global.css              # :root token layer (§5) + base + component CSS
├─ public/assets/logos/*.png     # moved from assets/logos/
├─ astro.config.mjs              # site: 'https://tkm-ai.com'; integrations: [sitemap]
├─ package.json · tsconfig.json
├─ DESIGN.md                     # design source of truth (already in repo)
├─ CLAUDE.md · README.md         # rewritten for the new build/deploy workflow
```

> **Astro version note:** target current Astro (5.x). Astro 5 uses the Content Layer API (`glob()` loader) and may prefer `src/content.config.ts`. Confirm the exact content-collections API against current Astro docs during implementation (see §8).

## 7. Component mapping & band pacing

Each of the 9 current sections becomes a component mapped to a DESIGN.md pattern; `index.astro` composes them in `BaseLayout`. Repetitive markup is **data-driven**: logos, the 4 features, 3 how-steps, 4 about-values, and 3 stats each become small arrays. The marquee renders `[...logos, ...logos]` in one loop — so the seamless `-50%` scroll still works but there's **one** list to edit (removes the CLAUDE.md "edit both copies" hazard).

**Section → DESIGN.md component:**

- **Nav** → `top-nav`: cream, 64px; TKM-AI diamond mark + "TKM-AI" wordmark left; nav links (Product / How it works / About / **Blog** / Contact); "Get in touch" `button-primary` (teal). In-page anchors become root-relative (`/#product`, …) so they work from any page.
- **Hero** → `hero-band` (6/6): h1 (`display-xl` serif) + sub + button row left; the **savings chart as the right-side artifact**, presented as a `product-mockup-card-dark`.
- **Savings chart** → `product-mockup-card-dark` styling (see §9).
- **Trust marquee** → soft cream band (`surface-soft`); grayscale logos, muted labels.
- **Stats / pillars** → `surface-cream-strong` band; pillar words as serif `display-md` (not 800 sans).
- **Features (4)** → `feature-card` grid (`surface-card` cream, `--r-lg`, padding `--s-xl`), 2-up desktop → 1-up mobile (4 cards read better 2-up than 3-up).
- **How it works (3)** → **dark band** (`surface-dark`), on-dark text — the cream→dark pacing beat.
- **About** → cream section; `about-card` (cream + hairline) + value cards (`feature-card` style).
- **CTA** → `callout-card` flipped to **full-bleed teal** (`--primary`), white text, padding `--s-xxl`/64px; the email input + an inverted **cream button on teal**; the "voltage" moment.
- **Footer** → `footer`: **dark** (`surface-dark`), `on-dark-soft` text, link columns, diamond mark + wordmark in `on-dark`. (Current footer is light; DESIGN.md's footer is always dark.)

**Band pacing** (DESIGN.md: don't repeat a surface mode in consecutive bands; step the cream family, hit contrast at dark/teal):

```
nav(cream) → hero(cream + dark chart card) → trust(surface-soft) → stats(cream-strong)
→ features(cream) → how(DARK) → about(cream) → CTA(TEAL) → footer(DARK)
```

Inline SVG icons carry over verbatim (Astro renders raw HTML).

## 8. Content model (blog)

- `src/content/config.ts` defines a `blog` collection with a **zod** schema: `title` (string), `description` (string), `pubDate` (date), `draft` (boolean, default `false`), `tags` (optional string array).
- `blog/index.astro` lists non-draft posts by `pubDate` desc; `blog/[...slug].astro` renders a post via `getStaticPaths()`.
- **Markdown** to start; MDX deferred. `hello-world.md` proves listing + detail end-to-end.
- Blog pages reuse `BaseLayout` + the teal/cream/serif system; post body uses the DESIGN.md editorial type scale.
- **Confirm the Content Collections API against current Astro docs** (Content Layer / `glob()` changed in Astro 5).

## 9. Savings chart (re-skin)

- Markup moves into `SavingsChart.astro`; the IIFE moves into its `<script>`, ported to **TypeScript**. Behavior is byte-for-byte identical: workload tabs, volume slider, `scaleX` bars, the `setTimeout`-not-`requestAnimationFrame` entrance (rAF is throttled offscreen). The CLAUDE.md gotchas become code comments.
- **Re-skinned as a dark product surface** (`product-mockup-card-dark`): `--surface-dark` card (`--r-lg`, padding `--s-lg`); tracks on `--surface-dark-soft`/elevated; **baseline bar** = muted on-dark fill, **TKM-AI bar** = `--primary` (teal); labels in `--on-dark` / `--on-dark-soft`; numbers in JetBrains Mono; active tab = `--surface-dark-elevated`.
- The `.rnote` "Illustrative estimate based on typical workloads — your numbers will vary." disclaimer is preserved **verbatim**, in `--on-dark-soft`.

## 10. Theme / CSS migration

- The `:root` token layer (§5) plus base + component CSS live in one `src/styles/global.css`, imported once in `BaseLayout`. Re-skinning later (or re-theming) means editing the token layer — the whole-site re-skin story is preserved, now anchored to DESIGN.md tokens.
- **Decision (a):** one global stylesheet for the first pass, not Astro scoped styles (scoping changes selector behavior and risks drift). Componentizing CSS is a later refactor.
- Fonts: a single Google Fonts `<link>` for Cormorant Garamond + Inter + JetBrains Mono, with `preconnect` hints, in `BaseLayout` `<head>`.

## 11. Hosting / deploy (Cloudflare Pages)

- Connect the repo to **Cloudflare Pages**: build `npm run build`, output `dist`, preset "Astro". `astro.config.mjs` sets `site: 'https://tkm-ai.com'`.
- Attach `tkm-ai.com` in Pages (straightforward — DNS already on Cloudflare).
- **Decision (c):** retire GitHub Pages; `CNAME` + `.nojekyll` no longer needed; rewrite the CLAUDE.md Pages/DNS notes for Cloudflare.
- Gains: per-PR **preview deploys** + instant rollbacks; a future path to a Cloudflare Pages Function for a real contact form. The `mailto:` form **stays** this migration.

## 12. Preserved vs. changed

**Preserved (non-negotiable):**
- **Behavior** of all 3 interactions (reveal, mailto form, savings chart).
- **Honest / genericized content** (CLAUDE.md): no invented people, metrics, or pricing; no "sign in" language implying a shipped product; claims stay aspirational/illustrative; chart numbers stay labeled illustrative.
- **SEO / meta:** title, description, canonical, Open Graph, Twitter card, `theme-color` (update to the cream canvas), inline SVG favicon — carried into `BaseLayout`, per-page canonical + OG (homepage + each post).
- **Accessibility:** aria on chart tabs/slider, logo alt text, reduced-motion, AA contrast (verify teal `#2F6E5F` on cream and white-on-teal for the CTA — see §14).
- **TKM-AI identity:** the diamond logo mark and "TKM-AI" name. Never Anthropic's spike-mark.

**Changed (intentionally):**
- Visual design → DESIGN.md editorial system (cream/dark/serif), teal accent.
- Fonts → Cormorant Garamond + Inter (+ JetBrains Mono).
- Footer → dark.
- Interaction states → DESIGN.md discipline (drop hover lifts/shadows; primary-active darken; teal focus ring).
- `theme-color` meta → cream (`#faf9f5`).

## 13. Verification

No unit-test framework (matches today). Build-based checks:
- `astro build` must succeed in CI (catches broken content schema, bad imports).
- `astro check` (TS + template diagnostics) in CI.
- **Visual review against DESIGN.md** (not the old monochrome page): run `astro dev` and check token fidelity (cream canvas, dark bands, teal accent only where coral would be), the serif/sans split with negative display tracking, band pacing, and the chart's dark-surface re-skin, at desktop / 768 / mobile breakpoints.
- Confirm chart tabs + slider + bar animation, marquee loop, scroll-reveal, and the mailto submit flow still work.
- The Cloudflare **preview deploy** is the real pre-promote check.
- Update `CLAUDE.md` + `README.md` for the new workflow ("edit `src/`, `npm run dev`, push → Cloudflare builds + preview-deploys"); note DESIGN.md is the design source of truth and the coral→teal substitution.

## 14. Out of scope (future specs)

- Docs + changelog collections; MDX; serverless contact form; sharing components with a product app; componentizing `global.css`; any DESIGN.md sections TKM-AI doesn't have (pricing, connectors, model-comparison).

## 15. Risks / footguns

- **Anthropic-adjacency** — mitigated by the teal accent substitution + TKM-AI's own logo/name. Implementation must not reintroduce coral or the spike-mark from DESIGN.md examples.
- **Contrast/accessibility of teal** — verify `#2F6E5F` text on cream and `--on-primary` white on the teal CTA meet WCAG AA; darken toward `--primary-active` for text uses if needed.
- **Serif legibility** — Cormorant Garamond is more delicate/high-contrast than Copernicus; keep it to display sizes only (never body), and verify rendering weight at `display-*` sizes.
- **Redesign drift** — the target is DESIGN.md, so "drift" is measured against it; keep one token layer and review token-by-token.
- **Marquee seam** — the data-driven loop must emit the **doubled** logo set for `-50%` to loop seamlessly.
- **New build step** — team now needs Node + `npm install`; document clearly so the "just edit a file" expectation resets.
- **Astro 5 content API churn** — confirm the Content Layer/`glob()` API + config location against current docs during implementation.
