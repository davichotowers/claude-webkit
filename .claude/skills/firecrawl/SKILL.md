---
name: firecrawl
description: Analyze reference websites with the Firecrawl CLI — scrape a single page, map a site's URLs, crawl multiple pages, and extract design/branding (colors, fonts, logo) or structured data. Use when the user shares reference URLs, wants to study a competitor's site, or needs real content/branding pulled from a live website.
license: MIT
---

# Firecrawl — Reference Site Analysis

Firecrawl turns websites into clean markdown, links, screenshots, or structured JSON.
In this project it is used during **Phase 1 (Discovery)** to analyze reference URLs,
and in **Phase 2 (Design System)** to extract branding from sites the user likes.

## Authentication

The API key is injected automatically by the environment (`FIRECRAWL_API_KEY` or proxy).
**Never** ask the user for a key, never pass `-k/--api-key`, and never print or write keys to files.

In cloud sessions the proxy adds the real key to every request to `api.firecrawl.dev`,
but the CLI refuses to run ("Not authenticated") unless it sees *some* key locally.
Run this once per shell before any `firecrawl` command — it sets a non-secret
placeholder only when no key exists, and the proxy swaps in the real one:

```bash
export FIRECRAWL_API_KEY="${FIRECRAWL_API_KEY:-fc-proxy-injected}"
```

After that, `firecrawl --status` shows "Authenticated via FIRECRAWL_API_KEY".

## Which command to use

| Goal | Command | Cost |
|------|---------|------|
| Read one page (copy, headings, structure) | `scrape` | 1 page |
| Get design tokens: colors, fonts, logo | `scrape --format branding` | 1 page |
| See all pages a site has | `map` | cheap |
| Read several pages of a site | `crawl` | 1 per page |
| Pull specific fields as JSON | `scrape --schema` or `-Q` | 1 page |

Rule of thumb: **map first, then scrape the few pages that matter.** Only crawl when you really need many pages, and always set `--limit`.

## scrape — one page

```bash
# Clean markdown of the main content
firecrawl scrape https://example.com --only-main-content

# Several formats at once (outputs JSON)
firecrawl scrape https://example.com --format markdown,links,images --pretty

# Screenshot of the full page (for visual reference)
firecrawl scrape https://example.com --full-page-screenshot --json -o .firecrawl/ref.json

# Wait for JS-heavy sites to render
firecrawl scrape https://example.com --wait-for 3000

# Quick summary or a direct question about the page
firecrawl scrape https://example.com --summary
firecrawl scrape https://example.com -Q "What is the main call to action?"
```

## Design & branding extraction

Use the `branding` format to get a site's visual identity (color palette, typography, logo, UI style):

```bash
firecrawl scrape https://reference-site.com --format branding --pretty -o .firecrawl/branding.json
```

Combine with a screenshot for full context:

```bash
firecrawl scrape https://reference-site.com --format branding,screenshot --pretty
```

Structured extraction with a JSON schema:

```bash
firecrawl scrape https://reference-site.com --schema '{
  "type": "object",
  "properties": {
    "headline": {"type": "string"},
    "sections": {"type": "array", "items": {"type": "string"}},
    "cta_text": {"type": "string"},
    "primary_color": {"type": "string"},
    "fonts": {"type": "array", "items": {"type": "string"}}
  }
}' --pretty
```

How to use the results:
- Take **inspiration**, don't copy: adapt the palette/fonts to the design rules in `docs/design-guide.md` (no Inter/Roboto/Arial, no AI palettes).
- Note section order and patterns, then map them to an archetype in `docs/landing-page-patterns.md`.
- Never copy text verbatim into the user's page — rewrite and run it through `humanizer`.

## map — list a site's URLs

```bash
firecrawl map https://example.com --limit 50
firecrawl map https://example.com --search pricing       # filter URLs by keyword
firecrawl map https://example.com --json --pretty -o .firecrawl/map.json
```

Use it to find the pages worth scraping (home, pricing, about, services).

## crawl — many pages

```bash
# Start and wait, with a hard page limit
firecrawl crawl https://example.com --limit 10 --max-depth 2 --wait --progress -o .firecrawl/crawl.json

# Only certain sections
firecrawl crawl https://example.com --include-paths /blog,/services --limit 20 --wait

# Check or cancel a running job
firecrawl crawl <job-id> --status
firecrawl crawl <job-id> --cancel
```

Always pass `--limit` to control credits.

## Output & housekeeping

- Save large outputs under `.firecrawl/` (not inside `site/`).
- Don't commit scraped content or screenshots.
- Check remaining credits: `firecrawl credits`.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| "Not authenticated" / auth error | Run the `export FIRECRAWL_API_KEY=...` line from Authentication. If it still fails, the key isn't configured in the environment — tell the user, don't ask for it |
| Empty/partial content | Add `--wait-for 3000` |
| Too much noise | Add `--only-main-content` or `--exclude-tags nav,footer` |
| Blocked site | Try `--proxy auto` |
