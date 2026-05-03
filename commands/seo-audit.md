---
description: Run a full SEO and GEO audit for a client
argument-hint: [client-name]
allowed-tools: Read, Write, Glob, Grep, TodoWrite, Task
---

Read the marketing-framework skill and `${CLAUDE_PLUGIN_ROOT}/skills/seo-geo/references/seo-audit-methodology.md` before starting.

> **Connector reference:** See [CONNECTORS.md](../CONNECTORS.md). Primary tools: **Semrush** (client + competitor SEO — domain authority, keywords, backlinks, content gaps); **Windsor `get_data`** (client's own GSC + GA4) with **Chrome → respective platform UI fallback** when Windsor isn't connected; **Chrome** for SERP and AI Overview inspection.

Run a comprehensive SEO and GEO (Generative Engine Optimization) audit for client "$ARGUMENTS".

## Step 0: Prerequisite — Client Folder

Before running, the client folder must exist at `/clients/$ARGUMENTS/` with `client-brief.md`. If not:

> "No client folder found for $ARGUMENTS. Run `/new-client $ARGUMENTS` first to create the brief and folder structure, then re-run this command."

Stop here if the folder is missing.

## Step 1: Load Client Context

Read `/clients/$ARGUMENTS/client-brief.md`. Extract:
- Website URL
- Tracked keywords
- Competitor URLs
- Windsor Account IDs (GA4, GSC, Google Ads)
- Any previous audit data from `/clients/$ARGUMENTS/raw-data/seo/`

## Step 2: Domain Health Overview

**Semrush MCP:**
- Authority Score for the homepage and the domain
- Authority Score history over the last 6 months (trend direction)
- Total organic traffic estimate
- Total referring domains count
- Referring domains history (growing or declining?)

Save to `/clients/$ARGUMENTS/raw-data/seo/seo-audit-domain-[YYYY-MM-DD].md`.

> *If Semrush is not connected:* Skip Steps 2-6. Note "SEO audit (Semrush sections) skipped — connect Semrush to enable." Continue with Windsor GSC (Step 9), GA4 (Step 10), and Chrome-based technical / SERP checks.

## Step 3: Keyword Rankings Analysis

**Semrush MCP:**
- All organic keywords the site ranks for (top 100)
- Ranking distribution: positions 1-3, 4-10, 11-20, 21-50, 51-100
- Top 20 keywords by traffic value
- Keywords that improved in the last 30 days (top 10)
- Keywords that declined in the last 30 days (top 10)
- Keywords on page 2 (positions 11-20) — quick-win opportunities

Save to `/clients/$ARGUMENTS/raw-data/seo/seo-audit-keywords-[YYYY-MM-DD].md`.

## Step 4: Backlink Profile

**Semrush MCP:**
- Total backlinks and referring domains
- New backlinks acquired in last 30 days (count and notable sources)
- Lost backlinks in last 30 days
- Top referring domains by Authority Score
- Anchor text distribution
- Dofollow vs nofollow ratio

Save to `/clients/$ARGUMENTS/raw-data/seo/seo-audit-backlinks-[YYYY-MM-DD].md`.

## Step 5: Content Gap Analysis

**Semrush MCP:**
- Run keyword gap / domain comparison: keywords competitors rank for that the client doesn't
- Filter for keywords with volume > 100 and difficulty < 50 (or appropriate threshold)
- Group by topic cluster
- Identify the highest-value content opportunities

Save to `/clients/$ARGUMENTS/raw-data/seo/seo-audit-content-gaps-[YYYY-MM-DD].md`.

## Step 6: Top Pages Analysis

**Semrush MCP:**
- Top 20 pages by organic traffic
- Top 20 pages by number of referring domains
- Pages with high traffic but declining rankings
- Pages with strong backlinks but underperforming in traffic

Save to `/clients/$ARGUMENTS/raw-data/seo/seo-audit-top-pages-[YYYY-MM-DD].md`.

## Step 7: Technical Spot-Check

**Chrome browser:**
- Visit the client's website
- Check page load speed (note if visibly slow)
- Check mobile responsiveness
- Look for broken links, missing images, poor navigation
- Check meta titles and descriptions on key pages
- Check HTTPS, sitemap accessibility, robots.txt

Note: This is a spot-check, not a full technical crawl. Flag issues for deeper investigation.

Save to `/clients/$ARGUMENTS/raw-data/seo/seo-audit-technical-[YYYY-MM-DD].md`.

## Step 8: SERP and GEO Analysis

**Chrome browser — For the client's top 5 keywords:**
- Search each keyword on Google
- Note the SERP layout: ads, featured snippets, People Also Ask, AI Overview, local pack
- If an AI Overview appears: which sources cited? client cited? competitors cited? content type pulled (lists, definitions, comparisons)?
- Note which competitors appear in top 10 organic results

**Semrush MCP — SERP overview:**
- Pull SERP overview for top 5 keywords
- Note Authority Score and traffic of ranking pages

Save to `/clients/$ARGUMENTS/raw-data/seo/seo-audit-serp-[YYYY-MM-DD].md`.

## Step 9: Google Search Console Data (Windsor → Chrome fallback)

**Primary — Windsor `get_data` connector `searchconsole`:**

Use the client's GSC Account ID from `client-brief.md`.

**Overall performance — Last 90 days vs previous 90 days:**
- Fields: `date`, `clicks`, `impressions`, `ctr`, `position`
- Pull totals for both periods; calculate period-over-period changes

**Query-level performance (last 90 days):**
- Fields: `query`, `clicks`, `impressions`, `ctr`, `position`
- Top 20 queries by clicks
- Quick-win queries (position 8–20): filter `[["position", "gte", 8], "and", ["position", "lte", 20]]`
- High impressions + low CTR — title/meta optimization candidates

**Page-level performance (last 90 days):**
- Fields: `page`, `clicks`, `impressions`, `ctr`, `position`
- Top 20 pages by clicks

**Branded vs non-branded split:**
- Fields: `branded_vs_nonbranded`, `clicks`, `impressions`

> *If Windsor not connected:* Fall back to Chrome → `search.google.com/search-console`. Capture the same fields manually (top queries, top pages, totals over 90 days). Note the Chrome fallback in the file.
>
> *If GSC Account ID is not in the client brief:* Note "GSC data unavailable — add the GSC Account ID to client-brief.md or sign into Search Console manually." Continue with remaining steps.

Save to `/clients/$ARGUMENTS/raw-data/seo/seo-audit-gsc-[YYYY-MM-DD].md`.

## Step 10: GA4 Organic Performance (Windsor → Chrome fallback)

**Primary — Windsor `get_data` connector `googleanalytics4`:**
- Fields: `date`, `sessions`, `users`, `conversions`, `totalrevenue`, `landing_page`, `source_medium`
- Account: client's GA4 Account ID
- Filter for organic: `[["default_channel_group", "eq", "Organic Search"]]`
- Pull last 90 days and previous 90 days
- Top landing pages from organic search by sessions and conversions
- Organic conversion rate trend
- Compare last 30 days vs previous 30 days

> *If Windsor not connected:* Fall back to Chrome → `analytics.google.com`, Reports → Acquisition → Traffic acquisition → filter to Organic Search. Capture the same fields.
>
> *If GA4 Account ID missing:* Note "GA4 data unavailable — add the GA4 Account ID to client-brief.md."

Save to `/clients/$ARGUMENTS/raw-data/seo/seo-audit-ga4-[YYYY-MM-DD].md`.

## Step 11: Generate SEO Audit Report

Compile all findings into a structured audit report at `/clients/$ARGUMENTS/audits/seo-audit-[YYYY-MM-DD].md`:

1. **SEO Health Score** — Overall (Strong / Needs Work / Critical)
2. **Domain Authority Profile** — Authority Score trend, backlink health, competitive position
3. **Keyword Performance** — Ranking distribution, movers, quick-win opportunities
4. **Google Search Console Insights** — real click/impression data, CTR optimization opportunities, queries with high impressions but low CTR, page-level performance, branded vs non-branded split
5. **Content Gap Opportunities** — top 10 content pieces to create, prioritized by impact (cross-reference GSC query data with Semrush keyword gaps)
6. **Backlink Assessment** — link building opportunities and concerns
7. **Technical Issues** — anything from the spot-check
8. **GEO Readiness** — AI Overview presence, citability gaps, llms.txt status, schema coverage
9. **SERP Landscape** — what the search results look like for key terms
10. **Quick Wins** — 5 actions that can improve rankings within 30 days (include CTR optimization)
11. **Strategic Priorities** — 90-day SEO roadmap with expected impact

Every recommendation must cite specific data from the audit.

Present the audit to the user.

## Follow-Up Prompts

After presenting:
- "Want me to `/draft-content $ARGUMENTS` targeting the top content gap keywords?"
- "Want me to `/competitor-scan $ARGUMENTS` for a full competitive deep-dive?"
- "Want me to `/campaign-plan $ARGUMENTS` to build an SEO-focused campaign?"
