## PLUTUS — Your AI Stock Analysis Companion

A futuristic, dark-mode dashboard that turns Yahoo Finance data into plain-English, gamified insights. **Facts only — no buy/sell/hold calls.** Fully responsive for desktop, tablet and mobile.

### Dashboard sections (in order)
- **About the Company** — business summary (justified, with Read more), sector, industry, HQ, website
- **Health Scan** — 0–100 score ring, six power stats
- **Key Numbers** — 12 KPI cards in 3 clickable categories of 4, full width (Growth & Profit, Value & Returns, Safety & Cash) with tap/hover explanations
- **Trajectory** — price (1Y/5Y), P/E history and Yearly/Quarterly business charts
- **Shareholding Pattern** — a crowd of 100 people (1 person = 1% of shares; a circle on phones) coloured by holder type, the change since the previous filing, and the biggest institutional and mutual-fund holders (Yahoo Finance)
- **Peer Arena** — companies from the **same industry**, medals, head-to-head chart and comparison table
- **Strengths & Weaknesses** — plain-English facts, including "VS INDUSTRY" items

Search favours Indian listings (NSE, then BSE), marked with an IN tag, and matches on the **company name** (not the ticker).

### What changed in v4
1. Top bar (logo + search) is now fixed to the top of the screen on the stock page.
2. Search ranks results by company name only; ticker is shown as secondary text.
3. Achievements removed from Health Scan.
4. Peers are real industry peers (verified against Yahoo's industry label), chosen closest in size.
5. Company logo beside the name in search suggestions, the stock header and the peer table.
6. Industry averages are available for every metric (strip, power stats, KPI cards, P/E and margin charts, peer table) — switched on with the **Compare with industry average** switch.

### Structure
index.html
manifest.webmanifest  ← PWA manifest (name, icons, colours)
sw.js                 ← service worker (installability + offline shell)
icons/                ← app icons (192, 512, maskable, Apple touch)
css/style.css
css/news.css
css/pwa.css       ← "Install app" modal
js/config.js      ← WORKER_URL, LOGO_SOURCES
js/analysis.js    ← metrics, scores, peer ranks, strengths/weaknesses
js/app.js         ← UI, search, logos, industry peers, shareholding, routing, charts
js/news.js        ← "Latest In News" sidebar (max 10: 5 company, then 5 industry)
js/pwa.js         ← service-worker registration + one-time install modal
worker/worker.js  ← Cloudflare Worker (deploy on Cloudflare, not GitHub)

### Compare with industry average
- Industry averages are **off by default**, so the page stays clean. The switch is in the fixed **top bar** (bars-with-average-line icon), so it never takes page space.
- Turning it on adds comparisons **in place** — nothing is inserted, so the layout never moves: a small ▲/▼ badge on the headline numbers (hover or tap it for the average), the average + difference in the sub-line of each Key Number card, a marker on each Power Stat meter and on the score ring, and dotted lines on the P/E and Profit Margins charts.
- Green / red = better / worse than the industry (where that is meaningful), grey = neutral or in line. The Peer Arena's own average bar and row are always shown.
- The choice is remembered in the browser (`plutus.compare.v1`). If peers can't be found the switch shows N/A. A one-time legend appears the first time it is turned on (`plutus.compareHint.v1`).

### Install as an app (PWA)
- The site is a Progressive Web App. On a visitor's first visit a modal offers to install it: one-tap **Install app** on Chrome/Edge/Android; the matching "menu → Install / Add to Home Screen" steps on phones whose browser has no install button (iPhone/iPad, Firefox, in-app browsers).
- It is shown only once per browser (localStorage key `plutus.installPrompt.v1`) and never inside the installed app. **To see it again, open the site with `?install=1`** (or clear that key).
- One-tap install and offline support need **HTTPS** (GitHub Pages is fine) or `localhost`.
- After you change files, bump `CACHE_VERSION` in `sw.js` and the `?v=` number in `index.html`.
- Live data (Yahoo via the Worker, Bing news, logos) is never cached by the service worker.

### Deploy
- **Worker (re-deploy — it changed in v4):** Cloudflare → Workers → your worker → Edit code → paste worker/worker.js → Deploy. v4 adds POST support for Yahoo's screener (used to list companies in the same industry). Without the update the site still works, but peer discovery falls back to less complete sources.
- Put the Worker URL in js/config.js (WORKER_URL).
- **GitHub Pages:** upload the folder contents (with index.html at the top level) → Settings → Pages → Deploy from branch main / root.
- _(Recommended)_ Set ALLOWED_ORIGINS = ['https://<user>.github.io'] in the Worker and redeploy.
Run locally: python -m http.server 8080.

### Notes
- Industry average = simple average of up to 6 listed peers in the same industry (closest in size), computed the same way as the company's own figures. EPS is per-share and isn't comparable across companies, so it shows n/a.
- Logos come from the company's website domain via public favicon services (see LOGO_SOURCES in config.js); a dedicated logo API can be added there. If none is found, an initials badge is shown.
- Yahoo's free data covers about 4–5 annual periods and up to 8 quarters. Many Indian companies publish cash flow and balance sheets only half-yearly or yearly, so those quarterly charts may say "No quarterly data published".
- Yahoo's endpoints are unofficial and can change without notice.
- Educational use only. Not investment advice.
