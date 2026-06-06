# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Marketing landing page for **TKM-AI** (a platform to optimize, deploy, and observe LLM-based agents). It is a **single self-contained static page** — `index.html` holds all markup, CSS (in one `<style>`), and JS (in one `<script>`). There is **no build step, no framework, no package manager, no tests**. Editing `index.html` and pushing is the entire workflow.

## Commands

```bash
# Local preview (any static server works)
python3 -m http.server 8000      # then open http://localhost:8000

# Deploy = push to main; GitHub Pages auto-builds from main:/ (repo TKM-AI/tkm-ai.com)
git push origin main

# Inspect Pages build / TLS cert / domain state
gh api repos/TKM-AI/tkm-ai.com/pages
gh api repos/TKM-AI/tkm-ai.com/pages/builds/latest -q .status
```

There is no lint/test/build to run. "Verifying a change" means opening `index.html` in a browser (or the local server) and looking at it — the README's note about not screenshotting design-tool files does not apply here; this is the real site.

## Architecture / things that aren't obvious from a glance

- **Theme is entirely CSS custom properties** in `:root` at the top of the `<style>`. The palette is intentionally **monochrome / grayscale** ("professional and trust" — not the original orange from the design handoff). To re-skin the whole site, change the tokens (`--brand`, `--ink`, `--muted`, `--line`, `--deep`, etc.) — almost nothing hardcodes a color outside `:root`.
- **Three interactive JS pieces**, all vanilla, all in the single `<script>`:
  1. **Scroll-reveal** — `IntersectionObserver` adds `.in` to every `.reveal` element. New sections that should fade in need the `reveal` class.
  2. **Contact form → `mailto:`** — there is no backend. The CTA form (`#contactForm`) builds a `mailto:hello@tkm-ai.com` link on submit. Changing the contact address means editing both this JS and the footer/CTA links.
  3. **Hero savings chart** (`#rchart`) — a self-contained IIFE. Its data lives in the `WORK` object (`chat`/`rag`/`agents` → baseline vs TKM-AI cost & latency). **These numbers are illustrative, not measured benchmarks** — the page labels them as such (`.rnote`). Bars animate via CSS `transform: scaleX()` (not width — width:100% resolves to 0 inside the flex track), and the first render fires via `setTimeout`, not `requestAnimationFrame` (rAF is throttled offscreen).
- **Logo marquee** (`.marquee-track`) scrolls by translating `-50%`, so the logo set is **duplicated in the HTML**. Adding/removing a logo means editing **both copies** or the loop will jump. Logos live in `assets/logos/` and are displayed grayscale via CSS `filter`.
- **Content is deliberately genericized-honest.** The original design-tool mockup had fabricated team members, founding details, and hard metrics; those were removed on purpose. Do not reintroduce invented facts (named people, precise stats, pricing, "sign in" language implying a shipped product) — keep claims aspirational/illustrative.

## Deployment / DNS (operational notes)

- Hosted on **GitHub Pages**, repo `TKM-AI/tkm-ai.com`, source `main` branch root. `CNAME` (contains `tkm-ai.com`) and `.nojekyll` must stay in the repo root. **Clearing the custom domain via the API deletes the `CNAME` file** — re-add it if you toggle the domain.
- DNS is managed on **Cloudflare**, records set **DNS-only (grey cloud)**: apex `A` → `185.199.108–111.153`, `AAAA` → `2606:50c0:8000–8003::153`, `www` CNAME → `tkm-ai.github.io`. Enabling Cloudflare's proxy (orange cloud) later requires SSL/TLS mode **"Full"** (Flexible causes a redirect loop).
- **If GitHub's TLS cert provisioning gets stuck** (`cert.state` stays `None` for a long time after DNS is correct): clear and re-set the custom domain — `gh api -X PUT .../pages --input -` with `{"cname":""}`, then `-f cname=tkm-ai.com`. This forces re-verification (`authorization_pending` → `approved`). Enforce HTTPS with `gh api -X PUT .../pages -F https_enforced=true` (typed boolean `-F`, not `-f`).
