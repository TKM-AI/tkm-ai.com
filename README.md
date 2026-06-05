# tkm-ai.com

Landing page for **TKM-AI** — a platform to optimize, deploy, and observe LLM-based agents.

Static single-page site (no build step). Hosted on GitHub Pages and served at
[tkm-ai.com](https://tkm-ai.com).

## Structure

- `index.html` — the entire landing page (HTML + CSS + JS inline)
- `assets/logos/` — agent/product logos used in the "Support all agents" marquee
- `CNAME` — custom domain for GitHub Pages

## Local preview

Open `index.html` directly in a browser, or serve the folder:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Notes

- The hero cost/speed chart is an **illustrative estimate** — figures are not measured benchmarks.
- The contact form opens the visitor's mail client to `hello@tkm-ai.com` (no backend).
