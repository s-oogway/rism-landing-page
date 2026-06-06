# Deployment

This is a campaign-only landing page served from GitHub Pages on a Rism
subdomain. It is intentionally not linked from the main Rism homepage.

## Custom domain

- **Target subdomain:** `salon.rism.co.jp`
- The `CNAME` file at the repo root pins this and tells GitHub Pages
  which hostname to serve.

## DNS record required

Add one CNAME record at the rism.co.jp DNS host:

| Field              | Value                |
| ------------------ | -------------------- |
| Type               | `CNAME`              |
| Host / Name        | `salon`              |
| Value / Target     | `s-oogway.github.io` |
| TTL                | default (auto / 1h)  |

Do **not** include the repository path in the DNS target — `s-oogway.github.io`
only, no `/rism-landing-page`. GitHub resolves the repo from the CNAME file.

DNS propagation can take from a few minutes up to ~24 hours depending on
the registrar and existing TTL.

## GitHub Pages settings

After the DNS record is in place:

1. Go to the repo → **Settings → Pages**.
2. Confirm **Custom domain** shows `salon.rism.co.jp` (GitHub will have
   read this from the `CNAME` file in the repo).
3. Wait for the DNS check to pass (small green status).
4. Tick **Enforce HTTPS** once GitHub allows it (it lights up after the
   DNS check completes and the certificate is issued — typically minutes
   to an hour after DNS resolves).

## Indexing

The page has:

```html
<meta name="robots" content="noindex, nofollow">
```

This is sufficient for a static landing page — no `robots.txt` is needed
and none should be added unless one already exists in the repo. Search
engines should not index the page, but **anyone with the URL can still
visit it** — treat the link as the access control.

Do **not** link this LP from `rism.co.jp` if it should remain
campaign-only.

## Analytics

GA4 is wired up but uses a placeholder Measurement ID `G-XXXXXXXXXX`. The
loader is guarded — while the placeholder is in place, no network requests
are made to Google and no events are sent. To enable:

1. Open `index.html`, find the `<script id="ga-init">` block at the bottom.
2. Replace `G-XXXXXXXXXX` with the real GA4 Measurement ID.
3. Re-deploy (push to `main`).

Events tracked once GA is real:

- `cta_click` — every click on the 「再来店導線について相談する」 CTA
- `form_start` — booking section (`§10`) becomes 40%+ visible (proxy; the
  TimeRex iframe is cross-origin so real input events inside it can't be
  observed from this page)
- `form_submit` — listens for `postMessage` events from the TimeRex origin.
  Best-effort only. For accurate booking-completion tracking, wire
  TimeRex's webhook to a server-side GA Measurement Protocol call.
- `scroll_75` — fires once per session at 75% scroll depth

## Booking form

The contact "form" is a **TimeRex inline widget** embedded in `§10`. Booking
data (slot, intake questions, email) is captured by TimeRex on their side;
Google Meet links are auto-generated through TimeRex's calendar integration.

Nothing is captured on this page itself, so no backend / webhook / API key
is required for the LP to function.

## Files changed for the subdomain switch

- `CNAME` — new, pins the custom domain
- `index.html` — added `<link rel="canonical">`, `<meta name="robots" noindex,nofollow>`, GA4 loader + event tracking
- `DEPLOYMENT.md` — this file
