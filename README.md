# tkm-ai.com

Marketing site for **TKM-AI** — a platform to optimize, deploy, and observe LLM-based agents.

Built with [Astro](https://astro.build), re-skinned to the editorial design system in
[`DESIGN.md`](./DESIGN.md) (warm cream + dark surfaces + serif display, deep-teal accent).
Deployed on Cloudflare Pages at [tkm-ai.com](https://tkm-ai.com).

## Quickstart

```bash
npm install
npm run dev        # http://localhost:4321
```

## Scripts

| Command | What it does |
|---|---|
| `npm run dev` | Local dev server with hot reload |
| `npm run build` | Static build to `dist/` |
| `npm run preview` | Serve the built `dist/` locally |
| `npm run check` | `astro check` — TypeScript + template diagnostics |

## Structure

- `src/pages/` — routes (`index.astro` homepage, `blog/`)
- `src/components/` — one component per page section
- `src/layouts/BaseLayout.astro` — `<head>`, meta/OG, fonts, favicon
- `src/styles/global.css` — the design token layer + all component CSS
- `src/content/blog/` — Markdown blog posts (typed by `src/content.config.ts`)
- `public/assets/logos/` — agent logos for the "Support all agents" marquee
- `DESIGN.md` — the design system (source of truth for tokens and components)

## Notes

- The hero cost/speed chart is an **illustrative estimate** — figures are not measured benchmarks.
- The contact form opens the visitor's mail client to `hello@tkm-ai.com` (no backend).

## Deploy

Cloudflare Pages (project `tkm-ai`): build `npm run build`, output `dist`. DNS on Cloudflare,
custom domain attached to the Pages project. See `CLAUDE.md` for details.
