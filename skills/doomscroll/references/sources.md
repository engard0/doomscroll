# Doomscroll sources

All free, no API key. Probe date: 2026-11-19 (HTTP 200 from a residential Windows IP unless noted).

| Lane | Endpoint | Notes / fallback |
|---|---|---|
| HN front page | `https://hn.algolia.com/api/v1/search?tags=front_page&hitsPerPage=20` | JSON. Empty → `https://hn.algolia.com/api/v1/search_by_date?tags=story&hitsPerPage=20`. Comment link: `https://news.ycombinator.com/item?id=<objectID>` |
| Reddit r/all | `https://www.reddit.com/r/all/hot.json?limit=25&raw_json=1` | Needs browser-ish UA; 403 from many IPs → real-browser fetch of the same URL, else skip. HTML instead of JSON = block page |
| Memes | `https://www.reddit.com/r/memes/hot.json?limit=15` (also `r/me_irl`) | Same pattern as r/all |
| Mastodon (social fallback) | `https://mastodon.social/api/v1/trends/statuses?limit=15` | No auth. Content is HTML — strip tags |
| News | `https://news.google.com/rss?hl=<lang>&gl=<CC>&ceid=<lang>:<CC>` | RSS, titles only. ES: `hl=es&gl=ES&ceid=ES:es` · EN-US: `hl=en-US&gl=US&ceid=en-US:us` |
| X | agent's X search tool, one query per trend | Absent tool → skip lane |
| GitHub trending (extras) | `https://github.com/trending` via `web_extract`, or curl + regex `href="/([\w.-]+/[\w.-]+)/stargazers"` | Verified pattern |
| Lobste.rs (extras) | `https://lobste.rs/hot.json` | Flaky (500s); best-effort |
| Product Hunt (extras) | `https://www.producthunt.com` via `web_extract` | Scrape, titles only |

## Fast paths (curl)

```bash
curl -s "https://hn.algolia.com/api/v1/search?tags=front_page&hitsPerPage=20"
curl -s -A "Mozilla/5.0" "https://www.reddit.com/r/all/hot.json?limit=25&raw_json=1"
curl -s "https://mastodon.social/api/v1/trends/statuses?limit=15"
curl -s "https://news.google.com/rss?hl=es&gl=ES&ceid=ES:es"
curl -s -A "Mozilla/5.0" "https://github.com/trending"
```

## Customization

- Locale: edit the Google News `hl`/`gl`/`ceid` triple to the user's language/country.
- Lanes: add/remove rows; any public subreddit works with the r/all pattern.
- Weights: per-lane cap lives in SKILL.md Procedure step 2 (default 6).
- Re-probe endpoints with the fast paths above after long gaps; Reddit posture changes often.
