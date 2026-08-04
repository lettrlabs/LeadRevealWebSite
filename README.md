# LeadRevealWebSite

A **test customer site** for the LettrLabs Lead Reveal container install, styled after
lettrlabs.com so the page looks like a real marketing property while the container and
beacon path are exercised from a real browser on a real HTTPS origin.

Not the real LettrLabs website. Every page carries `noindex, nofollow`, and the footer says
so on every page. Do not link to it publicly.

Brand assets are copied from the live site: `assets/img/favicon.ico`, `favicon-256.png` and
`logo.webp` come from the Webflow CDN. Colors are the live palette (`#f05a2a` orange,
`#130b46` ink, `#e4e6f1` border, `#f3f5fb` tint) and the typeface is Plus Jakarta Sans off
Google Fonts. The hero collage is CSS, not their marketing imagery.

## Layout

Built with Jekyll, which GitHub Pages runs natively (no Actions workflow, no build step).

```
_config.yml               site settings + the Lead Reveal public key
_layouts/default.html     the master page: <head>, nav, footer
index.html                /
products.html             /products/
milling.html              /milling/
contacts.html             /contacts/
assets/css/site.css       all styling, self-hosted (no CDN)
```

Every page is front matter plus a body and inherits `_layouts/default.html`, so the
`<head>` — including the Lead Reveal script tag — is declared in exactly one place.

## Installing the Lead Reveal container

Set the public key in `_config.yml`:

```yaml
lead_reveal:
  base_url: https://app-dev.lettrlabs.com/api/lead-reveal/pixels/configuration/
  public_key: "PASTE_THE_PUBLIC_KEY_HERE"
```

The layout emits the tag only when `public_key` is non-empty:

```html
<script async src="{base_url}{public_key}.js"></script>
```

`base_url` must match the environment whose App is serving the container. It is the same
value as the App's `LeadReveal__PixelConfigBaseUrl` env var — which **defaults to the prod
host**, so always set it explicitly here.

To hand-place a literal tag instead, edit the `<head>` in `_layouts/default.html` directly.

## Enabling GitHub Pages

Settings ▸ Pages ▸ Build and deployment ▸ Source = **Deploy from a branch** ▸ `main` / `/ (root)`.

Or:

```
gh api -X POST repos/lettrlabs/LeadRevealWebSite/pages -f "source[branch]=main" -f "source[path]=/"
```

Published at `https://lettrlabs.github.io/LeadRevealWebSite/`.

## Things that will bite you

**The pixel must be registered against `lettrlabs.github.io`, not the path.** A browser's
`Origin` header is scheme + host only — the `/LeadRevealWebSite/` path is not part of it. The
`/hit` receiver compares `Origin` to the pixel's stored `Domain`, so a pixel registered as
`lettrlabs.github.io/LeadRevealWebSite` will log `DroppedOriginMismatch` once the origin
guards are re-enabled (they are currently commented out on `feature/lead-reveal-overhaul`).

**CORS is a Function App platform setting.** The App Functions register no CORS middleware,
so `https://lettrlabs.github.io` has to be on the target Function App's CORS allowlist or
the `/hit` preflight fails.

**`baseurl` is load-bearing.** This is a project page, served from a subpath. Every internal
link goes through `{{ '/path/' | relative_url }}`. A bare `href="/products/"` 404s.

**GitHub Pages ignores `_headers`.** That file is a Cloudflare Pages feature. There is no way
to set response headers on GitHub Pages, so the CSP and `Referrer-Policy` from the sibling
Cloudflare test site do not apply here. If header control matters, host on Cloudflare Pages or
an Azure Static Web App (`staticwebapp.config.json`) instead.

**A fresh test domain will report `bodyLoaded:false`.** Inbound Insight only generates the real
pixel body after install plus real traffic, so an untrafficked test host gets a stub. Expected —
this site validates the container and beacon path, not end-to-end lead resolution.

## Local preview

```
bundle exec jekyll serve --baseurl ""
```

Or skip Jekyll entirely and just push — GitHub builds on every commit to `main` and emails on
build failure.
