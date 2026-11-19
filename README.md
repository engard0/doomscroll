# doomscroll ⤵

Doomscroll without the apps: an agent skill that turns the chat into the feed. Hacker News, Reddit, X, Mastodon and localized news, ranked by engagement × recency, one line per item, paginated when you say "more". No keys, no signups — every source is a free public endpoint.

## Install

```bash
npx skills add engard0/doomscroll                                # any skills-CLI agent
npx skills add engard0/doomscroll -s doomscroll -a claude-code   # one agent only
```

Hermes Agent: copy `skills/doomscroll/` into `~/.hermes/skills/`.
No CLI? Copy `skills/doomscroll/` (SKILL.md + `references/`) into whatever folder your agent reads skills from.

## Use

Say `/doomscroll` — or "dame el feed", "what's happening", "scroll".

- The agent fetches all lanes in parallel and renders one line per item, in your language.
- "más" → next page. "solo memes" / "solo tech" → lane filter. From the third page on, the extras lane opens (GitHub trending, Lobste.rs, Product Hunt).
- Lanes that fail (Reddit 403s some IPs) are skipped with one line — the feed never stalls.

## Customize

Edit `skills/doomscroll/references/sources.md`: locales (Google News `hl/gl/ceid`), subreddits, lane weights. Optional Hermes-desktop infinite-scroll widget: `references/widget.md`.

## Honest limits

- Reddit blocks plain HTTP from many IPs; the skill degrades curl → real-browser fetch → skip.
- X lane needs an X search tool; absent = skipped.

MIT — see [LICENSE](LICENSE).
