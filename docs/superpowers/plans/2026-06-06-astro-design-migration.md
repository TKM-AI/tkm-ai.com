# TKM-AI → Astro + DESIGN.md Re-skin + Cloudflare Pages — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild the single-file TKM-AI landing page as an Astro site re-skinned to the DESIGN.md editorial system (cream + dark + serif, deep-teal accent), add a typed blog collection, and deploy to Cloudflare Pages at tkm-ai.com.

**Architecture:** Static Astro site at repo root. One global stylesheet carries the DESIGN.md `:root` token layer (coral→teal substitution) + base + component CSS. Each of the 9 page sections becomes an `.astro` component composed by `src/pages/index.astro` inside `BaseLayout.astro`. The savings chart's vanilla JS is ported to a TypeScript `<script>` and re-skinned onto a dark product surface. A `blog` content collection (Markdown + zod schema) powers `/blog`. Build → Cloudflare Pages via Wrangler direct upload; custom domain + DNS cutover via Cloudflare API.

**Tech Stack:** Astro 5, `@astrojs/sitemap`, TypeScript, Google Fonts (Cormorant Garamond + Inter + JetBrains Mono), Wrangler / Cloudflare Pages.

**Verification model:** This is a static marketing site; the spec deliberately has **no unit-test framework**. Per-task verification = `npm run build` succeeds + `npx astro check` clean + targeted visual/behavioral check on `npm run dev`. "Tests" below mean these build/check/visual gates, not unit tests.

**Source materials the implementer has in-repo:** `DESIGN.md` (design source of truth), `docs/superpowers/specs/2026-06-06-astro-migration-design.md` (the spec), and the original `index.html` (content + markup + the chart IIFE to port). Port markup/content from `index.html`; map styling to DESIGN.md tokens per spec §5/§7.

---

## Task 1: Scaffold Astro project at repo root

**Files:**
- Create: `package.json`, `astro.config.mjs`, `tsconfig.json`, `.gitignore` (modify), `src/env.d.ts`
- Move: `assets/logos/*` → `public/assets/logos/*`

- [ ] **Step 1: Create `package.json`**

```json
{
  "name": "tkm-ai-site",
  "type": "module",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro preview",
    "check": "astro check"
  }
}
```

- [ ] **Step 2: Install dependencies**

Run: `npm install astro@^5 @astrojs/sitemap@^3 @astrojs/check@^0 typescript@^5`
Expected: `node_modules/` created, `astro` binary present, no peer-dep errors that fail install.

- [ ] **Step 3: Create `astro.config.mjs`**

```js
// @ts-check
import { defineConfig } from 'astro/config';
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://tkm-ai.com',
  integrations: [sitemap()],
});
```

- [ ] **Step 4: Create `tsconfig.json`**

```json
{
  "extends": "astro/tsconfigs/strict",
  "include": [".astro/types.d.ts", "**/*"],
  "exclude": ["dist"]
}
```

- [ ] **Step 5: Update `.gitignore`** (append)

```
node_modules/
dist/
.astro/
.env
.dev.vars
```

- [ ] **Step 6: Move logo assets**

Run: `mkdir -p public/assets/logos && git mv assets/logos/*.png public/assets/logos/ && rmdir assets/logos assets 2>/dev/null; ls public/assets/logos`
Expected: 8 PNGs (antigravity, claude, codex, copilot, cursor, deepseek, gemini-cli, openclaw).

- [ ] **Step 7: Verify skeleton builds** (after a placeholder page exists this is re-run; for now just confirm Astro resolves)

Run: `npx astro --version`
Expected: prints Astro 5.x.

- [ ] **Step 8: Commit**

```bash
git add package.json package-lock.json astro.config.mjs tsconfig.json .gitignore public/assets/logos
git rm -r --cached assets 2>/dev/null; git add -A
git commit -m "chore: scaffold Astro project, move logo assets to public/"
```

---

## Task 2: Global stylesheet — DESIGN.md token layer + base

**Files:**
- Create: `src/styles/global.css`

- [ ] **Step 1: Write the `:root` token layer + base.** Use the spec §5 block verbatim, then base resets and shared utilities ported from `index.html`'s `<style>` (the `.wrap`, `.eyebrow`, `.btn*`, `.reveal` rules) re-pointed to tokens.

```css
:root {
  /* Surfaces — DESIGN.md */
  --canvas:#faf9f5; --surface-soft:#f5f0e8; --surface-card:#efe9de; --surface-cream-strong:#e8e0d2;
  --surface-dark:#181715; --surface-dark-elevated:#252320; --surface-dark-soft:#1f1e1b;
  --hairline:#e6dfd8; --hairline-soft:#ebe6df;
  /* Text — DESIGN.md */
  --ink:#141413; --body-strong:#252523; --body:#3d3d3a; --muted:#6c6a64; --muted-soft:#8e8b82;
  --on-primary:#ffffff; --on-dark:#faf9f5; --on-dark-soft:#a09d96;
  /* Accent — coral → TKM-AI deep teal (the one substitution) */
  --primary:#2f6e5f; --primary-active:#245446; --primary-disabled:#d7e0db;
  --accent-amber:#e8a55a;
  /* Semantic */
  --success:#5db872; --warning:#d4a017; --error:#c64545;
  /* Type */
  --font-display:"Cormorant Garamond","EB Garamond",Garamond,"Times New Roman",serif;
  --font-body:"Inter",-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,sans-serif;
  --font-mono:"JetBrains Mono",ui-monospace,monospace;
  /* Radius */
  --r-sm:6px; --r-md:8px; --r-lg:12px; --r-xl:16px; --r-pill:9999px;
  /* Spacing (4px base) */
  --s-xxs:4px; --s-xs:8px; --s-sm:12px; --s-md:16px; --s-lg:24px; --s-xl:32px; --s-xxl:48px; --s-section:96px;
  --maxw:1200px;
  font-size:16px;
}
*{margin:0;padding:0;box-sizing:border-box;}
html{scroll-behavior:smooth;}
body{font-family:var(--font-body);background:var(--canvas);color:var(--body);-webkit-font-smoothing:antialiased;line-height:1.55;}
h1,h2,h3,.display{font-family:var(--font-display);font-weight:500;color:var(--ink);letter-spacing:-0.02em;line-height:1.1;}
h4,h5{font-family:var(--font-body);font-weight:500;color:var(--ink);}
a{color:inherit;text-decoration:none;}
.wrap{max-width:var(--maxw);margin:0 auto;padding:0 32px;}
.eyebrow{font-family:var(--font-body);font-size:12px;font-weight:500;letter-spacing:1.5px;text-transform:uppercase;color:var(--muted);}
/* Buttons — DESIGN.md: default + active only, no hover lift */
.btn{display:inline-flex;align-items:center;gap:8px;font-family:var(--font-body);font-weight:500;font-size:14px;border-radius:var(--r-md);padding:12px 20px;cursor:pointer;border:1px solid transparent;transition:background .15s,border-color .15s;}
.btn-primary{background:var(--primary);color:var(--on-primary);}
.btn-primary:active{background:var(--primary-active);}
.btn-secondary{background:var(--canvas);color:var(--ink);border-color:var(--hairline);}
.btn-on-teal{background:var(--canvas);color:var(--ink);}
.reveal{opacity:0;transform:translateY(18px);transition:opacity .6s ease,transform .6s ease;}
.reveal.in{opacity:1;transform:none;}
@media (prefers-reduced-motion:reduce){.reveal{transition:none;opacity:1;transform:none;}}
.section{padding:var(--s-section) 0;}
@media (max-width:900px){.section{padding:64px 0;}}
```

> Remaining section-specific CSS (nav, hero, marquee, stats, features, how, about, cta, footer, chart) is added by each component task below, appended to this same file under a labelled comment block. Keep one stylesheet (spec decision a).

- [ ] **Step 2: Commit**

```bash
git add src/styles/global.css
git commit -m "feat(style): add DESIGN.md token layer + base styles (teal accent)"
```

---

## Task 3: BaseLayout

**Files:**
- Create: `src/layouts/BaseLayout.astro`

- [ ] **Step 1: Write the layout.** Carries `<head>` (meta/OG/Twitter/canonical/favicon, fonts, `theme-color` = cream), imports global.css, exposes props for per-page title/description/canonical, renders `<slot/>`. Favicon: keep the existing inline SVG mark from `index.html:17` but recolor the tile to ink and diamond to canvas (TKM diamond — unchanged shape).

```astro
---
import '../styles/global.css';
interface Props { title?: string; description?: string; path?: string; }
const {
  title = 'TKM-AI — Ship reliable AI agents',
  description = 'TKM-AI helps teams optimize and deploy LLM-based agents — cutting latency and cost while keeping quality high, from prototype to production.',
  path = '/',
} = Astro.props;
const canonical = new URL(path, Astro.site).href;
---
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>{title}</title>
  <meta name="description" content={description} />
  <meta name="theme-color" content="#faf9f5" />
  <link rel="canonical" href={canonical} />
  <meta property="og:type" content="website" />
  <meta property="og:title" content={title} />
  <meta property="og:description" content={description} />
  <meta property="og:url" content={canonical} />
  <meta name="twitter:card" content="summary_large_image" />
  <link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 32 32'%3E%3Crect width='32' height='32' rx='8' fill='%23141413'/%3E%3Crect x='10.5' y='10.5' width='11' height='11' rx='3' fill='%23faf9f5' transform='rotate(45 16 16)'/%3E%3C/svg%3E" />
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@500;600&family=Inter:wght@400;500;600&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet" />
</head>
<body>
  <slot />
</body>
</html>
```

- [ ] **Step 2: Commit** `git add src/layouts/BaseLayout.astro && git commit -m "feat: add BaseLayout (head, fonts, meta, favicon)"`

---

## Task 4: Nav (`top-nav`)

**Files:** Create `src/components/Nav.astro`; append nav CSS to `global.css`.

- [ ] **Step 1: Component.** Cream, sticky, 64px. TKM-AI diamond mark (CSS `::before` rotated square in `--ink`) + "TKM-AI" wordmark; links Product / How it works / About / **Blog** / Contact (root-relative `/#product`, `/blog`, etc.); `Get in touch` → `.btn .btn-primary` (teal) linking `/#contact`. Port structure from `index.html:276-289`, re-skin to tokens. Mobile (<900px): hide `.nav-links`.

- [ ] **Step 2: Verify** `npm run dev`, load `/`, confirm nav renders cream with teal CTA, diamond mark visible.

- [ ] **Step 3: Commit** `git add -A && git commit -m "feat: add Nav component"`

---

## Task 5: SavingsChart (dark product surface) + Hero

**Files:** Create `src/components/SavingsChart.astro`, `src/components/Hero.astro`; append hero + chart CSS to `global.css`.

- [ ] **Step 1: SavingsChart markup.** Port `index.html:305-330` verbatim (tabs, two `.rgroup` comparison blocks, volume slider, `.rnote`). Wrap in a dark card (`--surface-dark`, `--r-lg`, padding `--s-lg`). Keep all `data-*` hooks.

- [ ] **Step 2: SavingsChart `<script lang="ts">`.** Port the IIFE from `index.html:519-595` to TypeScript inside the component. Keep behavior identical: `WORK` data object, `tween`/`setNow`, `render(animate)`, tab clicks, slider `input`, `setTimeout(start, 60)` entrance (NOT rAF — throttled offscreen), bars via `transform: scaleX()`. Add types for the element maps. Keep the CLAUDE.md gotchas as comments. The `<script>` is bundled by Astro automatically (runs client-side).

```astro
<script>
  // Port of the savings-chart IIFE. Numbers are ILLUSTRATIVE (see .rnote), not measured.
  // Bars animate via transform:scaleX (width:100% resolves to 0 inside the flex track).
  // Entrance fires via setTimeout, not requestAnimationFrame (rAF is throttled offscreen).
  const root = document.getElementById('rchart');
  if (root) {
    const WORK: Record<string, {baseC:number;tkmC:number;baseL:number;tkmL:number}> = {
      chat:   { baseC: 4200,  tkmC: 1340, baseL: 1180, tkmL: 360 },
      rag:    { baseC: 6800,  tkmC: 2450, baseL: 1640, tkmL: 520 },
      agents: { baseC: 11500, tkmC: 3900, baseL: 2950, tkmL: 880 },
    };
    let cur = 'chat', volM = 1.0, started = false;
    const $ = (s:string) => root.querySelector(s) as HTMLElement;
    const fills:Record<string,HTMLElement> = { costBase:$('[data-fill=costBase]'), costTkm:$('[data-fill=costTkm]'), latBase:$('[data-fill=latBase]'), latTkm:$('[data-fill=latTkm]') };
    const vals:Record<string,HTMLElement> = { costBase:$('[data-val=costBase]'), costTkm:$('[data-val=costTkm]'), latBase:$('[data-val=latBase]'), latTkm:$('[data-val=latTkm]'), save:$('[data-val=save]'), vol:$('[data-val=vol]') };
    const chips = { cost:$('[data-chip=cost]'), lat:$('[data-chip=lat]') };
    const buttons = Array.from(root.querySelectorAll('.seg button')) as HTMLButtonElement[];
    const range = root.querySelector('#rvolRange') as HTMLInputElement;
    const money = (n:number) => '$' + Math.round(n).toLocaleString('en-US');
    const ms = (n:number) => Math.round(n).toLocaleString('en-US') + ' ms';
    function tween(el:HTMLElement, to:number, fmt:(n:number)=>string){
      const from = parseFloat(el.dataset.n || '0'); el.dataset.n = String(to);
      const t0 = performance.now(), dur = 620; let done = false;
      const finish = () => { if(!done){ done=true; el.textContent = fmt(to);} };
      (function step(t:number){ const k=Math.min(1,(t-t0)/dur), e=1-Math.pow(1-k,3); if(k>=1){finish();return;} el.textContent=fmt(from+(to-from)*e); requestAnimationFrame(step); })(performance.now());
      setTimeout(finish, dur+80);
    }
    function setNow(el:HTMLElement, to:number, fmt:(n:number)=>string){ el.dataset.n=String(to); el.textContent=fmt(to); }
    function render(animate:boolean){
      const d = WORK[cur]; const cb=d.baseC*volM, ct=d.tkmC*volM, saved=cb-ct;
      fills.costBase.style.transform='scaleX(1)'; fills.costTkm.style.transform='scaleX('+(d.tkmC/d.baseC)+')';
      fills.latBase.style.transform='scaleX(1)';  fills.latTkm.style.transform='scaleX('+(d.tkmL/d.baseL)+')';
      const put = animate ? tween : setNow;
      put(vals.costBase, cb, money); put(vals.costTkm, ct, money); put(vals.save, saved, money);
      if(animate){ tween(vals.latBase,d.baseL,ms); tween(vals.latTkm,d.tkmL,ms); } else { setNow(vals.latBase,d.baseL,ms); setNow(vals.latTkm,d.tkmL,ms); }
      vals.vol.textContent = volM.toFixed(1)+'M';
      chips.cost.innerHTML = '&#8595; ' + Math.round((1-d.tkmC/d.baseC)*100) + '%';
      chips.lat.innerHTML  = (d.baseL/d.tkmL).toFixed(1) + '&#215; faster';
    }
    buttons.forEach(b => b.addEventListener('click', () => { if(b.dataset.work===cur) return; buttons.forEach(x=>x.setAttribute('aria-selected', x===b?'true':'false')); cur=b.dataset.work!; render(true); }));
    range.addEventListener('input', () => { volM = +range.value/10; render(false); });
    const start = () => { if(started) return; started=true; render(true); };
    if (document.readyState==='loading') window.addEventListener('DOMContentLoaded', ()=>setTimeout(start,60)); else setTimeout(start,60);
  }
</script>
```

- [ ] **Step 3: Chart CSS (dark re-skin).** Append to `global.css`: tracks on `--surface-dark-soft`; `.rfill.base` = `color-mix(in srgb, var(--on-dark) 16%, transparent)`; `.rfill.tkm` = `var(--primary)` (teal); labels `--on-dark`/`--on-dark-soft`; `.seg button[aria-selected=true]` bg `--surface-dark-elevated`; numbers `--font-mono`; keep `@media (prefers-reduced-motion){.rfill{transition:none}}`. Port the geometry (`.rtrack`,`.rfill`,`.rrow`,`.seg`,`.rvol`) from `index.html:235-271`, swap colors to dark-surface tokens.

- [ ] **Step 4: Hero (`hero-band`).** Port `index.html:292-303` copy. 6/6 grid: left = pill badge (`accent-amber` tag), `h1` (`display-xl`: clamp serif, -1.5px), sub (`body-md`, `--body`), button row (`Get in touch` primary + `See how it works` secondary), hero-note. Right = `<SavingsChart />`. Mobile: 1-up, chart below.

- [ ] **Step 5: Verify** dev server: chart animates on load (bars grow, teal TKM bar shorter than baseline), tabs switch chat/rag/agents, slider updates volume + savings, hero is cream with serif h1.

- [ ] **Step 6: Commit** `git add -A && git commit -m "feat: add Hero + SavingsChart (dark product surface, teal bars)"`

---

## Task 6: TrustMarquee (data-driven)

**Files:** Create `src/components/TrustMarquee.astro`; append marquee CSS.

- [ ] **Step 1: Component.** Define `const logos = [{src,alt,name}, ...]` (8 entries from `index.html:342-349`). Render `{[...logos, ...logos].map(...)}` so the doubled set still loops at `-50%` but there is ONE list. Soft-cream band (`--surface-soft`). Logos grayscale via CSS filter; names `--muted`. Port `.marquee`/`.marquee-track`/`@keyframes marquee-scroll`/mask from `index.html:117-131`. Keep `:hover` pause + reduced-motion stop.

- [ ] **Step 2: Verify** marquee scrolls seamlessly (no jump at loop point), pauses on hover.

- [ ] **Step 3: Commit** `git add -A && git commit -m "feat: add TrustMarquee (data-driven doubled logos)"`

---

## Task 7: StatsBand

**Files:** Create `src/components/StatsBand.astro`; append stats CSS.

- [ ] **Step 1: Component.** `const stats=[{label:'Cost',desc...},{label:'Speed'...},{label:'Scale'...}]` from `index.html:366-368`. Band background `--surface-cream-strong`. Pillar words as **serif `display-md` (36px, weight 500)** — not the old 800 sans. 3-up grid, hairline dividers; 1-up at <560px. `.reveal` on the band.

- [ ] **Step 2: Verify** three pillars render in serif, cream-strong band, dividers between.

- [ ] **Step 3: Commit** `git add -A && git commit -m "feat: add StatsBand (serif pillars, cream-strong band)"`

---

## Task 8: Features (`feature-card` grid)

**Files:** Create `src/components/Features.astro`; append features CSS.

- [ ] **Step 1: Component.** `const features=[{icon,title,desc,tags}, ...]` (4 entries from `index.html:380-403`; keep inline SVG icon strings, render with `set:html`). Section on cream; cards `--surface-card`, `--r-lg`, padding `--s-xl`, icon tile `--surface-cream-strong`, title `title-md`, body `body-md`, tags in `--font-mono` with hairline border. Grid 2-up desktop → 1-up <560px. NO hover lift/shadow (DESIGN.md). Section head: eyebrow + serif h2 + intro (`index.html:374-378`).

- [ ] **Step 2: Verify** 4 cream cards, 2-up, icons render, no hover motion.

- [ ] **Step 3: Commit** `git add -A && git commit -m "feat: add Features (cream feature-card grid)"`

---

## Task 9: HowItWorks (dark band)

**Files:** Create `src/components/HowItWorks.astro`; append how CSS.

- [ ] **Step 1: Component.** `const steps=[{n:'01',title,desc}, ...]` (3 from `index.html:416-433`). Full dark band (`--surface-dark`, `--r-xl`, padding 64px), eyebrow `--on-dark-soft`, serif h2 `--on-dark`, step titles `title-md` `--on-dark`, body `--on-dark-soft`, the `.line` accent in teal. 3-up → 1-up mobile.

- [ ] **Step 2: Verify** dark band with three steps, teal accent line, readable on dark.

- [ ] **Step 3: Commit** `git add -A && git commit -m "feat: add HowItWorks (dark band)"`

---

## Task 10: About

**Files:** Create `src/components/About.astro`; append about CSS.

- [ ] **Step 1: Component.** Port `index.html:439-461`. Left: eyebrow + serif h2 + lead + `about-card` (cream, hairline rows: Focus / What we tackle / Deployment / Stage). Right: 2×2 value cards (`--surface-card`, `--r-lg`) — Cost-aware / Production-first / Model-agnostic / Developer-friendly. `const values=[...]`. Move the inline styles from the original into `global.css` classes.

- [ ] **Step 2: Verify** about grid renders, card rows + 4 value cards.

- [ ] **Step 3: Commit** `git add -A && git commit -m "feat: add About"`

---

## Task 11: CTA (teal callout + mailto form)

**Files:** Create `src/components/CTA.astro`; append cta CSS.

- [ ] **Step 1: Component.** Port `index.html:464-478`. Full-bleed **teal** callout (`--primary`, `--r-xl`, padding `--s-xxl`/64px), white h2 (serif) + sub. Form `#contactForm`: email input (`text-input` style, focus ring teal) + `Get in touch` button = `.btn .btn-on-teal` (cream on teal). Fine print with `mailto:hello@tkm-ai.com` link in `--on-primary`.

- [ ] **Step 2: mailto `<script>`.** Port `index.html:506-516` (preventDefault, build `mailto:hello@tkm-ai.com?subject=...&body=...`, set `window.location.href`). TS-typed element lookups.

- [ ] **Step 3: Verify** submitting the form opens a `mailto:` with prefilled subject/body; teal card, cream button.

- [ ] **Step 4: Commit** `git add -A && git commit -m "feat: add CTA (teal callout + mailto form)"`

---

## Task 12: Footer (dark)

**Files:** Create `src/components/Footer.astro`; append footer CSS.

- [ ] **Step 1: Component.** Port `index.html:481-496` but **dark** (`--surface-dark`, padding 64px), text `--on-dark-soft`, headings `--on-dark`, diamond mark + "TKM-AI" wordmark in `--on-dark`. Columns Product / Company. Bottom row: `© 2026 TKM-AI` / `tkm-ai.com`. Blog link added to a column.

- [ ] **Step 2: Verify** footer is dark, links present, mark visible.

- [ ] **Step 3: Commit** `git add -A && git commit -m "feat: add dark Footer"`

---

## Task 13: Homepage composition + scroll-reveal

**Files:** Create `src/pages/index.astro`.

- [ ] **Step 1: Compose.** Import BaseLayout + all components; render in band-pacing order: Nav, Hero, TrustMarquee, StatsBand, Features, HowItWorks, About, CTA, Footer. Add `class="reveal"` to the section wrappers that should fade in (Stats, Features, How, About, CTA) — match the originals.

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
import Nav from '../components/Nav.astro';
import Hero from '../components/Hero.astro';
import TrustMarquee from '../components/TrustMarquee.astro';
import StatsBand from '../components/StatsBand.astro';
import Features from '../components/Features.astro';
import HowItWorks from '../components/HowItWorks.astro';
import About from '../components/About.astro';
import CTA from '../components/CTA.astro';
import Footer from '../components/Footer.astro';
---
<BaseLayout>
  <Nav />
  <main>
    <Hero />
    <TrustMarquee />
    <StatsBand />
    <Features />
    <HowItWorks />
    <About />
    <CTA />
  </main>
  <Footer />
</BaseLayout>
<script>
  const io = new IntersectionObserver((entries) => {
    entries.forEach((e) => { if (e.isIntersecting) { e.target.classList.add('in'); io.unobserve(e.target); } });
  }, { threshold: 0.12 });
  document.querySelectorAll('.reveal').forEach((el) => io.observe(el));
</script>
```

- [ ] **Step 2: Verify** full homepage renders top-to-bottom with correct band alternation (cream → soft → cream-strong → cream → DARK → cream → TEAL → DARK), reveals fire on scroll.

- [ ] **Step 3: Commit** `git add -A && git commit -m "feat: compose homepage + scroll-reveal"`

---

## Task 14: Blog content collection + sample post

**Files:** Create `src/content.config.ts`, `src/content/blog/hello-world.md`.

- [ ] **Step 1: Collection config** (Astro 5 Content Layer — confirm API against current docs).

```ts
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

const blog = defineCollection({
  loader: glob({ pattern: '**/*.md', base: './src/content/blog' }),
  schema: z.object({
    title: z.string(),
    description: z.string(),
    pubDate: z.coerce.date(),
    draft: z.boolean().default(false),
    tags: z.array(z.string()).optional(),
  }),
});
export const collections = { blog };
```

- [ ] **Step 2: Sample post `hello-world.md`.**

```markdown
---
title: "Why agent cost and latency are the same problem"
description: "A short note on why optimizing LLM agents for cost usually makes them faster too — and vice versa."
pubDate: 2026-06-06
tags: ["agents", "cost", "latency"]
---

Most teams treat token spend and response time as separate dials. In practice they
move together: the same prompt bloat that inflates your bill also slows every call…

*(Illustrative example — replace with real content.)*
```

- [ ] **Step 3: Verify** `npm run build` picks up the collection with no schema error.

- [ ] **Step 4: Commit** `git add -A && git commit -m "feat: add blog content collection + sample post"`

---

## Task 15: Blog listing + post pages

**Files:** Create `src/pages/blog/index.astro`, `src/pages/blog/[...slug].astro`; append blog CSS.

- [ ] **Step 1: Listing.** `getCollection('blog')`, filter `!data.draft`, sort by `pubDate` desc; render Nav + a cream editorial list (serif post titles, `body-md` descriptions, muted dates) + Footer, inside BaseLayout (`title="Blog — TKM-AI"`, `path="/blog"`).

- [ ] **Step 2: Post page.** `getStaticPaths()` from the collection; `render(entry)` → `<Content />`; article layout with serif h1, muted date, prose body using the editorial type scale; Nav + Footer; per-post canonical/OG via BaseLayout props.

- [ ] **Step 3: Verify** `/blog` lists the sample post; clicking opens `/blog/hello-world` rendered with the editorial styling.

- [ ] **Step 4: Commit** `git add -A && git commit -m "feat: add blog listing + post pages"`

---

## Task 16: Retire old index.html, rewrite CLAUDE.md + README

**Files:** Delete `index.html`, `CNAME`, `.nojekyll`; modify `CLAUDE.md`, `README.md`.

- [ ] **Step 1: Remove superseded files.** `git rm index.html CNAME .nojekyll` (Cloudflare Pages doesn't need CNAME/.nojekyll; custom domain handled in Pages).

- [ ] **Step 2: Rewrite `CLAUDE.md`** for the new reality: Astro project, `npm run dev`/`build`, components in `src/components`, tokens in `src/styles/global.css` (DESIGN.md system, teal accent), blog in `src/content/blog`, deploy = Cloudflare Pages (build `npm run build`, output `dist`), DNS on Cloudflare. Remove the GitHub Pages/Jekyll/CNAME operational notes; keep the honest-content rule and the chart/marquee gotchas.

- [ ] **Step 3: Rewrite `README.md`** with quickstart (`npm install`, `npm run dev`), structure, and deploy summary.

- [ ] **Step 4: Verify** `npm run build` still succeeds after removals; `grep -ri "github pages" CLAUDE.md` returns nothing stale.

- [ ] **Step 5: Commit** `git add -A && git commit -m "chore: retire static index.html + GitHub Pages files; rewrite CLAUDE.md/README for Astro"`

---

## Task 17: Full build + check + visual parity pass

- [ ] **Step 1:** `npx astro check` → expected: 0 errors. Fix any type/template diagnostics.
- [ ] **Step 2:** `npm run build` → expected: succeeds, emits `dist/` with `index.html`, `/blog/index.html`, `/blog/hello-world/index.html`, `sitemap-index.xml`, hashed JS for the chart.
- [ ] **Step 3:** `npm run preview`; review at desktop / 768 / mobile widths against DESIGN.md (cream canvas, serif display with negative tracking, teal accent only where coral would be, dark how/footer bands, teal CTA). Confirm chart tabs/slider/animation, marquee loop, reveals, mailto. Fix visual gaps.
- [ ] **Step 4: Commit** any fixes `git commit -am "fix: visual parity pass against DESIGN.md"`

---

## Task 18: Push branch + open/merge

- [ ] **Step 1:** `git push -u origin astro-migration`
- [ ] **Step 2:** Decide with user: open PR (`gh pr create`) for review, or fast-forward `main`. For deploy we need the build on `main` if using Pages git integration; for Wrangler direct upload (Task 19) the branch is enough.

---

## Task 19: Deploy to Cloudflare Pages (Wrangler direct upload)

> **Auth required** — no Cloudflare credentials in the environment. The user must either run `npx wrangler login` (interactive, via `! npx wrangler login`) **or** provide a `CLOUDFLARE_API_TOKEN` (Pages:Edit + DNS:Edit + Account:Read) and `CLOUDFLARE_ACCOUNT_ID`. Wait for this before proceeding.

- [ ] **Step 1: Confirm auth.** `npx wrangler whoami` → expected: lists the account + email.
- [ ] **Step 2: Create the Pages project (once).** `npx wrangler pages project create tkm-ai --production-branch main`
- [ ] **Step 3: Deploy the build.** `npx wrangler pages deploy dist --project-name tkm-ai --branch main`
  Expected: prints a `https://<hash>.tkm-ai.pages.dev` URL.
- [ ] **Step 4: Validate the pages.dev URL.** `curl -sI https://<deploy>.pages.dev | head -1` → `HTTP/2 200`; open it and verify the live site + /blog.

---

## Task 20: Custom domain tkm-ai.com + DNS cutover + HTTPS

> The apex currently has GitHub Pages A/AAAA records (185.199.108–111.153 / 2606:50c0:8000–8003::153). Adding the Pages custom domain replaces these with a Cloudflare-managed record. Do this only after Task 19's pages.dev URL is validated.

- [ ] **Step 1: Add custom domain to the Pages project.** Via API (token must have Pages:Edit):
  `curl -X POST "https://api.cloudflare.com/client/v4/accounts/$CLOUDFLARE_ACCOUNT_ID/pages/projects/tkm-ai/domains" -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN" -H "Content-Type: application/json" -d '{"name":"tkm-ai.com"}'`
  (Repeat for `www.tkm-ai.com` if desired.) This triggers cert provisioning.
- [ ] **Step 2: Fix DNS.** In the `tkm-ai.com` zone, remove the GitHub Pages apex `A`/`AAAA` records and ensure Cloudflare's Pages CNAME/flattening points apex → `tkm-ai.pages.dev`. (List: `GET .../zones/{zone_id}/dns_records`; delete old A/AAAA; the custom-domain step usually auto-creates the CNAME — verify.) DNS-only (grey cloud) is fine; orange-proxy needs SSL mode "Full".
- [ ] **Step 3: Wait for cert + verify.** Poll `curl -sI https://tkm-ai.com | head -1` until `HTTP/2 200`; confirm the served HTML is the new Astro site (`curl -s https://tkm-ai.com | grep -o '<title>[^<]*'`).
- [ ] **Step 4: Verify blog + redirects** at `https://tkm-ai.com/blog`.
- [ ] **Step 5: Set up auto-deploy (recommended, optional).** Connect the GitHub repo to the Pages project in the Cloudflare dashboard (Settings → Builds & deployments → Connect to Git), production branch `main`, build `npm run build`, output `dist` — so future pushes auto-build. (Dashboard step; or leave on Wrangler direct-upload.)
- [ ] **Step 6: Final commit/tag** if any deploy config (e.g. `wrangler.toml`) was added.

---

## Self-Review notes (filled during writing)

- **Spec coverage:** §5 tokens → Task 2; §5 typography/interaction → Tasks 2–12; §6 structure → Tasks 1,3–15; §7 component mapping + band pacing → Tasks 4–13; §8 blog → Tasks 14–15; §9 chart → Task 5; §10 CSS → Task 2 + per-component; §11 deploy → Tasks 19–20; §12 preserved/changed → Tasks 3,5,11,12,16; §13 verification → Task 17; §14 out-of-scope → not built; §15 risks → noted (teal contrast check in Task 17, marquee doubling in Task 6, GH Pages retirement in Task 16, DNS cutover in Task 20). No gaps.
- **Placeholders:** none — code shown for config/token/chart/layout/collection/deploy; mechanical section ports reference exact `index.html` line ranges + token mapping.
- **Type consistency:** `WORK`, `render(animate)`, `tween/setNow`, `data-fill`/`data-val`/`data-chip` hooks consistent between Task 5 markup and script; `blog` collection name + fields consistent between Tasks 14 and 15.
