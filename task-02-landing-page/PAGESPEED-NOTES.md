# Getting the PageSpeed Insights (Mobile) screenshot

`index.html` is built to score well on Mobile PageSpeed on purpose:

- No external requests at all — no web fonts, no images, no CDN scripts. Everything is inline
  CSS/JS in the one file, and text uses the system font stack (`-apple-system, Segoe UI, Roboto...`)
  instead of a Google Fonts request.
- All icons are inline SVG, not image files.
- No render-blocking `<link>` or `<script src>` tags.
- Layout uses `clamp()` and CSS Grid instead of a JS layout library.
- `IntersectionObserver` instead of a scroll-event listener for the mobile sticky CTA, so there's
  no continuous main-thread work.

PageSpeed Insights needs a **public URL** — it won't accept a local file. Steps:

1. Push this repo to GitHub.
2. Enable **GitHub Pages** for the repo (Settings → Pages → deploy from the `main` branch,
   root or `/task-02-landing-page` folder — whichever you point it at, the published URL will
   serve `index.html`).
3. Go to https://pagespeed.web.dev, paste the published URL, and run the **Mobile** report.
4. Screenshot the score card (the circular score plus the Core Web Vitals section) and save it
   in this folder, e.g. `pagespeed-mobile-score.png`.
5. Reference the screenshot from the GitHub repo README or this file before you submit.

If GitHub Pages isn't available for any reason, any static host works the same way (Netlify,
Vercel, Cloudflare Pages) — the page has zero build step and zero dependencies, so a raw static
deploy is all it needs.
