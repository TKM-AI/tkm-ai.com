# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Marketing site for **TKM-AI** — the company building tools to optimize, deploy, and observe LLM-based agents. It is an **Astro static site** built on the cream-canvas + dark-product-surface editorial system from `DESIGN.md`, with **two TKM-specific changes**: the display face is **Google Sans** (Google Fonts, not DESIGN.md's serif) and the accent is **warm ink black `#141413`** (not Anthropic's coral / the old teal). It builds to static HTML and deploys on **Cloudflare Pages** at [tkm-ai.com](https://tkm-ai.com).

**It is a multi-page site** (redesigned 2026-06-06): a **company homepage** (`/`, jumbotron hero) plus one page per product — **Uniro** (`/uniro`, agent routing for optimal cost-performance — carries the old landing layout + savings chart) and **PaperMentor** (`/papermentor`, academic paper-review agent for universities). It started life as a single hand-authored `index.html`, then a single-product Astro landing page; both have been superseded.

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

- **The `:root` token layer at the top of `src/styles/global.css` is the theme.** Surfaces/text/spacing/radius come from `DESIGN.md`; the two overrides are `--font-display:"Google Sans"` and `--primary:#141413` (warm ink black; press `#000`). Re-skin by editing tokens; almost nothing hardcodes a color outside `:root`. **Accent flips on dark surfaces:** a near-black accent is invisible on the dark product surfaces, so chart bars / slider thumb / the how-step tick use `--on-dark` (cream) instead of `--primary` — preserve that pattern. Display headlines are **Google Sans** (weight 600, negative tracking); body is **Inter**; code/mono is **JetBrains Mono**. Fonts load via the `<link>` in `BaseLayout.astro`. The brand mark is the **TKM monogram** (`src/components/Brand.astro`, inlined so the letters follow `currentColor` — ink in the nav, cream in the footer); it keeps one small **coral `#cc785c` diamond** accent, which is the *only* coral on the site (deliberate — it's part of the supplied logo). `public/favicon.svg` matches. Never use Anthropic's spike-mark. The nav (`Nav.astro`) has a **Product dropdown** (Uniro/PaperMentor, data-driven) — hidden under 900px like the rest of `.nav-links` (no hamburger yet).
- **One global stylesheet.** `src/styles/global.css` holds the tokens + base + every section's CSS (not Astro scoped styles). Components are pure markup referencing those classes.
- **Three pages, shared components.** `src/pages/index.astro` (company home), `uniro.astro`, `papermentor.astro` each compose components inside `BaseLayout.astro` (owns `<head>`, meta/OG, fonts, favicon, **and the scroll-reveal observer — so every page gets it**). Shared section components (`TrustMarquee`, `StatsBand`, `Features`, `HowItWorks`, `About`, `CTA`) are **prop-driven** (sensible defaults; pass content per page). Page-specific: `HomeHero` (kinetic jumbotron — see below), `ProductsShowcase` (the two product cards on the home page), `ProductHero` (generic product hero; product visual passed via `<slot/>`), `SavingsChart` (Uniro's visual), `ReviewMockup` (PaperMentor's visual). `Nav`/`Footer` are shared and link across the routes. **Band pacing matters** (DESIGN.md): e.g. home is cream → soft → cream-strong → cream → DARK (how) → cream → INK (cta) → DARK (footer). The CTA callout is now **ink (`--primary`)**, not teal. Don't put two of the same surface mode back-to-back.
- **Interactive pieces**, vanilla TS in component `<script>` blocks:
  1. **Scroll-reveal** — `IntersectionObserver` adds `.in` to `.reveal` elements; **lives in `BaseLayout.astro` so all pages get it**. New fade-in sections need the `reveal` class. (Don't put `reveal` on above-the-fold hero copy — it flashes invisible before JS runs; `HomeHero` uses a CSS `jumboIn` load-in animation instead.)
  2. **Kinetic hero** (`HomeHero.astro`) — the headline word cycles via the pure-CSS `rotword` keyframes (4 stacked `.rot-word`s, staggered delays, crossfade); a small script sets `--mx/--my` for a cursor-following warm glow. Both respect `prefers-reduced-motion`.
  3. **Contact form → `mailto:`** — no backend; `#contactForm` (in `CTA.astro`) builds a `mailto:hello@tkm-ai.com` link. Subject/body are **per-page props passed via `data-subject`/`data-body`** (Astro `<script>` can't read props directly). Changing the address means editing this script + the footer/CTA links.
  4. **Savings chart** (`#rchart`, `SavingsChart.astro`) — **Uniro's** hero visual now. Data in the `WORK` object (`chat`/`rag`/`agents`). **Numbers are illustrative, not benchmarks** (`.rnote`). Bars animate via `transform: scaleX()` (not width — width:100% resolves to 0 in the flex track); entrance fires via `setTimeout`, not rAF (throttled offscreen). On the dark surface the TKM bar uses `--on-dark` (cream), not the accent.
  5. **Review mockup** (`ReviewMockup.astro`) — **PaperMentor's** hero visual; static dark-surface mockup of a paper review (annotation + citation checks + score). Illustrative (`.rv-note-fine`).
- **Logo marquee** (`TrustMarquee.astro`) is **data-driven**: edit the single `logos` array; it's rendered doubled (`[...logos, ...logos]`) so the `-50%` scroll loops seamlessly. No more editing two copies.
- **Blog** lives in `src/content/blog/*.md`, typed by `src/content.config.ts` (zod schema: `title`, `description`, `pubDate`, `draft`, `tags`). `/blog` lists non-draft posts; `/blog/[...slug]` renders one. Markdown only for now (MDX not wired up).
- **Content is deliberately genericized-honest.** No invented people, precise stats, pricing, or "sign in" language implying a shipped product — keep claims aspirational/illustrative.

## Deployment / CI / DNS (operational handoff)

**Hosting:** Cloudflare Pages, project `tkm-ai` → `tkm-ai.pages.dev`. Production = the `main`-branch deploy, live at https://tkm-ai.com (TLS auto-managed by Cloudflare / Google Trust Services). **GitHub Pages is disabled — do not re-enable.** It errored on every push once the static `index.html` was removed; it was deleted via `gh api -X DELETE repos/TKM-AI/tkm-ai.com/pages` (a `GET …/pages` should 404).

**Identifiers (not secrets — safe here, still need auth to use):** CF account `6f3c9ca99d20c819d9d4288f95fc4b96` (Lekhang4497@gmail.com); `tkm-ai.com` zone `cbfe7c6b0c59bf8ff9215e4c62c7cf95`.

**Two ways to deploy:**
- **Manual:** `npm run build` then `npx wrangler pages deploy dist --project-name tkm-ai --branch main` (`--branch main` marks it production).
- **Push-to-deploy (`.github/workflows/deploy.yml`):** runs `npm ci → npm run build → wrangler pages deploy` on every push to `main`. ⚠️ **As of 2026-06-06 this is NOT working** — the repo secret **`CLOUDFLARE_API_TOKEN`** is unset, so every run fails with *"necessary to set a CLOUDFLARE_API_TOKEN"* (verify: `gh run list`). **Manual `wrangler pages deploy` is the path that actually keeps the site live.** A silver lining: since CI never deploys, a `git push` can't overwrite production with stale code. To make CI work, create a Cloudflare API token (Account → Cloudflare Pages → Edit) and `gh secret set CLOUDFLARE_API_TOKEN --repo TKM-AI/tkm-ai.com`. The Pages project is **Direct Upload, NOT Git-connected** — there is no Cloudflare-side auto-build.

**Auth:** `wrangler` needs `wrangler login` (OAuth, persisted to disk, shared by any shell on this box) **or** `CLOUDFLARE_API_TOKEN`. The user keeps the **Cloudflare MCP** connected; **minting a standing API token via the API/MCP is auto-denied by the harness** — use `wrangler login` for the file upload, and the MCP for everything else.

**Cloudflare MCP — preferred for DNS/Pages/domain ops:** `cloudflare-api__execute` runs JS with `cloudflare.request({method,path,body})` and a pre-set `accountId`; `cloudflare-api__search` searches the API spec (search first, then execute). Used this session for project-create, custom-domain attach, and the DNS cutover.

**DNS (Cloudflare zone — manage via MCP):** apex `tkm-ai.com` + `www` are `CNAME → tkm-ai.pages.dev`, **proxied (orange cloud)** — correct for Pages, do **not** switch to grey/DNS-only. `_domainconnect` CNAME + `_dmarc` TXT are registrar/email records — leave them. The old GitHub-Pages apex `A` (185.199.108–111.153) + `AAAA` (2606:50c0:8000–8003::153) records were deleted during cutover. Custom domains live on the Pages project (`…/pages/projects/tkm-ai/domains`).

**Gotchas:**
- After any DNS/domain change, Pages custom-domain activation takes a few minutes → expect HTTP **522** until routing + cert are ready.
- The local resolver caches stale records (showed `NXDOMAIN` mid-cutover). Verify globally: `dig @1.1.1.1 tkm-ai.com` or `curl --resolve tkm-ai.com:443:172.67.158.126 https://tkm-ai.com`.
- Pushing `.github/workflows/*` needs the `gh` token to have **`workflow`** scope.

**Verify a deploy:** `curl -sI https://tkm-ai.com` (expect 200) · `gh run list --repo TKM-AI/tkm-ai.com` · the Astro rebuild merged from `astro-migration` into `main` (the deploy source); `dist/`, `node_modules/`, `.astro/` are gitignored.
