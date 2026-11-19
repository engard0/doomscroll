---
name: doomscroll
description: "Fresh chat feed: HN, Reddit, X, news. Paginates on demand."
version: 0.1.0
author: Engard (engard0), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [doomscroll, feed, news, reddit, hacker-news, trends]
    homepage: https://github.com/engard0/doomscroll
    related_skills: []
---

# Doomscroll Skill

Turns the chat into the feed: pulls what is fresh right now across Hacker News, Reddit, X and localized news, ranks it by engagement × recency, and renders one compact line per item. No apps opened, no opaque algorithm — the agent is the algorithm. Needs only web fetch/search tools (`web_extract`, `web_search`, `terminal` curl); the X lane activates only if the agent has an X search tool, and any failing lane is skipped, never fatal.

## When to Use

- User says `/doomscroll`, "doomscroll", "scroll", "dame el feed", "qué está pasando", "what's new/trending today" without naming a specific topic.
- User wants casual browsing of several sources at once, or asks for "more" after a feed was rendered.
- Don't use for: focused research on one topic (`web_search` directly), per-stock market checks, or scheduled monitoring (cron digest instead).

## Prerequisites

- A web fetch path: `web_extract` on JSON/RSS endpoints or `terminal` with `curl`.
- Current date: run `date -Iseconds` first; discard items older than 48 h unless the lane is thin.
- Optional: an X search tool (e.g. `x_search`) — absent tool = skip the X lane.
- Source table with exact endpoints, fallbacks and customization: `references/sources.md`.

## Procedure

1. **Fetch lanes in parallel** — one batched round of tool calls, never sequential. Default lanes (exact endpoints in `references/sources.md`):
   - Tech: Hacker News front page (Algolia JSON, no key).
   - Social: Reddit r/all hot JSON (fast path curl, browser fallback), Mastodon trending statuses as the no-auth fallback.
   - News: Google News RSS for the user's locale (`hl`/`gl`/`ceid` from their language/country).
   - X: one search for current trends/topics, only if a search tool exists.
   Completion criterion: for each lane, titles + URLs + engagement captured, or a one-line skip logged.
2. **Rank**: dedupe by URL; score = engagement (points/ups/comments) × recency decay (halve per 12 h); cap 6 per lane, 12–18 items total.
3. **Render** in the user's language, grouped by lane (Tech / Social / News / X):
   `- [Title](url) — source · one-line hook · ▲points 💬comments`
   Titles stay in their original language; the hook is the gloss: what it is + why it is worth ten seconds, in the user's language.
4. **Paginate**: close with "¿más? / more?". On "más/next/more", fetch the next slice (deeper page, wider window) with no repeated URLs. From the third page on, open the extras lane (GitHub trending, Lobste.rs, Product Hunt via `web_extract`).
5. **Filter**: "solo tech / solo memes / solo X" → fetch only that lane (memes = `r/memes`, `r/me_irl` through the Reddit pattern).

## Pitfalls

- Reddit 403s curl from many IPs (serves an HTML block page): on 403 or HTML-instead-of-JSON, retry once through the agent's real-browser tool on `https://www.reddit.com/r/all/hot.json?limit=25&raw_json=1` (browser traffic passes); no browser → skip the lane with one line. Never retry more than twice.
- Google News RSS links are redirect wrappers (`news.google.com/rss/articles/...`): render as-is; they resolve in the browser.
- The hook is ONE line per item. Do not summarize everything and do not auto-open paywalled links.
- HN Algolia `front_page` empty → `search_by_date?tags=story` fallback (in `references/sources.md`).
- A lane failing must never stall the whole feed: HN + News alone are enough for page one.

## Verification

- Page one shows ≥8 items across ≥2 lanes, each a working `[title](url)` with engagement numbers and a one-line hook in the user's language.
- "más" returns a batch with zero repeated URLs (dedupe by URL).
- Any failed lane is disclosed in one line and the feed still renders.

## Optional widget (Hermes desktop)

For the real infinite-scroll experience, write `doomscroll.html` (self-fetching HN Algolia, theme-aware) and open it with `::preview{file="doomscroll.html"}`. Full recipe: `references/widget.md`.
