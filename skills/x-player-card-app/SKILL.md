---
name: x-player-card-app
description: >-
  Use when someone wants a small web app or interactive tool that runs
  embedded inside an X (Twitter) post via a player card, or asks how a
  post can show a usable app inline.
---

# X player card app

A player card is not an app uploaded into a post. The post contains a HTTPS
link to a public page. That page's HTML includes Twitter Card meta tags that
tell X to load the page in an iframe inside the post, so people can use the
tool without leaving X.

## When to use this

- Build or adapt a small web app so it can run inside an X post.
- Add or fix `twitter:card` / `twitter:player` tags and iframe headers.
- Debug why a link previews as a normal summary card instead of an
  interactive player.

## Required page setup

Serve the page over HTTPS on a public URL. Put these tags in the document
`<head>` (or the framework's metadata API). All URLs must be absolute and
HTTPS:

```html
<meta name="twitter:card" content="player" />
<meta name="twitter:site" content="@handle" />
<meta name="twitter:title" content="Tool title" />
<meta name="twitter:description" content="One-line description" />
<meta name="twitter:image" content="https://example.com/path/preview.png" />
<meta name="twitter:player" content="https://example.com/path" />
<meta name="twitter:player:width" content="1200" />
<meta name="twitter:player:height" content="900" />
```

Also set the Open Graph equivalents (`og:title`, `og:description`,
`og:image`, `og:url`) for other clients and for fallbacks.

Notes:

- `twitter:site` is the @handle the card should be attributed to. Keep it as
  a placeholder like `@handle` until the site owner provides their handle.
- `twitter:player` is the HTTPS URL of the iframe document. It can be the
  same page or a dedicated embed route. It must not trigger mixed-content
  warnings.
- `twitter:image` is the fallback poster shown when a client cannot render
  the iframe. Use an absolute HTTPS image; 1200×630 is a solid preview size.
  Prefer making the image match the player aspect when possible.
- `twitter:player:width` and `twitter:player:height` are the iframe's
  intrinsic size in pixels (for example 1200×900). X will scale it to the
  timeline.

If the stack is Next.js (App Router), set these with `metadata` or
`generateMetadata`, and build absolute URLs from an env var such as
`NEXT_PUBLIC_SITE_URL`. A preview image can be generated with `next/og`
(`ImageResponse`) at a route like `opengraph-image`.

## Allow X to embed the page

The player URL must be iframe-able on x.com and twitter.com:

- Do not send `X-Frame-Options: DENY` or `X-Frame-Options: SAMEORIGIN`.
- Send a Content-Security-Policy `frame-ancestors` directive that allows the
  page itself and X, for example:

  `Content-Security-Policy: frame-ancestors 'self' https://x.com https://twitter.com https://*.x.com https://*.twitter.com`

Set this in the host config, middleware, or platform headers. Watch for a
default from the framework, CDN, or host (for example Next.js or a security
plugin) that adds a restrictive `X-Frame-Options` or `frame-ancestors`.

The player document should stay on HTTPS, avoid mixed content, and not
depend on a login wall, cookie banner, or bot challenge that breaks inside
an iframe.

## UX inside the iframe

- Design for a tight iframe (the 1200×900 intrinsic size, and narrower
  mobile widths). No horizontal overflow.
- Keep the primary flow inside the iframe. Do not use `window.open` or
  popups for core actions.
- For downloads, use an `<a download>` link or a Blob URL, not a new window.
- Prefer a client-only tool when possible: no required backend or login just
  to try it.

## Check the tags and headers

Before posting, confirm the live URL returns the tags and embedding headers:

```bash
curl -sL https://example.com/path | grep -i twitter:
curl -sI https://example.com/path
```

On the headers, confirm there is no blocking `X-Frame-Options`, and that
`Content-Security-Policy` (if present) allows the `frame-ancestors` list
above.

## If the post shows a normal preview

This is common. Causes, in order:

1. **Cache.** X caches cards. After a tag change, try a fresh URL (new path
   or a cache-busting query) or wait; changing the title can help you see
   whether a new scrape happened.
2. **Validation / approval.** Player cards may need X to validate or
   whitelist the domain. Until that happens, the same URL often appears as a
   regular summary or image preview. Use X's card validator if it is
   available for the account, and request player-card approval when the
   validator says the domain is not whitelisted.
3. **Missing or invalid tags.** Relative URLs, HTTP player/image URLs, or a
   missing `twitter:card` / `twitter:player` value will prevent a player
   from rendering.
4. **Blocked iframe.** Restrictive frame headers, mixed content, or a page
   that errors in an iframe will fail even if the tags look right.

Do not assume a successful `curl` of the meta tags means the timeline will
show the player immediately.

## Ship and test

1. Deploy the public HTTPS page.
2. Verify tags and headers with `curl`.
3. Post the page URL in an X post (a draft or test account is enough).
4. If only a normal preview appears, treat cache and player-card approval as
   the likely causes, then re-check headers and URLs.
