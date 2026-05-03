---
description: Onboard a new client — guided Q&A, brief generation, folder setup, initial audit. Run this BEFORE any other client-scoped command.
argument-hint: [client-name]
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch, AskUserQuestion, TodoWrite, Task
---

Read the marketing-framework skill and `${CLAUDE_PLUGIN_ROOT}/skills/gtm-research/references/client-brief-template.md` before starting.

> **Connector reference:** See [CONNECTORS.md](../CONNECTORS.md). **Windsor `get_data` is the primary source for the client's own GA4, GSC, Google Ads, Meta Ads, GBP, and Merchant data — falls back to Chrome on the respective platform UI when not connected.** **Semrush** is the source for SEO data (client + competitor). **Shopify MCP** powers commerce data for merchants.

> **This command is the prerequisite for every other client-scoped command** (`/seo-audit`, `/competitor-scan`, `/shopify-review`, `/kol-campaign`, `/kol-discovery`, `/kol-outreach`, `/kol-performance`, `/campaign-plan`, `/draft-content`, `/email-sequence`, `/brand-review`). Without `/clients/[name]/client-brief.md`, those commands stop and ask the user to run `/new-client` first.

Onboard a new client named "$ARGUMENTS". Follow the steps in order.

## Step 1: Guided Interview

Use AskUserQuestion to walk the user through the onboarding questions defined in `${CLAUDE_PLUGIN_ROOT}/skills/gtm-research/references/client-brief-template.md`. Ask in batches of 2-4 — never dump all questions at once. Wait for answers before proceeding.

Key batches:
1. Business basics (type, website URL, platform, location)
2. What they sell (products/services, price points, target customer)
3. Business goals (90-day goal, north star metric, current baseline numbers)
4. Marketing history (active channels, budget, what worked/didn't)
5. Competitors (known competitor names and URLs)
6. Brand & positioning (tagline, tone, self-description)
7. Tech & tracking (GA4 Property ID, GSC property URL, Meta Pixel, email platform, Semrush project, Shopify store URL if merchant)
8. **Windsor account mapping** — Identify the client's Windsor account IDs (if Windsor is connected). Walk the user through finding each one if they're unsure:
   - GA4 Account ID — <https://onboard.windsor.ai?datasource=googleanalytics4>
   - GSC Account ID — `sc-domain:example.com` or full URL property; <https://onboard.windsor.ai?datasource=searchconsole>
   - Google Ads Account ID — typically `XXX-XXX-XXXX`; <https://onboard.windsor.ai?datasource=google_ads>
   - Meta Ads Account ID — <https://onboard.windsor.ai?datasource=facebook>
   - Google Business Profile Account ID — <https://onboard.windsor.ai?datasource=google_my_business>
   - Google Merchant Center Account ID — <https://onboard.windsor.ai?datasource=google_merchant>
   - **If Windsor is not connected**, note this in the brief — owned-data sections will fall back to Chrome on the respective platform UIs.
9. **Bizkol account check** — Confirm the user has signed in at <https://bizkol.ai>. If they haven't run a Bizkol command yet, mention the OAuth flow will trigger on first use.
10. **Shopify check (merchant only)** — If they sell on Shopify, note the store URL and whether the Shopify MCP is connected at <https://setup.shopify.com/mcp>.
11. Reporting preferences (delivery method, what they care about, meeting cadence)
12. Tracked keywords list and special instructions

## Step 2: Generate Client Brief

Generate `client-brief.md` following the template. **Include the Windsor Accounts section AND the Bizkol Campaign Index section** (initially empty — `/kol-campaign` populates it as campaigns are created):

```markdown
## Windsor Accounts
- **GA4 Account ID:** [ID or "Not connected"]
- **GSC Account ID:** [sc-domain:xxx or URL or "Not connected"]
- **Google Ads Account ID:** [ID or "Not connected"]
- **Meta Ads Account ID:** [ID or "Not connected"]
- **GBP Account ID:** [ID or "Not connected"]
- **Merchant Account ID:** [ID or "Not connected"]

## Bizkol Campaign Index
| Campaign Name | Bizkol campaignId | Status | Strategic Brief | Created |
|---|---|---|---|---|
(empty until /kol-campaign creates the first one)
```

Save to `/clients/$ARGUMENTS/client-brief.md` (use the client name lowercased and hyphenated for the folder).

## Step 3: Create Folder Structure

Create the full client structure:

```
/clients/[client-name]/
  client-brief.md
  /raw-data/
    /seo/
    /competitor/
    /voc/
    /analytics/
    /paid/
    /commerce/
    /kol/
  /audits/
  /plans/
  /content/
    /drafts/
    /sequences/
```

Use `[client-name]` = $ARGUMENTS lowercased with spaces → hyphens.

## Step 4: Run Initial Brand Audit

After the folder is set up, run a light-touch initial audit using the data in `client-brief.md`:

1. **SEO Audit (Semrush)** — domain overview, top keywords, content-gap pull against competitors, Chrome SERP/AI Overview spot-check on top 3 keywords.
   - Save raw to `/clients/[name]/raw-data/seo/`.
   - *If Semrush not connected:* skip and note "SEO audit skipped — connect Semrush to enable. Run `/seo-audit $ARGUMENTS` later."

2. **Competitor Analysis** — Semrush domain comparison for each competitor; Chrome → Meta Ad Library spot-check; competitor website visit.
   - Save raw to `/clients/[name]/raw-data/competitor/` and `/raw-data/paid/` (ad library).

3. **GA4 Baseline (Windsor → Chrome fallback)** — `get_data` with connector `googleanalytics4`:
   - Fields: `date`, `sessions`, `users`, `conversions`, `totalrevenue`, `default_channel_group`, `landing_page`
   - Account: client's GA4 Account ID; Date range: last 90 days
   - Aggregate: sessions, users, conversions, conversion rate, revenue; channel breakdown via `default_channel_group`; top landing pages and converting pages.
   - Save raw to `/clients/[name]/raw-data/analytics/ga4-baseline-[YYYY-MM-DD].md`.
   - *If Windsor not connected:* fall back to Chrome on `analytics.google.com` and capture the same fields. Note the fallback in the file.
   - *If GA4 Account ID missing entirely:* skip and note "GA4 baseline skipped — add the GA4 Account ID to client-brief.md."

4. **Google Search Console Baseline (Windsor → Chrome fallback)** — `get_data` with connector `searchconsole`:
   - Fields: `date`, `query`, `page`, `clicks`, `impressions`, `ctr`, `position`
   - Account: client's GSC Account ID; Date range: last 90 days
   - Pull: total clicks, impressions, average CTR, average position; top 20 queries by clicks; top 20 pages by clicks.
   - Save to `/clients/[name]/raw-data/seo/gsc-baseline-[YYYY-MM-DD].md`.
   - Same Chrome fallback to `search.google.com/search-console` if Windsor missing.

5. **Google Ads Baseline (Windsor → Chrome fallback)** — connector `google_ads`. Save to `/clients/[name]/raw-data/paid/google-ads-baseline-[YYYY-MM-DD].md`.

6. **Meta Ads Baseline (Windsor → Chrome fallback)** — connector `facebook`. Save to `/raw-data/paid/meta-ads-baseline-[YYYY-MM-DD].md`.

7. **Google Business Profile** — connector `google_my_business`. Save to `/raw-data/analytics/gbp-baseline-[YYYY-MM-DD].md`.

8. **Google Merchant** — connector `google_merchant`. Save to `/raw-data/commerce/merchant-baseline-[YYYY-MM-DD].md`.

9. **Shopify Baseline (merchant only)** — Shopify MCP: orders, products, customers summary for last 30 days. Save to `/clients/[name]/raw-data/commerce/shopify-baseline-[YYYY-MM-DD].md`.
   - *If Shopify MCP not connected:* fall back to Chrome → `*.myshopify.com/admin`.

## Step 5: Generate Initial Audit Report

Compile findings into `/clients/[name]/audits/initial-audit-[YYYY-MM-DD].md`:

- SEO health assessment and top issues
- Content gap opportunities (top keywords to target)
- Competitor threat ranking
- GEO readiness assessment
- Google Search Console insights (real traffic data)
- Google Ads performance baseline (if applicable)
- Meta Ads performance baseline (if applicable)
- Google Business Profile insights (if applicable)
- Shopify commerce baseline (if merchant)
- Quick wins (30-day actions)
- Strategic priorities (90-day roadmap)
- Channel recommendations

Present a summary to the user when complete.

## Follow-Up Prompts

After onboarding, suggest next steps based on what was found:
- "Run `/seo-audit $ARGUMENTS` for the full SEO + GEO deep-dive."
- "Run `/competitor-scan $ARGUMENTS` for the full competitor sweep."
- "Run `/shopify-review $ARGUMENTS` for the commerce performance + attribution view." (merchant only)
- "Run `/reddit-research $ARGUMENTS` to mine customer language and pain points."
- "Run `/campaign-plan $ARGUMENTS` to build the first campaign from the audit findings."
- "Run `/kol-campaign $ARGUMENTS` to spin up the first KOL campaign."
