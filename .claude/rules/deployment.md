# Deployment and Build

## Dual Deployment Targets

This app is configured for two independent deployment targets:

| Target | Config | Output |
|---|---|---|
| Vercel (primary) | `vercel.json` | standard Vite build |
| GitHub Pages (fallback) | `vite.config.js` base + SPA hack in `index.html` | `_site/` directory |

Both targets must remain functional. Do not change `vite.config.js` base or outDir without
verifying GitHub Pages still works.

## Vercel Configuration (`vercel.json`)

**Known bug**: `buildCommand` is set to `"npm run vercel"` but no `vercel` script exists in
`package.json`. Warn the user before any Vercel deployment work and offer to fix this first.

The framework is declared as `"eleventy"` which is incorrect — this is a Vite/React app.
This mismatch may cause Vercel to apply wrong build assumptions.

Security headers are enforced in `vercel.json` (CSP, X-Frame-Options, Permissions-Policy).
Do not remove them. If adding new external resources (fonts, CDNs), the CSP `connect-src`
and `script-src` directives need updating.

## GitHub Pages SPA Routing

`index.html` contains a runtime script that restores deep-link URLs after GitHub Pages
redirects 404s through `public/404.html`. **Do not remove this script.**

```html
<!-- index.html: this script must stay — GitHub Pages SPA routing -->
<script>(function(l) { if (l.search[1] === '/') { ... } })(window.location)</script>
```

`public/404.html` redirects unknown paths back to `index.html` with the path encoded as
a query param. Both files must stay in sync.

## Build Output

Vite builds to `_site/` (not the default `dist/`). This matches the GitHub Actions artifact
upload expectation. Do not change `outDir` in `vite.config.js`.

## Build Order for Deployment

```bash
bun run build:index   # must run first — regenerates notes/index.json
bun run build         # Vite build → _site/
```

Running `bun run build` without `build:index` first produces a stale or missing index,
which causes the app to fail to load notes.

## Environment and Basename

The router basename is detected at runtime in `src/main.jsx`:
- On `*.github.io` hosts: `/${repoName}` extracted from `window.location.pathname`
- Elsewhere: `Config.basename` from `config.json` (undefined → `/`)

No environment variables or `.env` files are used. Configuration is entirely in `config.json`.
