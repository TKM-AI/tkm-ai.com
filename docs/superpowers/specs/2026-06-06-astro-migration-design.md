# Migrate TKM-AI landing page to Astro + Cloudflare Pages

- **Date:** 2026-06-06
- **Status:** Approved (design) — ready for implementation planning
- **Scope of this spec:** Faithful 1:1 port of the existing single-page site into Astro, plus a typed blog content collection with one sample post to prove the content-growth path. Docs/changelog and other content types are explicitly deferred to follow-on specs.

## 1. Context / current state

The site is a single self-contained static page: `index.html` (598 lines, ~35 KB) holds all markup, one `<style>` block, and one `<script>`. There is no build step, framework, package manager, or tests. It deploys to **GitHub Pages** (repo `TKM-AI/tkm-ai.com`, `main` branch root) with a custom domain `tkm-ai.com` (`CNAME` + `.nojekyll` in the repo root). **DNS is already managed on Cloudflare** (DNS-only / grey cloud).

The page has 9 sections — nav, hero, trust marquee, stats band, features, how-it-works, about, CTA, footer — a monochrome theme driven entirely by CSS custom properties in `:root`, and 3 vanilla-JS interactions:

1. **Scroll-reveal** — `IntersectionObserver` adds `.in` to `.reveal` elements.
2. **Contact form → `mailto:`** — `#contactForm` builds a `mailto:hello@tkm-ai.com` link on submit (no backend).
3. **Savings chart** (`#rchart`) — a ~80-line IIFE: workload tabs (chat/rag/agents), a volume slider, and bars animated via CSS `transform: scaleX()`. Numbers come from a `WORK` object and are **explicitly labeled illustrative**, not measured.

Assets: 8 logo PNGs in `assets/logos/`, displayed grayscale. The logo set is **hand-duplicated** in the HTML so the marquee's `-50%` translate loops seamlessly.

### Why we're migrating

The site is going to grow in a **content-driven** direction — a blog, docs, a changelog, and additional marketing/feature pages. That growth needs reusable layout/components, file-based routing, and a structured content model — none of which a single hand-authored HTML file provides well. (There is no React product app to unify with; component-sharing across apps is **not** a driver.)

## 2. Goals / non-goals

**Goals**

- Rebuild the existing page as an Astro site that renders **pixel-identical** to today.
- Preserve all 3 interactions with identical behavior.
- Establish the structure for content growth: layouts, components, file-based routing, and a typed blog content collection (with one working sample post).
- Deploy on Cloudflare Pages with per-PR preview deploys, keeping the `tkm-ai.com` domain.
- Keep the existing **honest / genericized-content** discipline intact.

**Non-goals (this spec)**

- No visual redesign.
- No docs or changelog collections yet.
- No MDX (interactive components inside posts) yet.
- No serverless contact form — the `mailto:` form stays.
- No sharing of components with a separate product app.
- No splitting CSS into per-component scoped styles.

## 3. Locked decisions

| Decision | Choice |
|---|---|
| Growth shape | Content-driven (blog, docs, more pages) |
| Framework | **Astro** (static output) |
| Hosting | **Cloudflare Pages** (DNS already on Cloudflare) |
| First-migration scope | Faithful 1:1 port + typed `blog` collection with one sample post |
| Chart interactivity | Vanilla (TypeScript) inside an Astro `<script>` — **not** a framework island |

**Three defaults chosen during brainstorming (and accepted):**

- **(a) One global stylesheet, not per-component scoped CSS.** Scoping changes selector behavior and risks visual drift during the port. Componentizing styles is a possible later refactor.
- **(b) Markdown-only for the blog**, not MDX from day one. MDX is a one-line integration add later if posts need interactive components.
- **(c) Retire GitHub Pages.** The domain moves to Cloudflare Pages. `CNAME` and `.nojekyll` become unnecessary; the CLAUDE.md Pages/DNS operational notes get rewritten for Cloudflare.

## 4. Approaches considered

- **Astro — chosen.** Purpose-built for content + marketing. Ships **zero JS by default** (only the chart sends any), preserving the current page's performance profile. `.astro` files are essentially HTML + components, making the port "paste, then componentize." Typed content collections provide a blog/docs model out of the box. Static output deploys cleanly to Cloudflare Pages and retains the domain.
- **Next.js (static export) — rejected.** Ships a React runtime + hydration on every static marketing page that doesn't need it, and requires more configuration to remain static. Justified only if a React product app were coming; it isn't.
- **Eleventy / light static — runner-up, rejected.** Lightest content generator and also zero-JS, but a weaker component/island story and less momentum than Astro for a team that wants reusable components. Astro yields the same zero-JS output with a better component model.
- **Chart interactivity — vanilla over island.** The chart is one self-contained widget; introducing a framework runtime (React/Preact/Solid island) isn't worth it. Keep it vanilla/TypeScript in an Astro `<script>`. Revisit islands only if interactivity multiplies.

## 5. Architecture / project structure

```
/ (repo root)
├─ src/
│  ├─ layouts/
│  │  └─ BaseLayout.astro        # <head> (meta, OG, fonts, favicon), global CSS import, <slot/>
│  ├─ components/
│  │  ├─ Nav.astro
│  │  ├─ Hero.astro
│  │  ├─ SavingsChart.astro      # markup + typed <script> (ported IIFE)
│  │  ├─ TrustMarquee.astro      # logos from a data array, rendered doubled in a loop
│  │  ├─ StatsBand.astro
│  │  ├─ Features.astro          # feature cards from a data array
│  │  ├─ HowItWorks.astro
│  │  ├─ About.astro
│  │  ├─ CTA.astro               # contact form → mailto (typed <script>)
│  │  └─ Footer.astro
│  ├─ pages/
│  │  ├─ index.astro             # composes the homepage from components inside BaseLayout
│  │  └─ blog/
│  │     ├─ index.astro          # blog listing (non-draft posts, sorted by date)
│  │     └─ [...slug].astro      # single post via getStaticPaths()
│  ├─ content/
│  │  ├─ config.ts               # `blog` collection definition + zod schema
│  │  └─ blog/
│  │     └─ hello-world.md       # one sample post
│  └─ styles/
│     └─ global.css              # :root tokens + base + all section CSS, lifted 1:1 from index.html
├─ public/
│  └─ assets/logos/*.png         # moved from assets/logos/ (served as-is)
├─ astro.config.mjs              # site: 'https://tkm-ai.com'; integrations: [sitemap]
├─ package.json
├─ tsconfig.json
├─ CLAUDE.md                     # rewritten for the new build/deploy workflow
└─ README.md                     # rewritten for the new build/deploy workflow
```

> **Astro version note:** Target the current Astro release (5.x at time of writing). Astro 5 uses the Content Layer API (`glob()` loader) and may prefer `src/content.config.ts`. The exact content-collections API must be confirmed against current Astro docs during implementation — see §7.

## 6. Component mapping

Each of the 9 current sections becomes one component; `index.astro` composes them inside `BaseLayout`. Repetitive markup is **data-driven** so growth is easy and the marquee duplication footgun disappears:

- **Logos** → a `logos` array. `TrustMarquee.astro` renders `[...logos, ...logos]` in a single loop, so the seamless `-50%` scroll still works but there is **one** list to edit (removes the CLAUDE.md "edit both copies" hazard).
- **Features (4), How-steps (3), About values (4), Stats (3)** → small arrays mapped within their components.
- Inline SVG icons are preserved verbatim (Astro renders raw HTML).

The nav gains a **"Blog"** link. Existing in-page anchors (`#product`, `#how`, `#about`, `#contact`) are rewritten as root-relative (`/#product`, …) so they resolve correctly from any page, not just the homepage.

## 7. Content model (blog)

- `src/content/config.ts` (or `src/content.config.ts` per Astro 5) defines a `blog` collection with a **zod** schema: `title` (string), `description` (string), `pubDate` (date), `draft` (boolean, default `false`), `tags` (optional string array).
- `src/pages/blog/index.astro` lists non-draft posts sorted by `pubDate` descending.
- `src/pages/blog/[...slug].astro` renders a single post via `getStaticPaths()` and Astro's `<Content />` / `render()` output.
- **Markdown** to start; MDX is deferred (a one-line `@astrojs/mdx` add later).
- `src/content/blog/hello-world.md` is one sample post that proves listing + detail rendering end-to-end.

**Implementation must confirm the exact Content Collections API against current Astro docs** (the Content Layer `glob()` loader and config file location changed in Astro 5).

## 8. Savings chart strategy

- `SavingsChart.astro` holds the exact current markup. The IIFE moves into the component's `<script>` block and is ported to **TypeScript** (Astro compiles TS in `<script>` natively). Astro bundles it as one small module — behavior is byte-for-byte identical: workload tabs, volume slider, `scaleX` bars, and the `setTimeout`-not-`requestAnimationFrame` entrance (rAF is throttled offscreen).
- The `WORK` data object and the CLAUDE.md gotchas (scaleX not width; setTimeout entrance; numbers are illustrative) carry over as typed data and code comments.
- The `.rnote` "Illustrative estimate based on typical workloads — your numbers will vary." disclaimer is preserved **verbatim**.

## 9. Theme / CSS migration

- The entire `<style>` block lifts into `src/styles/global.css`, imported **once** in `BaseLayout`. The `:root` token set is unchanged, so the whole-site re-skin story (change tokens, restyle everything) is preserved exactly.
- **Decision (a):** keep a single global stylesheet, 1:1 with today, rather than per-component Astro scoped styles. Scoped styles change selector behavior and risk visual drift; componentizing CSS is a possible later refactor.
- Fonts: keep the Google Fonts `<link>` (Plus Jakarta Sans + JetBrains Mono) and `preconnect` hints in `BaseLayout`'s `<head>`, identical to today. (`@fontsource` self-hosting is a possible later optimization.)

## 10. Hosting / deploy (Cloudflare Pages)

- Connect the GitHub repo to **Cloudflare Pages**: build command `npm run build`, output directory `dist`, framework preset "Astro".
- `astro.config.mjs` sets `site: 'https://tkm-ai.com'` (used by the sitemap integration and canonical URLs).
- Attach the custom domain `tkm-ai.com` in Cloudflare Pages. Because DNS is already on Cloudflare, this is a straightforward Pages route/record change.
- **Decision (c):** this retires GitHub Pages. `CNAME` and `.nojekyll` are no longer needed (remove, or keep only if GitHub Pages is desired as a fallback). The CLAUDE.md GitHub Pages/DNS operational notes are rewritten for Cloudflare Pages.
- **Gains:** per-PR **preview deploys** and instant rollbacks; a future path to a real serverless contact form via a Cloudflare Pages Function.
- The `mailto:` contact form **stays** in this migration; a Pages Function form is a noted follow-on.

## 11. Preserved (non-negotiable)

- **Pixel-identical design:** tokens, spacing, type scale, dark bands, hover states, responsive breakpoints at 900px and 560px, and `prefers-reduced-motion` handling.
- **All 3 interactions** (reveal, mailto form, savings chart) behave identically.
- **Honest / genericized content** (CLAUDE.md rule): no invented people, metrics, or pricing; no "sign in" language implying a shipped product; claims stay aspirational/illustrative; chart numbers stay labeled illustrative.
- **SEO / meta:** `<title>`, description, canonical, Open Graph, Twitter card, `theme-color`, and the inline SVG favicon — all carried into `BaseLayout`, with per-page canonical + OG (homepage and each blog post).
- **Accessibility:** aria roles on the chart tabs and slider, alt text on logos, reduced-motion fallbacks.

## 12. Verification

There is no unit-test framework today, and this spec does not add one. Verification appropriate to a build-based static site:

- `astro build` must succeed in CI (catches broken content schema, bad imports, missing files).
- `astro check` (TypeScript + template diagnostics) runs in CI.
- **Manual visual parity:** run `astro dev` and compare against the current `index.html` side-by-side at desktop, 900px, and 560px widths. Confirm: chart tabs + slider + bar animation, marquee loop seamlessness, scroll-reveal, and the mailto submit flow.
- The Cloudflare **preview deploy** is the real pre-promote check before pointing the production domain at Pages.
- `CLAUDE.md` and `README.md` are updated to describe the new workflow ("edit `src/`, `npm run dev`, push → Cloudflare builds and preview-deploys").

## 13. Out of scope (future specs)

- Docs and changelog sections/collections.
- MDX + interactive components in posts.
- Serverless contact form (Cloudflare Pages Function) replacing `mailto:`.
- Any visual redesign.
- Componentizing `global.css` into per-component scoped styles.
- Sharing components with a product app (not applicable to the chosen direction).

## 14. Risks / footguns

- **Visual drift during the port** — mitigated by keeping one `global.css` 1:1 and doing side-by-side comparison at all three breakpoints.
- **Domain cutover (GitHub Pages → Cloudflare Pages)** — validate the Pages preview deploy first, then switch the production route. Low risk since DNS is already on Cloudflare.
- **Marquee seam** — the data-driven loop must still emit the **doubled** logo set for the `-50%` translate to loop seamlessly; keep the same 8-logo set rendered twice.
- **New build step** — the team now needs Node + `npm install`; document this clearly in CLAUDE.md/README so the "just edit a file" expectation is reset.
- **Astro 5 content API churn** — confirm the Content Layer/`glob()` loader API and config-file location against current docs during implementation rather than relying on older examples.
