# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Marketing site for **TKM-AI** (a platform to optimize, deploy, and observe LLM-based agents). It is an **Astro static site** re-skinned to the editorial design system in `DESIGN.md` (warm cream canvas + dark product surfaces + serif display), with one deliberate substitution: TKM-AI's **deep-teal accent (`#2f6e5f`)** wherever the system specifies Anthropic's coral. It builds to static HTML and deploys on **Cloudflare Pages** at [tkm-ai.com](https://tkm-ai.com).

It started life as a single hand-authored `index.html`; that has been replaced by the Astro project described below. The homepage content and the savings-chart behavior were carried over faithfully; the look was redesigned.

## Commands

```bash
npm install            # once
npm run dev            # local dev server (http://localhost:4321)
npm run build          # static build → dist/
npm run preview        # serve the built dist/ locally
npm run check          # astro check — TypeScript + template diagnostics (the closest thing to tests)

# Deploy (Cloudflare Pages, project "tkm-ai")
npx wrangler pages deploy dist --project-name tkm-ai --branch main
```

There is **no unit-test framework** (by design). "Verifying a change" = `npm run check` is clean, `npm run build` succeeds, and the page looks right in `npm run dev` / a Cloudflare preview deploy.

## Architecture / things that aren't obvious from a glance

- **`DESIGN.md` is the design source of truth.** The whole theme is the `:root` token layer at the top of `src/styles/global.css` — surfaces/text/spacing/radius adopted from DESIGN.md as-is, with coral→teal as the single change. Re-skin by editing tokens; almost nothing hardcodes a color outside `:root`. Display headlines are **Cormorant Garamond** (serif, weight 500, negative tracking) — never bold; body is **Inter**; code/mono is **JetBrains Mono**. Keep the **TKM-AI diamond logo/name** — never Anthropic's spike-mark.
- **One global stylesheet.** `src/styles/global.css` holds the tokens + base + every section's CSS (not Astro scoped styles). Components are pure markup referencing those classes.
- **Components** in `src/components/` map 1:1 to page sections (`Nav`, `Hero`, `SavingsChart`, `TrustMarquee`, `StatsBand`, `Features`, `HowItWorks`, `About`, `CTA`, `Footer`). `src/pages/index.astro` composes them inside `src/layouts/BaseLayout.astro` (which owns `<head>`, meta/OG, fonts, favicon). **Band pacing matters** (DESIGN.md): cream → soft → cream-strong → cream → DARK (how) → cream → TEAL (cta) → DARK (footer). Don't put two of the same surface mode back-to-back.
- **Three interactive pieces**, vanilla TS in component `<script>` blocks:
  1. **Scroll-reveal** — `IntersectionObserver` adds `.in` to `.reveal` elements (in `index.astro`). New fade-in sections need the `reveal` class.
  2. **Contact form → `mailto:`** — no backend; `#contactForm` (in `CTA.astro`) builds a `mailto:hello@tkm-ai.com` link. Changing the address means editing this script + the footer/CTA links.
  3. **Savings chart** (`#rchart`, in `SavingsChart.astro`) — data in the `WORK` object (`chat`/`rag`/`agents`). **Numbers are illustrative, not benchmarks** — the page says so (`.rnote`). Bars animate via `transform: scaleX()` (not width — width:100% resolves to 0 in the flex track); entrance fires via `setTimeout`, not `requestAnimationFrame` (rAF is throttled offscreen). Re-skinned onto a dark product surface; the TKM bar uses `--primary` (teal).
- **Logo marquee** (`TrustMarquee.astro`) is **data-driven**: edit the single `logos` array; it's rendered doubled (`[...logos, ...logos]`) so the `-50%` scroll loops seamlessly. No more editing two copies.
- **Blog** lives in `src/content/blog/*.md`, typed by `src/content.config.ts` (zod schema: `title`, `description`, `pubDate`, `draft`, `tags`). `/blog` lists non-draft posts; `/blog/[...slug]` renders one. Markdown only for now (MDX not wired up).
- **Content is deliberately genericized-honest.** No invented people, precise stats, pricing, or "sign in" language implying a shipped product — keep claims aspirational/illustrative.

## Deployment / DNS (operational notes)

- Hosted on **Cloudflare Pages**, project `tkm-ai`, build `npm run build`, output `dist`. Deploy via Wrangler direct upload (above) or connect the GitHub repo in the Pages dashboard for auto-builds on push to `main`.
- **DNS is on Cloudflare.** The custom domain `tkm-ai.com` is attached to the Pages project (Pages → Custom domains), which manages the apex record. The old GitHub Pages `A`/`AAAA` records and the repo's `CNAME`/`.nojekyll` files have been retired.
- DNS-only (grey cloud) is fine; if you enable Cloudflare's proxy (orange cloud), use SSL/TLS mode **"Full"**.
