---
name: kol-discovery
description: Discover relevant creators on Instagram, TikTok, YouTube, or X by topic, follower range, and engagement rate using the Bizkol MCP. Returns a ranked shortlist with handles and sample content. Discovery results are not persisted in Bizkol unless added to a campaign — only the final shortlist is exported locally.
---

# /kol-discovery $ARGUMENTS

Run an ad-hoc creator discovery sweep using the Bizkol MCP. Use when you need a shortlist for a topic, audience, or campaign brief — without spinning up a full campaign.

## Step 0: Prerequisite — Client Folder

Before running, the client folder must exist at `/clients/[client-name]/` with `client-brief.md`. If not:

> "No client folder found for [client-name]. Run `/new-client [client-name]` first to create the brief and folder structure, then re-run this command."

If `$ARGUMENTS` is purely a topic (no client context), you may run in **topic-only mode** — save outputs under `/sessions/[topic-slug]-[YYYY-MM-DD]/` and skip the brief lookup. Warn the user that brand-fit scoring will be weaker without a client brief.

## Workflow

### Step 1 — Confirm the discovery brief

If a client brief exists, pull voice / audience / brand context. Then confirm with the user:
- **Topic / vertical** (e.g., "skincare for sensitive skin", "EV reviews", "small-batch coffee")
- **Platforms** (Instagram, TikTok, YouTube, X — pick 1–3)
- **Follower band** (nano <10k, micro 10k–100k, mid 100k–1M, macro 1M+)
- **Geography or language** (if relevant)
- **Engagement floor** (default 2% for most verticals; 1% for macro)
- **Budget ceiling** (per-post or per-campaign — optional)

### Step 2 — Search Bizkol's KOL DB first

For each platform, run:

```
Tool: search_kols
Parameters:
  query: "<topic keywords>"
  platform: "<instagram|tiktok|youtube|x>"
  minFollowers: <number>
  maxFollowers: <number>
  minEngagementRate: 0.02
  maxPrice: <number>
  pageSize: 25
```

Save the raw results per platform to `/clients/[client-name]/raw-data/kol/discovery-[topic-slug]-[platform]-[YYYY-MM-DD].md` **before** any filtering.

### Step 3 — Augment with live platform discovery

For creators not yet in the Bizkol DB:

```
Tool: call_scraper
Parameters:
  scraperId: "<platform>_search_kols"  # instagram_search_kols | tiktok_search_kols | youtube_search_kols | x_search_kols
  input:
    query: "<topic keyword>"
    count: 20
```

Note: on Instagram (`instagram_search_kols`), `query` is treated as a hashtag.

Save these live results to `/clients/[client-name]/raw-data/kol/discovery-live-[topic-slug]-[platform]-[YYYY-MM-DD].md`.

### Step 4 — Enrich the top candidates

For the most promising 10–15 (mix of Bizkol DB + live):

- Bizkol DB candidates: `get_kol`, `get_kol_performance`, `get_kol_posts`
- Live-only candidates: `call_scraper` with `<platform>_get_profile`, then `call_scraper` with `<platform>_get_post` on a recent post

### Step 5 — Score and rank

Score each candidate on a 1–5 scale across:

| Criterion | Weight | What "5" looks like |
|-----------|--------|---------------------|
| Audience fit | 30% | Engaged audience demographics match the brief tightly |
| Engagement quality | 25% | High ER with substantive comments, not just emoji reacts |
| Content fit | 20% | Recent posts in the same vertical, native style |
| Price/value | 15% | Strong reach per dollar (use `quotes` from Bizkol when available) |
| Brand safety | 10% | No recent controversy, clean tone, on-brand |

### Step 6 — Output the shortlist

Save to `/clients/[client-name]/audits/kol-discovery-[topic-slug]-[YYYY-MM-DD].md`:

```markdown
# KOL Discovery — [topic]
**Date:** [YYYY-MM-DD] | **Platforms:** [list] | **Follower band:** [band]

## Top picks

| Rank | Handle | Platform | Followers | ER | Score | In Bizkol DB? | Notes |
|------|--------|----------|-----------|----|-------|---------------|-------|
| 1 | @handle | TikTok | 245k | 4.8% | 4.6 | Yes (kolId: ...) | Strong audience fit |
| 2 | ... | ... | ... | ... | ... | No — would need import | ... |

## Honorable mentions
- ...

## Next steps
- [ ] Move top picks into a campaign with `/kol-campaign`
- [ ] Import live-only handles via `import_kols_to_campaign_by_username`
```

This is **not** a Bizkol persistence step — discovery shortlists become Bizkol state only when added to a campaign via `/kol-campaign`.

## Graceful degradation

- **If Bizkol MCP isn't connected:** Skip Steps 2 and 4 (Bizkol DB lookups). Run Step 3 (live discovery) only and note in the output that Bizkol enrichment is unavailable. Recommend connecting Bizkol for richer scoring data.
- **If a platform returns no results:** Note "No data found" with the search parameters used and continue with the other platforms.
