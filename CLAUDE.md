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

## Deployment / CI / DNS (operational handoff)

**Hosting:** Cloudflare Pages, project `tkm-ai` → `tkm-ai.pages.dev`. Production = the `main`-branch deploy, live at https://tkm-ai.com (TLS auto-managed by Cloudflare / Google Trust Services). **GitHub Pages is disabled — do not re-enable.** It errored on every push once the static `index.html` was removed; it was deleted via `gh api -X DELETE repos/TKM-AI/tkm-ai.com/pages` (a `GET …/pages` should 404).

**Identifiers (not secrets — safe here, still need auth to use):** CF account `6f3c9ca99d20c819d9d4288f95fc4b96` (Lekhang4497@gmail.com); `tkm-ai.com` zone `cbfe7c6b0c59bf8ff9215e4c62c7cf95`.

**Two ways to deploy:**
- **Manual:** `npm run build` then `npx wrangler pages deploy dist --project-name tkm-ai --branch main` (`--branch main` marks it production).
- **Push-to-deploy:** `.github/workflows/deploy.yml` runs `npm ci → npm run build → wrangler pages deploy` on every push to `main`. Requires repo secret **`CLOUDFLARE_API_TOKEN`** (scope: Account → Cloudflare Pages → Edit; account ID is hardcoded in the workflow). The Pages project is **Direct Upload, NOT Git-connected** — a plain `git push` deploys *only because of this workflow*, there is no Cloudflare-side auto-build.

**Auth:** `wrangler` needs `wrangler login` (OAuth, persisted to disk, shared by any shell on this box) **or** `CLOUDFLARE_API_TOKEN`. The user keeps the **Cloudflare MCP** connected; **minting a standing API token via the API/MCP is auto-denied by the harness** — use `wrangler login` for the file upload, and the MCP for everything else.

**Cloudflare MCP — preferred for DNS/Pages/domain ops:** `cloudflare-api__execute` runs JS with `cloudflare.request({method,path,body})` and a pre-set `accountId`; `cloudflare-api__search` searches the API spec (search first, then execute). Used this session for project-create, custom-domain attach, and the DNS cutover.

**DNS (Cloudflare zone — manage via MCP):** apex `tkm-ai.com` + `www` are `CNAME → tkm-ai.pages.dev`, **proxied (orange cloud)** — correct for Pages, do **not** switch to grey/DNS-only. `_domainconnect` CNAME + `_dmarc` TXT are registrar/email records — leave them. The old GitHub-Pages apex `A` (185.199.108–111.153) + `AAAA` (2606:50c0:8000–8003::153) records were deleted during cutover. Custom domains live on the Pages project (`…/pages/projects/tkm-ai/domains`).

**Gotchas:**
- After any DNS/domain change, Pages custom-domain activation takes a few minutes → expect HTTP **522** until routing + cert are ready.
- The local resolver caches stale records (showed `NXDOMAIN` mid-cutover). Verify globally: `dig @1.1.1.1 tkm-ai.com` or `curl --resolve tkm-ai.com:443:172.67.158.126 https://tkm-ai.com`.
- Pushing `.github/workflows/*` needs the `gh` token to have **`workflow`** scope.

**Verify a deploy:** `curl -sI https://tkm-ai.com` (expect 200) · `gh run list --repo TKM-AI/tkm-ai.com` · the Astro rebuild merged from `astro-migration` into `main` (the deploy source); `dist/`, `node_modules/`, `.astro/` are gitignored.
