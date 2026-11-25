---
name: x-player-card-app
description: Use when someone wants a small web app or interactive tool that runs embedded inside an X (Twitter) post via a player card, or asks how a post can show a usable app inline. Works with any stack (plain HTML, React, Next.js, Angular, Vue, Nuxt, Svelte, Astro, server-rendered apps).
---

# Build an app that runs inside an X post (player card)

An X post can't host code. It links to a web page whose HTML declares a **player card**; X then renders that page in an iframe inside the post, so people use it without leaving X. The technique is stack-agnostic: any framework works as long as three things are true.

## The three requirements (any stack)
1. **The card tags are in the raw HTML** that the server returns. X's crawler does not run JavaScript, so tags injected at runtime by a client-side router or a `<Helmet>`-style library are invisible to it. Use a static `index.html`, server-side rendering, or prerendering.
2. **The page can be framed by X.** No `X-Frame-Options: DENY` or `SAMEORIGIN`, and a CSP that allows X as a frame ancestor.
3. **Everything is HTTPS with absolute URLs** (page, `twitter:player`, preview image).

## 1. Gather
- What the app does (one screen, client-side, no login).
- The stack and host the user wants. If they have no preference, pick the simplest that fits (a static Vite build or plain HTML is enough for most tools).
- The public HTTPS URL where it will live and, optionally, the X handle for `twitter:site`.

## 2. Card tags (same for every stack)
```html
<meta name="twitter:card" content="player">
<meta name="twitter:site" content="@handle">
<meta name="twitter:title" content="...">
<meta name="twitter:description" content="...">
<meta name="twitter:image" content="https://example.com/preview.png">
<meta name="twitter:player" content="https://example.com/tools/slug">
<meta name="twitter:player:width" content="1200">
<meta name="twitter:player:height" content="900">
<meta property="og:title" content="...">
<meta property="og:description" content="...">
<meta property="og:image" content="https://example.com/preview.png">
<meta property="og:url" content="https://example.com/tools/slug">
```
`twitter:player` points to the page (or a dedicated `/embed` route) that X loads in the iframe. Width/height set the aspect ratio X uses.

## 3. Framing header (same for every stack)
```
Content-Security-Policy: frame-ancestors 'self' https://x.com https://twitter.com https://*.x.com https://*.twitter.com
```
And do not send `X-Frame-Options`.

## 4. Per-stack instructions

### Plain HTML / any static site
Put the tags directly in the page's `<head>`. Serve the headers from the host (see section 5).

### React (Vite, CRA or any SPA)
- SPAs ship one `index.html`; put the tags there (`index.html` at the project root in Vite, `public/index.html` in CRA). Use absolute URLs; in Vite you can template them with `%VITE_SITE_URL%`.
- If the app has several tools that each need their own card, either give each its own HTML entry (Vite multi-page: `build.rollupOptions.input`), or prerender routes (`vite-plugin-ssr`/Vike, `react-snap`, or similar). `react-helmet` alone is not enough because it runs on the client.

### Next.js
- App Router: export `metadata` or `generateMetadata` from the page. The `twitter` field does not cover every player tag, so add the missing ones via `metadata.other` (`'twitter:player'`, `'twitter:player:width'`, `'twitter:player:height'`). Set `metadataBase` from a `SITE_URL` env var.
- Pages Router: add the `<meta>` tags in `next/head` in that page (it is server-rendered, so crawlers see them).
- Headers: `headers()` in `next.config.js` or `middleware.ts`.
- Preview image: `opengraph-image.tsx` with `next/og`.

### Angular
- Single card for the whole app: put the tags in `src/index.html`.
- Per-route cards: use Angular SSR (`ng add @angular/ssr`) or prerendering (`"prerender": true` with the route listed), and set tags in the component with the `Meta` service (`meta.updateTag({ name: 'twitter:card', content: 'player' })`). Without SSR or prerender, `Meta` updates only happen in the browser and X won't see them.
- Headers: set them in the SSR `server.ts` (Express) or at the host.

### Vue / Nuxt
- Vue + Vite SPA: tags in `index.html`, same caveats as React.
- Nuxt: `useSeoMeta({ twitterCard: 'player', ... })` plus `useHead({ meta: [{ name: 'twitter:player', content: '...' }, ...] })` for the player tags. SSR or `nuxi generate` makes them crawlable. Headers via `routeRules` in `nuxt.config` (`headers: { 'Content-Security-Policy': ... }`).

### Svelte / SvelteKit
- SvelteKit: put the tags in `<svelte:head>` in the route's `+page.svelte` and keep SSR or `export const prerender = true`. Headers via `setHeaders` in `+page.server.js`/`+layout.server.js` or `hooks.server.js`.
- Plain Svelte + Vite: tags in `index.html`.

### Astro
Tags in the page's `<head>` (or a layout). Astro renders static HTML by default, so crawlers see them. Headers at the host.

### Server-rendered apps (Express, Django, Rails, Laravel, Go, etc.)
Render the tags in the page template and set the CSP header in the response (middleware or the route handler).

## 5. Per-host header config
- **Vercel**: `vercel.json` → `"headers": [{ "source": "/(.*)", "headers": [{ "key": "Content-Security-Policy", "value": "frame-ancestors ..." }] }]` (or framework config).
- **Netlify**: `_headers` file → `/*` then `  Content-Security-Policy: frame-ancestors ...`.
- **Cloudflare Pages**: `_headers` file, same format as Netlify.
- **GitHub Pages**: can't set custom headers. It sends no `X-Frame-Options`, so framing usually works, but you can't add the CSP.
- **nginx**: `add_header Content-Security-Policy "frame-ancestors ..." always;` and make sure nothing adds `X-Frame-Options`.

## 6. Design for the iframe
Fits the declared width/height and mobile, no horizontal scroll, no popups or `window.open` for core features, downloads via `<a download>`/Blob, no login or third-party cookies (they are blocked in iframes).

## 7. Verify
- `curl -s <URL> | grep -E 'twitter:|og:'` shows every tag in the raw HTML with absolute HTTPS URLs. If nothing shows up, the tags are being added client-side; fix with prerender/SSR or `index.html`.
- `curl -sI <URL>` shows the frame-ancestors header and no blocking `X-Frame-Options`.
- Load the page inside a local test iframe to confirm it works framed.

## 8. Publish
- Post the URL in an X post. X caches cards, so changes can take time; add a query string (`?v=2`) to force a refresh.
- X has historically validated or approved player cards. If the post shows only a normal preview, the tags may be fine and the card just isn't enabled for that domain yet. Say so instead of guessing at other causes.

## Deliverable
Full code file by file for the chosen stack, the host config for the headers, a README with deploy and verify steps, and the live URL once deployed.
