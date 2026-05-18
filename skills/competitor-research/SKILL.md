---
name: competitor-research
description: >
  Use this skill for competitor intelligence and voice-of-customer research —
  competitor SEO / paid / social / KOL teardowns, battlecard generation,
  positioning comparison, ad creative analysis, review-mining, and Reddit /
  forum / community VoC research for copy, messaging, and product insight.
  Trigger on phrases like "competitor scan", "competitor research",
  "battlecard", "competitor ads", "competitor SEO", "Reddit research",
  "voice of customer", "VoC", "review mining", "customer language", or the
  `/competitor-scan`, `/reddit-research` commands.
metadata:
  version: "0.1.0"
---

# Competitor Research + VoC Skill

Two related research jobs in one skill: **competitor teardowns** (what competitors are doing) and **voice-of-customer research** (what real customers say). Both feed into positioning, content, and copy decisions.

> **Read first:** `marketing-framework/SKILL.md` for tool-priority and the data-first principle. **Details:**
> - `${CLAUDE_PLUGIN_ROOT}/skills/competitor-research/references/competitor-analysis-methodology.md` — competitor data collection + battlecard structure
> - `${CLAUDE_PLUGIN_ROOT}/skills/competitor-research/references/reddit-research-methodology.md` — Reddit / forum / community VoC research process

## When to Use This Skill

- Building or updating competitive battlecards
- Pre-launch competitor sweep for a new campaign or product
- Diagnosing a ranking / share loss against a specific competitor
- Mining customer language for blog / ad / landing-page copy
- Pain-point and decision-criteria research for new market entry
- Sentiment and review analysis

## Tool Priority

| Need | Primary | Notes |
|------|---------|-------|
| Competitor SEO (Authority Score, keywords, content, backlinks) | Semrush MCP | Never use Windsor for competitors |
| Competitor traffic estimate | Semrush MCP | — |
| Competitor Meta ads | Chrome → Meta Ad Library | Live + historical |
| Competitor Google ads | Chrome → Google Ads Transparency Center | — |
| Competitor social content | Bizkol MCP `call_scraper` with `<platform>_get_profile` / `<platform>_get_post` | Falls back to Chrome for platforms Bizkol doesn't cover |
| Competitor KOL partnerships | Bizkol MCP `search_kols` filtered by brand mentions; Chrome | — |
| Reviews + sentiment | Chrome → G2, Trustpilot, Amazon, App Store, Google reviews | — |
| Reddit VoC | Bizkol MCP `call_scraper` with reddit scraperIds (`reddit_search_posts`, `reddit_search_subreddits`, `reddit_get_post`, `reddit_get_subreddit`, `reddit_get_user`) | — |
| Forum / community VoC | Web search + Chrome | — |
| Press / PR / podcasts | Web search + Chrome | — |

**CRITICAL:** Windsor is owned-data only. Competitor data must come from Semrush / Chrome / Bizkol MCP / web search.

## Battlecard Structure

For each competitor, produce:

1. **Snapshot** — name, URL, founded, funding, est. employees, est. revenue, primary geo
2. **Positioning** — homepage hero claim, target ICP, primary differentiator
3. **Strengths** vs. our client (data-backed)
4. **Weaknesses** vs. our client (data-backed)
5. **Pricing + offer** — packaging, free tier, trial, money-back
6. **Content strategy** — blog cadence, top topics, top-performing pages (Semrush), social presence
7. **Ad creative** — top 3 active angles per channel (Meta Ad Library, GA Transparency)
8. **KOL partnerships** — visible creator partnerships (Bizkol search + manual scan)
9. **Win / loss patterns** (if user has rep input)
10. **Recommended counter-moves** — 2–3 specific, sized actions

See `references/competitor-analysis-methodology.md` for the full data-collection sequence.

## VoC Research (Reddit + reviews + communities)

Goal: surface the **language customers actually use** — pain points, outcomes desired, decision criteria, switching triggers, objections — for use in copy, ads, landing pages, and product positioning.

Output sections:

1. **Pain points** — ranked by frequency, with quote evidence
2. **Outcomes desired** — what "win" looks like, customer's words
3. **Decision criteria** — what customers compare on
4. **Switching triggers** — what makes them leave a competitor
5. **Objections** — common reasons not to buy
6. **Vocabulary table** — "what customers say" → "what marketers say" mapping
7. **Quote bank** — 20–30 verbatim quotes, attributed (source + date), ready for copy lift

See `references/reddit-research-methodology.md` for subreddit sourcing, query patterns, and quote-attribution standards.

## Prerequisite — Client Folder

`/competitor-scan` requires the client folder at `/clients/[client-name]/` (run `/new-client` first if missing). `/reddit-research` may run in topic-only mode (no client) and save to a session folder.

## Folder Output

```
/clients/[client-name]/
  /raw-data/
    /competitor/
      competitor-list-from-search-[YYYY-MM-DD].md   # seed candidates from /new-client Step 0
      competitor-list.md                            # confirmed seed list (curated)
      competitor-snapshot-[YYYY-MM-DD].md           # light scan from /new-client Step 4.2
      competitor-deep-dive-[competitor]-[YYYY-MM-DD].md  # per-competitor teardown — /competitor-scan only
    /paid/
      ad-library-data-[competitor]-[YYYY-MM-DD].md
    /voc/
      reddit-research-[topic]-[YYYY-MM-DD].md
      social-research-[topic]-[YYYY-MM-DD].md
      reviews-[platform]-[YYYY-MM-DD].md
  /audits/
    competitor-scan-[YYYY-MM-DD].md
    voc-research-[topic]-[YYYY-MM-DD].md
```

**Filename boundaries** — keep these distinct so onboarding doesn't oversell its output:
- `competitor-list-from-search-*` and `competitor-list.md` — seed lists (from `/new-client`).
- `competitor-snapshot-*` — light scan over the seed list (from `/new-client` Step 4.2). One file, not per-competitor.
- `competitor-deep-dive-[competitor]-*` — full battlecard-style teardown, one file per competitor. **Only `/competitor-scan` produces these.** Do not use this filename during onboarding.

## Audience Modes

- **Agency**: competitor list + battlecards per client; refresh on cadence (monthly recommended).
- **Merchant**: single competitor list for your brand; refresh quarterly + before each major launch.

## Related Skills / Commands

- **`marketing-framework`** — operating principles
- **`seo-geo`** — feeds competitor SEO data into audits
- **`paid-media`** — competitor ad creative analysis informs your own creative
- **`content-brand`** — VoC quote bank powers copy + headlines
- `/competitor-scan`, `/reddit-research`
