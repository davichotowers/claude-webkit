---
name: lighthouse
description: Audit any URL with Google Lighthouse (npx lighthouse) using the pre-installed headless Chromium. Returns the 4 scores — Performance, Accessibility, Best Practices, SEO — plus the top issues. Use in Phase 5 (QA) on the local dev/prod server, before deploying, after deploying to check the live URL, or when the user asks "how fast is my site", "Lighthouse score", "PageSpeed", "Core Web Vitals", or wants to compare their site against reference/competitor sites.
---

# Lighthouse Audit

Runs Google Lighthouse from the command line against a URL and saves JSON + HTML reports to `.lighthouse/` (git-ignored).

## When to use

| Moment | URL to audit |
|--------|--------------|
| Phase 5 (QA), after the page is built | `http://localhost:3000` — prefer a production build (`npm run build && npm start`); `npm run dev` scores Performance much lower |
| Before Phase 6 (deploy) | Same, as the final gate |
| After deploy | The live preview URL |
| Discovery / reference analysis | The user's reference or competitor URLs, to set a target |

Do not use it to judge visual design — use `playwright-cli` screenshots for that.

## Browser setup

Uses the same Chromium as `.mcp.json` (Playwright MCP): `/opt/pw-browsers/chromium`, headless, no sandbox, ignoring certificate errors (needed behind the container's HTTPS proxy). Lighthouse reads the browser path from `CHROME_PATH`.

## Run one audit

```bash
mkdir -p .lighthouse
CHROME_PATH=/opt/pw-browsers/chromium npx -y lighthouse@latest "https://example.com" \
  --chrome-flags="--headless=new --no-sandbox --ignore-certificate-errors" \
  --only-categories=performance,accessibility,best-practices,seo \
  --output=json --output=html \
  --output-path=.lighthouse/example \
  --quiet
```

Produces `.lighthouse/example.report.json` and `.lighthouse/example.report.html` (with several `--output` formats Lighthouse appends `.report.<ext>`; with a single format it writes to `--output-path` exactly).

Default is **mobile** emulation (what Google uses for ranking). For desktop add `--preset=desktop`.

Print the 4 scores:

```bash
node -e 'const r=require("./.lighthouse/example.report.json");
if(r.runtimeError)console.log("ERROR:",r.runtimeError.code);
for(const c of Object.values(r.categories))console.log(c.title.padEnd(15),Math.round(c.score*100))'
```

List the failing audits worth fixing (score < 0.9):

```bash
node -e 'const r=require("./.lighthouse/example.report.json");
for(const a of Object.values(r.audits))if(a.score!==null&&a.score<0.9&&a.scoreDisplayMode!=="informative")console.log(Math.round(a.score*100),a.id,"-",a.title,a.displayValue||"")'
```

## Compare my site vs reference sites

Run the same settings (same device preset) on every URL, then print a table:

```bash
mkdir -p .lighthouse
for url in "http://localhost:3000" "https://reference-one.com" "https://reference-two.com"; do
  name=$(echo "$url" | sed -E 's#https?://##; s#[^a-zA-Z0-9]+#-#g; s#-$##')
  CHROME_PATH=/opt/pw-browsers/chromium npx -y lighthouse@latest "$url" \
    --chrome-flags="--headless=new --no-sandbox --ignore-certificate-errors" \
    --only-categories=performance,accessibility,best-practices,seo \
    --output=json --output-path=".lighthouse/$name.report.json" --quiet
done
node -e 'const fs=require("fs");
console.log("Site".padEnd(32),"Perf","A11y","BP ","SEO","LCP","CLS");
for(const f of fs.readdirSync(".lighthouse").filter(f=>f.endsWith(".report.json"))){
  const r=JSON.parse(fs.readFileSync(".lighthouse/"+f));const s=k=>String(Math.round((r.categories[k]?.score??0)*100)).padEnd(4);
  console.log(f.replace(".report.json","").padEnd(32),s("performance"),s("accessibility"),s("best-practices"),s("seo"),
    r.audits["largest-contentful-paint"].displayValue,r.audits["cumulative-layout-shift"].displayValue)}'
```

Tips for a fair comparison:
- Scores vary ±5 between runs (network, CPU). For a decision, run each URL 2–3 times and use the median.
- Compare like with like: production build vs live sites, same preset (mobile vs desktop).
- Beating references on Accessibility and SEO is always achievable; on Performance, aim to be equal or better.

## How to read the 4 scores

Each is 0–100. **90–100 green (good), 50–89 orange (needs work), 0–49 red (poor).**

| Score | What it measures | Typical fixes in this project |
|-------|------------------|-------------------------------|
| **Performance** | Load speed on a throttled mobile. Weighted from Core Web Vitals: LCP (largest element visible, target < 2.5 s), TBT (main-thread blocking, < 200 ms), CLS (layout jumps, < 0.1), FCP and Speed Index. | `next/image` with `priority` on the hero image, `next/font` with `display: "swap"`, fewer `"use client"` components, lazy-load below-the-fold motion. See `docs/performance-checklist.md`. |
| **Accessibility** | Automated WCAG checks: contrast, alt text, labels, heading order, ARIA, link names. 100 does not mean fully accessible — only automatable checks. | Fix contrast, add `alt`, visible `<label>`s, `aria-label` on icon buttons. See `docs/accessibility-checklist.md`. |
| **Best Practices** | Security and modern-web hygiene: HTTPS, no console errors, correct image aspect ratios, no deprecated APIs, CSP/headers. | Remove console errors, fix image sizes, avoid third-party scripts with known issues. |
| **SEO** | Basic crawlability: `<title>`, meta description, `lang`, valid links with descriptive text, crawlable, mobile viewport, `robots.txt`. | Set `metadata` in `layout.tsx`, descriptive link text, `robots.ts`/`sitemap.ts`. For deeper SEO use the `seo-audit` skill. |

When reporting to the user, give the 4 scores first, then at most 3 concrete fixes in plain language. Then ASK the user before applying any change to their website — never edit site files without their explicit approval.

## Troubleshooting

- `CHROME_PATH` / "No Chrome installations found" → check `ls -l /opt/pw-browsers/chromium`.
- `runtimeError` `ERRORED_DOCUMENT_REQUEST` or `FAILED_DOCUMENT_REQUEST` → the URL is not reachable; for localhost make sure the server is running.
- Certificate / `ERR_CERT` errors → confirm `--ignore-certificate-errors` is inside `--chrome-flags`.
- Chrome crashes on start → confirm `--no-sandbox` and `--headless=new` are inside `--chrome-flags`.
- `NO_FCP` → page rendered nothing visible; check the page in `playwright-cli`.
