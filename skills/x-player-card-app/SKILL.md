---
name: x-player-card-app
description: Use when someone wants a small web app or interactive tool that runs embedded inside an X (Twitter) post via a player card, or asks how a post can show a usable app inline.
---

# Build an app that runs inside an X post (player card)

An X post can't host code. It links to a web page whose `<head>` declares a **player card**; X then renders that page in an iframe inside the post, so people use it without leaving X.

## 1. Gather
- What the app does (one screen, client-side, no login).
- Framework/host (default: Next.js App Router + TypeScript + Tailwind on Vercel).
- The public HTTPS URL where it will live and the X handle for `twitter:site` (optional).

## 2. Build the page
1. One public route, e.g. `/tools/<slug>`, served over HTTPS.
2. Add these tags (absolute URLs, built from a `SITE_URL` env var; in Next.js via `metadata`/`generateMetadata`):
```html
<meta name="twitter:card" content="player">
<meta name="twitter:site" content="@handle">
<meta name="twitter:title" content="...">
<meta name="twitter:description" content="...">
<meta name="twitter:image" content="https://.../preview.png">
<meta name="twitter:player" content="https://.../tools/slug">
<meta name="twitter:player:width" content="1200">
<meta name="twitter:player:height" content="900">
<meta property="og:title" content="...">
<meta property="og:description" content="...">
<meta property="og:image" content="https://.../preview.png">
<meta property="og:url" content="https://.../tools/slug">
```
3. Allow framing by X: send no `X-Frame-Options: DENY/SAMEORIGIN`; set `Content-Security-Policy: frame-ancestors 'self' https://x.com https://twitter.com https://*.x.com https://*.twitter.com`.
4. Provide a preview image (1200x630), e.g. generated with `next/og`.
5. Design for the iframe: fits the declared width/height and mobile, no horizontal scroll, no popups or `window.open` for core features; downloads via `<a download>`/Blob; no third-party cookies or login.

## 3. Verify
- `curl -s <URL> | grep -E 'twitter:|og:'` shows every tag with absolute HTTPS URLs.
- `curl -sI <URL>` shows the frame-ancestors header and no blocking X-Frame-Options.
- Load the page inside a local test iframe to confirm it works framed.

## 4. Publish
- Post the URL in an X post. X caches cards, so changes can take time; add a query string (`?v=2`) to force a refresh.
- X has historically validated or approved player cards; if the post shows only a normal preview, the tags may be fine and the card just isn't enabled for that domain yet. Say so instead of guessing at other causes.

## Deliverable
Full code file by file, a README with deploy and verify steps, and the live URL once deployed.
