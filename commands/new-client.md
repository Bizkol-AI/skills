---
description: Onboard a new client — pre-research from URL if provided, guided Q&A for what's missing, brief generation, folder setup, initial audit. Run this BEFORE any other client-scoped command.
argument-hint: [client-name-or-url]
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch, AskUserQuestion, TodoWrite, Task
---

Read the marketing-framework skill and `${CLAUDE_PLUGIN_ROOT}/skills/gtm-research/references/client-brief-template.md` before starting.

> **Connector reference:** See [CONNECTORS.md](../CONNECTORS.md). **Windsor `get_data` is the primary source for the client's own GA4, GSC, Google Ads, Meta Ads, GBP, and Merchant data — falls back to Chrome on the respective platform UI when not connected.** **Semrush** is the source for SEO data (client + competitor). **Shopify MCP** powers commerce data for merchants.

> **This command is the prerequisite for every other client-scoped command** (`/seo-audit`, `/competitor-scan`, `/shopify-review`, `/kol-campaign`, `/kol-discovery`, `/kol-outreach`, `/kol-performance`, `/campaign-plan`, `/draft-content`, `/email-sequence`, `/brand-review`). Without `/clients/[name]/client-brief.md`, those commands stop and ask the user to run `/new-client` first.

Onboard a new client. The argument "$ARGUMENTS" may be a name (`acme`), a URL (`https://acme.com`), or both (`acme https://acme.com`). Follow the steps in order.

## Step 0: Parse Argument and Pre-Research from URL

Parse "$ARGUMENTS":

- **If a URL is present** (anything matching `https?://...` or a bare domain like `acme.com`):
  1. Derive the **client slug** from the registrable domain (e.g., `bizkol.ai` → `bizkol`, `acme-corp.com` → `acme-corp`). Strip `www.`, the TLD, and any path. Use this as `[client-name]` for the folder; confirm it with the user at the end of pre-research before writing files.
  2. **Research the site and the company on the open web** to pre-fill as much of the brief as possible. The goal is to minimize the questions the client must answer.
     - **WebFetch** the homepage. Extract: business name, tagline, one-line description, primary product/service, pricing tiers if visible, target audience signals, contact info, geographic footprint.
     - **WebFetch** likely pages: `/about`, `/pricing`, `/products`, `/services`, `/contact`, `/team`, `/blog`, `/case-studies`, `/customers`. Skip silently if 404.
     - Inspect the homepage HTML/text for **tech signals**: Shopify (`cdn.shopify.com`, `myshopify.com`), Webflow, WordPress, Klaviyo (`klaviyo.com/onsite`), Mailchimp, GA4 (`G-XXXXXXX`), Meta Pixel (`fbq(`), Google Ads tag (`AW-`).
     - **WebSearch** queries to round out context (run 3–6 in parallel):
       - `"<brand>" review` / `"<brand>" reddit` (VoC + reputation signals)
       - `"<brand>" alternative` / `"<brand>" vs` (competitor surfaces)
       - `"<brand>" funding` / `"<brand>" crunchbase` (stage signals)
       - `"<brand>" pricing`
       - `site:linkedin.com/company "<brand>"` (size, headcount, location)
       - `site:trustpilot.com OR site:g2.com "<brand>"` (reviews — useful for VoC seed)
     - If browser automation is available **and** the homepage is JS-heavy (sparse WebFetch text), use Chrome to fetch the rendered page; note "rendered via Chrome" in the research log.
  3. Save **everything you found** to `/clients/[client-slug]/raw-data/voc/onboarding-research-[YYYY-MM-DD].md` and `/clients/[client-slug]/raw-data/competitor/competitor-list-from-search-[YYYY-MM-DD].md` (the latter for competitor candidates surfaced via "alternatives" / "vs" searches). Create the folder structure (Step 3) early so these files have a home.
  4. **Draft a pre-filled brief** using everything inferred from research. Mark each field with its source: `[from website]`, `[from search]`, `[inferred]`, or `[needs confirmation]`.
  5. **Show the user a research summary** before asking anything: what you found, what you inferred, what's still missing. Format:
     ```
     I researched [Brand] at [URL]. Here's what I gathered:

     ✅ Found
     - Business name, tagline, one-line description
     - Primary offering: [...]
     - Pricing visible: [...]
     - Tech detected: [Shopify | GA4 G-XXX | Meta Pixel | Klaviyo | ...]
     - Likely competitors (from "alternatives" search): [...]
     - Reviews / VoC signals: [...]

     🔎 Inferred (please confirm)
     - Target customer: [...]
     - Tone: [...]
     - Stage: [...]

     ❓ Still need from you
     - 90-day goal + north star metric
     - Current baseline numbers (revenue, sessions, orders/leads, conversion rate)
     - Monthly marketing budget
     - Windsor account IDs (GA4, GSC, Google Ads, Meta Ads, GBP, Merchant)
     - Tracked keyword list
     - Special instructions
     ```

- **If only a name is provided** (no URL): skip pre-research and go straight to Step 1. Ask for the website URL in Batch 1; once you have it, you may run pre-research before continuing the rest of the interview.

## Step 1: Guided Interview — Ask Only What's Missing

Use AskUserQuestion to fill the gaps left after Step 0. **Do not re-ask anything pre-research already answered confidently** — instead, present those values for confirmation in a single batched check ("I found these — anything to correct?"). Ask in batches of 2–4. Wait for answers before proceeding.

Order, in priority of "almost always needs the client":
1. **Business goals** — 90-day goal, north star metric, current baseline numbers (revenue, sessions, orders/leads, conversion rate). *Almost never on the website.*
2. **Marketing budget + history** — monthly budget, what's worked, what hasn't. *Internal knowledge.*
3. **Confirm inferred fields** — single batch: target customer, tone, stage, primary offering, pricing tiers, top competitors. Show what you inferred; ask for corrections.
4. **Competitors not surfaced by search** — "Anyone we should add or remove?"
5. **Tech & tracking** — GA4 Property ID, GSC verification status, Meta Pixel installed?, email platform + list size, Semrush project + tracked-keyword count, Shopify store URL (if merchant). Pre-fill anything detected on the site (e.g., GA4 ID from page source) and ask for confirmation.
6. **Windsor account mapping** — IDs the client must look up themselves:
   - GA4 Account ID — <https://onboard.windsor.ai?datasource=googleanalytics4>
   - GSC Account ID — `sc-domain:example.com` or full URL property; <https://onboard.windsor.ai?datasource=searchconsole>
   - Google Ads Account ID — typically `XXX-XXX-XXXX`; <https://onboard.windsor.ai?datasource=google_ads>
   - Meta Ads Account ID — <https://onboard.windsor.ai?datasource=facebook>
   - Google Business Profile Account ID — <https://onboard.windsor.ai?datasource=google_my_business>
   - Google Merchant Center Account ID — <https://onboard.windsor.ai?datasource=google_merchant>
   - **If Windsor is not connected**, note this in the brief — owned-data sections will fall back to Chrome on the respective platform UIs.
7. **Bizkol account check** — Confirm the user has signed in at <https://bizkol.ai>. If they haven't run a Bizkol command yet, mention the OAuth flow will trigger on first use.
8. **Shopify check (merchant only)** — If they sell on Shopify, confirm the store URL and whether the Shopify MCP is connected at <https://setup.shopify.com/mcp>.
9. **Reporting focus + cadence** — what the client most cares about seeing in reports, and meeting cadence (weekly / biweekly / monthly). *Reports always land in `/clients/[name]/audits/` and `/plans/`; do not ask about delivery method — there isn't one.*
10. **Tracked keyword list** — the 20–50 keywords to monitor in Semrush (seed from research if you found ranked keywords; ask for additions).
11. **Special instructions** — anything client-specific (forbidden words, claim rules, embargoes).

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

2. **Competitor Snapshot (light scan)** — Semrush domain comparison across the seed list; Chrome → Meta Ad Library spot-check; competitor website visit. This is a single-file snapshot, not a per-competitor teardown.
   - Save to `/clients/[name]/raw-data/competitor/competitor-snapshot-[YYYY-MM-DD].md` (one file covering the seed list) and `/raw-data/paid/` (ad library captures).
   - Full per-competitor teardowns (`competitor-deep-dive-[competitor]-*.md`) are produced by `/competitor-scan`, not here. If the user wants a real teardown during onboarding, point them to `/competitor-scan` as a follow-up.

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
