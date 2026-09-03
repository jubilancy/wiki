# Web Scraping with Claude: What's Actually Possible

> A practical guide to the tools Claude has for fetching and searching web content — what they can and can't do, and how to use them well.

## Contents
- [The Two Core Tools](#the-two-core-tools)
- [What Claude Can Do](#what-claude-can-do)
- [What Claude Cannot Do](#what-claude-cannot-do)
- [Practical Patterns](#practical-patterns)
- [Copyright & Reproduction Limits](#copyright--reproduction-limits)
- [When to Use the Code Environment Instead](#when-to-use-the-code-environment-instead)
- [Good Prompts vs. Bad Prompts](#good-prompts-vs-bad-prompts)

## The Two Core Tools

Claude (in this chat interface) has two web-related tools:

- **Web search** — runs a query against a search engine and returns the top results (titles, snippets, URLs). Good for discovery: finding pages, checking current facts, locating sources.
- **Web fetch** — retrieves the actual content of a specific URL (one that's already appeared in the conversation, from search results, or that you pasted). Good for extraction: pulling the real text of a page once you know where it is.

Typical flow: search to find candidate pages → fetch the promising ones → read/summarize/extract from the fetched content.

## What Claude Can Do

- Fetch a public, non-authenticated webpage and read its rendered text content
- Follow a chain of a few pages (e.g., a search result → the article it points to)
- Pull structured-looking data out of page text (headings, lists, tables) and reformat it
- Extract metadata like page titles and meta descriptions when present in the fetched content
- Compare content across a handful of pages (e.g., "how did these 3 outlets cover this story")
- Do this dozens of times in one conversation for a research-style task — scaling up tool calls for broader questions

## What Claude Cannot Do

- **No JavaScript rendering.** Fetch retrieves the page's static/server-rendered content. Sites that build content entirely client-side (heavy React/Vue SPAs with no server-rendered fallback) may return mostly empty shells.
- **No login/paywall access.** Anything behind authentication — private docs, logged-in dashboards, most paywalled news sites — is off-limits.
- **No arbitrary URL guessing.** Claude can only fetch a URL that has already appeared in the conversation — one you provided, or one that came back from a prior search/fetch. It can't construct a plausible-looking URL from memory or by editing a path and fetch that.
- **No bulk/automated crawling.** There's no "crawl this whole site" or "fetch these 3,000 URLs" primitive. Each fetch is a discrete, visible tool call — this makes it a poor fit for large-scale scraping jobs (hundreds to thousands of pages).
- **No CAPTCHA solving, no bypassing bot-detection, no scraping tools that require browser automation** (clicking, scrolling, filling forms) — Claude's fetch is a simple content retrieval, not a headless browser.
- **No reproduction of full copyrighted text.** Even when a page is successfully fetched, Claude paraphrases rather than reproducing large verbatim chunks (see [Copyright & Reproduction Limits](#copyright--reproduction-limits)).
- **No real-time/streaming data.** Each fetch is a one-time snapshot of the page at request time, not a live feed.

## Practical Patterns

### 1. Verify or enrich a small list of URLs
Good fit. If you have ~10–50 URLs and want to check they're live, pull a cleaner title, or grab a fuller description than what you have, Claude can fetch each one and report back. This is naturally throttled by the conversation itself — each fetch is a visible, individual step.

### 2. Research a topic across multiple sources
Good fit. Search for the topic, fetch the most relevant/authoritative results, synthesize a summary with citations. This is what the search+fetch combination is built for.

### 3. Scrape a single site's structure (e.g., a sitemap or listing page)
Partial fit. Claude can fetch a listing/index page and extract the links and text visible on that page. It cannot then recursively crawl every link found unless you're willing to have Claude fetch each one as a separate, explicit step (which gets impractical past a few dozen).

### 4. Bulk-scrape hundreds or thousands of pages
Poor fit for the fetch tool directly. This is better handled by:
- Exporting from the source tool itself (e.g., Raindrop.io CSV export, browser bookmark export) rather than scraping
- A dedicated scraping script/library run in Claude's code execution environment (see below) — but note the code environment's network access is also allowlisted to specific domains, not the open internet

### 5. Get data that requires a login
Not possible via fetch. Workarounds: export the data yourself from the authenticated source and upload the file, or use a connected tool/integration (e.g., a Google Drive or Notion connector) if one exists for that service.

## Copyright & Reproduction Limits

Even where fetching works technically, Claude follows strict limits on reproducing what it finds, regardless of the request:

- Quotes are capped at roughly 15 words, and only one direct quote per source
- No reproduction of song lyrics, poems, or full article passages, however short
- Summaries are written in Claude's own words, not close paraphrases that mirror the original's structure or phrasing
- For a research task spanning many sources, expect paraphrase-heavy output with light, attributed sourcing — not a document that reconstructs the original pages

This means "scrape this article and give me the full text" isn't something Claude will do verbatim, even if the fetch itself succeeds. Claude can summarize it, extract specific facts, or point to the source.

## When to Use the Code Environment Instead

For structured data work — not novel scraping, but *processing* data you already have — Claude's code execution environment (Python/bash) is the better tool:

- Parsing and reformatting CSV/JSON exports (e.g., Raindrop.io, Obsidian, browser bookmarks)
- Deduplicating and normalizing large URL lists
- Regex-based extraction and diffing (e.g., "which URLs are in file A but not file B")
- Generating output files (markdown, CSV, etc.) at scale

The code environment's network access is restricted to an allowlist of package/dependency domains (PyPI, npm, GitHub, etc.) — it is **not** a general-purpose internet connection, so it can't be used to fetch arbitrary websites either. Actual webpage retrieval still goes through the fetch tool, one URL at a time.

## Good Prompts vs. Bad Prompts

| Instead of... | Try... |
|---|---|
| "Scrape all 3,000 pages on this site" | "Fetch these 20 specific URLs and pull their titles and descriptions" |
| "Give me the full text of this paywalled article" | "Search for other coverage of this story I can access, or summarize what's in the preview text" |
| "Scrape this login-only dashboard" | "Here's a CSV I exported from that dashboard — process this instead" |
| "Crawl this site and build me a sitemap" | "Fetch the site's sitemap.xml or listing page directly, if one exists" |
| "Reproduce this article word for word" | "Summarize this article's key points" |

---

**Bottom line:** Claude is well suited to *targeted* fetching — a known list of URLs, a research question needing a handful of sources — and to *processing* data you've already exported. It is not a general-purpose crawler or scraper for large, authenticated, or JavaScript-heavy sites.
