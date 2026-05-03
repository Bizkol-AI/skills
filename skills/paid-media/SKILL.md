---
name: paid-media
description: >
  Use this skill for paid advertising work across Google Ads, Meta Ads,
  YouTube Ads, LinkedIn Ads, TikTok Ads, and Google Merchant — campaign
  strategy, audience targeting, budget pacing, creative analysis, ROAS / CPA
  / CTR review, and competitor ad inspection. Trigger on phrases like "paid
  ads", "ad campaign", "ROAS", "CPA", "ad creative", "Meta ads", "Google ads",
  "ad library", "boost", "budget pacing", or the paid-media sections of
  campaign commands.
metadata:
  version: "0.1.0"
---

# Paid Media Skill

Strategy, execution review, and creative analysis for paid advertising. Covers the client's own paid performance (via Windsor MCP) and competitive ad intelligence (via Chrome + ad libraries).

> **Read first:** `marketing-framework/SKILL.md` for tool-priority rules. **Detail:** `references/paid-media-framework.md` for budget allocation, channel strategy, and creative analysis frameworks.

## When to Use This Skill

- Reviewing paid performance for a campaign or ad-hoc audit
- Building a campaign plan that includes a paid component
- Analyzing competitor ads (Meta Ad Library, Google Ads Transparency Center)
- Creative review and creative-fatigue diagnosis
- Budget pacing checks and reallocation recommendations

## Prerequisite — Client Folder

Before any paid-media command runs, the client folder must exist at `/clients/[client-name]/` with `client-brief.md` populated (including Windsor account IDs). If missing, instruct the user to run `/new-client [name]` first.

## Tool Priority

| Need | Primary | Fallback if not connected |
|------|---------|---------------------------|
| Client's Google Ads performance | Windsor MCP `get_data` (`google_ads`) | Chrome → `ads.google.com` — capture spend / clicks / impressions / conversions per campaign |
| Client's Meta Ads performance | Windsor MCP `get_data` (`facebook`) | Chrome → `adsmanager.facebook.com` |
| Client's GA4 conversion data | Windsor MCP `get_data` (`googleanalytics4`) | Chrome → `analytics.google.com` |
| Client's GSC search data | Windsor MCP `get_data` (`searchconsole`) | Chrome → `search.google.com/search-console` |
| Client's Google Business Profile | Windsor MCP `get_data` (`google_my_business`) | Chrome → `business.google.com` |
| Client's Google Merchant data | Windsor MCP `get_data` (`google_merchant`) | Chrome → `merchants.google.com` |
| Commerce / sales context | Shopify MCP (see `shopify-commerce` skill) | Chrome → `*.myshopify.com/admin` |
| Competitor Meta ads | Chrome → Meta Ad Library | — |
| Competitor Google ads | Chrome → Google Ads Transparency Center | — |
| Creative inspiration / scraping | Chrome + competitor sites | — |
| Creative generation | Canva MCP | — |

**CRITICAL:** Windsor is for the **client's own** paid data. Competitor ads come from Chrome — never Windsor. **If Windsor is not connected**, fall back to Chrome on the respective platform UI for the same data shape — never block the review.

## Standard Output Sections

Per channel, every paid review produces:

1. **Headline metrics** — spend, impressions, clicks, CTR, CPC, conversions, CPA, ROAS — with WoW or MoM delta.
2. **Pacing check** — budget burned vs. budget planned for the period.
3. **Creative performance** — top creatives by ROAS / CTR; creative-fatigue signals (CTR decay > 20% over 7 days).
4. **Audience performance** — best / worst performing audiences or segments.
5. **Anomalies** — sudden CPM jumps, conversion drops, disapproval issues.
6. **Recommendations** — one or two actions per channel, sized by expected impact.

See **`references/paid-media-framework.md`** for budget split heuristics, creative-fatigue thresholds, audience-testing structure, and per-platform diagnostic flows.

## Cross-Channel Budget Heuristics

Default starting splits (tune by client / industry):

- **Brand-aware DTC ecommerce**: 50% Meta, 30% Google (search + shopping), 10% YouTube, 10% test budget
- **B2B SaaS**: 50% Google search, 25% LinkedIn, 15% Meta retargeting, 10% test budget
- **Local services**: 60% Google search + GBP, 25% Meta local awareness, 15% YouTube / display
- **Marketplace product launch**: 40% Meta prospecting, 30% Google shopping, 20% TikTok / Reels creators, 10% retargeting

These are starting points — the framework reference has full diagnostic flows for adjusting.

## Folder Output

```
/clients/[client-name]/
  /raw-data/
    /paid/
      google-ads-data-[YYYY-MM-DD].md
      meta-ads-data-[YYYY-MM-DD].md
      ad-library-data-[YYYY-MM-DD].md   ← competitor ads (Chrome)
    /analytics/
      ga4-data-[YYYY-MM-DD].md
      gbp-data-[YYYY-MM-DD].md
    /seo/
      gsc-data-[YYYY-MM-DD].md
    /commerce/
      merchant-data-[YYYY-MM-DD].md
  /audits/
    paid-review-[YYYY-MM-DD].md         ← standalone reviews
```

## Audience Modes

- **Agency**: per-client account mapping in `client-brief.md` under `## Windsor Accounts`. Multiple ad accounts per client are common — confirm which to pull.
- **Merchant**: single set of own ad accounts; mapping in your brand brief.

## Related Skills / Commands

- **`marketing-framework`** — operating principles
- **`competitor-research`** — competitor ad creative analysis
- **`content-brand`** — for ad creative drafting
- `/campaign-plan`
