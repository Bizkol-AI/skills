---
name: marketing-framework
description: >
  The cross-cutting operating manual for the Bizkol marketing toolbox — read
  this skill at the start of any marketing, KOL, GTM, or commerce task. It
  defines tool priority (Bizkol-first for KOL, Windsor-first for owned
  analytics with Chrome fallback, Semrush for SEO, Shopify for commerce,
  Chrome for competitor research), the data-first analysis rule,
  graceful-degradation behavior, the agency vs. merchant mode distinction,
  the per-client folder layout under `/clients/[name]/`, the Bizkol-native
  campaign rule (no local campaign folder), the new-client prerequisite,
  and the index of focused skills to load for specialist work.
metadata:
  version: "0.4.0"
---

# Bizkol Marketing Toolbox — Operating Framework

This is the **always-load-first** skill for the Bizkol toolbox. It contains only the cross-cutting rules every command and every other skill depends on. Specialist methodology lives in the focused skills listed at the bottom.

## Audience

Two audiences share this toolbox:

- **Agencies** running campaigns on behalf of named clients (multi-client portfolios — many folders under `/clients/`).
- **Direct merchants** running campaigns for their own brand (one folder under `/clients/` for their own brand).

The toolbox is **unified** — both audiences use the same skills. The only difference is how many client folders exist and which connectors are typically wired up.

## Prerequisite — Client Folder MUST Exist

**Every client-scoped command requires a populated client folder before it runs.** The client folder is created by `/new-client`. If the user invokes any other command and the folder is missing, **stop and instruct them to run `/new-client [name]` first.**

```
If no /clients/[name]/client-brief.md exists for "$ARGUMENTS":
  STOP and respond:
  "No client folder found for [name]. Run `/new-client [name]` first to
   create the brief and folder structure, then re-run this command."
```

The only commands exempt from this rule are `/new-client` itself, and the **topic-only** modes of `/reddit-research` and `/gtm-research` (which save to a session folder when no client is named).

## Tool Priority (CRITICAL)

For any task, follow this order — never reach for a lower-priority tool when a higher-priority one applies.

| Job | Primary | Fallback if not connected |
|-----|---------|---------------------------|
| **KOL / influencer / creator work** | Bizkol MCP | None — Bizkol is required for KOL workflows |
| **Owned analytics** (GA4, GSC, Google Ads, Meta Ads, GBP, Merchant) | Windsor MCP `get_data` | Chrome → respective platform UI (GA4, Search Console, Ads Manager, etc.) |
| **Commerce data** (orders, products, customers, inventory, AOV) | Shopify MCP | Chrome → Shopify admin |
| **Own + competitor SEO** (Authority Score, keywords, backlinks, gaps) | Semrush MCP | Chrome → Google SERP, Search Console |
| **Competitor ads** | Chrome → Meta Ad Library, Google Ads Transparency Center | — |
| **Competitor social** | Bizkol MCP `get_social_kol_profile` / `get_social_post_info` | Chrome (LinkedIn, Pinterest, podcasts) |
| **Voice-of-customer** | Bizkol MCP `search_reddit` and friends | Chrome → forums, reviews |
| **Creative asset generation** | Canva MCP | — |

**The non-negotiables:**

1. **Bizkol MCP first for KOL work.** Anything KOL-related — discovery, campaigns, outreach, performance — routes through `mcp.bizkol.ai/api/mcp` first. Chrome is only the fallback for surfaces Bizkol doesn't yet cover.
2. **Windsor is owned-data only.** Use Windsor for the client's own paid + analytics data. Never use Windsor for competitor data — competitor SEO goes through Semrush, competitor ads through Chrome.
3. **If Windsor is not connected, fall back to Chrome** on the relevant platform UI for the same data — never fail the workflow.

See `CONNECTORS.md` (in the repo root) for the full connector reference and Bizkol MCP tool list.

## Data-First Operating Principle

**Complete ALL data collection before drawing ANY conclusions.**

After each data source, present raw findings in a structured format. Only begin analysis after all raw data is collected. Every conclusion must reference specific data points.

**Raw-data rule:**
- Always save raw data to `raw-data/` BEFORE performing analysis.
- Never overwrite — create new files (`[source]-[YYYY-MM-DD].md`) or append.
- If a research step returns no data, create the file anyway and note "No data found" with the reason.
- If a tool fails or a site blocks, note it in the file and continue with what's available.

## Graceful Degradation

When a tool category is not connected, commands and skills must:

1. Note the limitation in the output (one line).
2. Continue with the data that *is* available.
3. Suggest the connection (link to onboarding) rather than blocking the workflow.

**Never block a workflow because one tool is missing** — except the new-client prerequisite (above) and Bizkol-required KOL workflows.

## Bizkol-Native Campaign Rule

**KOL campaigns live in Bizkol's database, not in local files.** When the user creates or works with a campaign, the source of truth is the Bizkol MCP — `get_campaign`, `get_campaign_kols`, `list_email_conversations`, etc. Do **not** create local `/campaigns/[name]/` folders mirroring runtime state.

What lives locally:
- The **strategic brief** that informs campaign creation → `/clients/[name]/plans/kol-campaign-brief-[campaign].md`
- The **campaign index** → in `client-brief.md` mapping campaign-name → Bizkol `campaignId`
- One-off **generated artifacts** when the user asks for them:
  - Performance snapshots → `/audits/kol-performance-[campaign]-[YYYY-MM-DD].md`
  - Outreach reply drafts → `/content/drafts/outreach-[campaign]-[YYYY-MM-DD].md`
  - Discovery shortlists → `/audits/kol-discovery-[topic]-[YYYY-MM-DD].md`
- Optional **raw API archival** if explicitly requested → `/raw-data/kol/`

When the user asks "what's the status of campaign X" or "show me the shortlist", **always pull live from Bizkol** — never read a stale local file.

## Folder Layout (per client)

Every client (agency mode) or brand (merchant mode) gets the same structure under `/clients/[client-name]/`:

```
/clients/[client-name]/
  client-brief.md              ← single source of truth (brand, ICP, voice, Windsor IDs, Bizkol campaign index)
  /raw-data/                   ← every raw data pull, dated; never overwritten
    /seo/                        seo-audit-*, gsc-*, keyword-*, content-gaps-*
    /competitor/                 competitor-deep-dive-*, ad-library-*, press-*
    /voc/                        reddit-*, social-listening-*, reviews-*
    /analytics/                  ga4-*, gbp-*
    /paid/                       google-ads-*, meta-ads-*
    /commerce/                   shopify-orders-*, shopify-products-*, shopify-customers-*, merchant-*
    /kol/                        bizkol-search-*, campaign-snapshot-* (optional archival)
  /audits/                     ← finished analyses + reviews (one file per run, dated)
    initial-audit-[YYYY-MM-DD].md
    seo-audit-[YYYY-MM-DD].md
    competitor-scan-[YYYY-MM-DD].md
    voc-research-[topic]-[YYYY-MM-DD].md
    paid-review-[YYYY-MM-DD].md
    commerce-review-[YYYY-MM-DD].md
    brand-review-[YYYY-MM-DD].md
    kol-performance-[campaign]-[YYYY-MM-DD].md
    kol-discovery-[topic]-[YYYY-MM-DD].md
  /plans/                      ← strategy + planning docs
    gtm-plan-[YYYY-MM-DD].md
    campaign-plan-[YYYY-MM-DD].md
    kol-campaign-brief-[campaign].md
  /content/                    ← drafted, deliverable content
    /drafts/                     blog-, social-, landing-, press-, case-study-, outreach-
    /sequences/                  email-sequence-[type]-[YYYY-MM-DD].md
```

**No `/campaigns/` folder** — campaigns are Bizkol-native (see Bizkol-Native Campaign Rule above).

The `/new-client` command (via the **`gtm-research`** skill) creates this structure on onboarding.

## Skill Index — load the right one for the job

| If the job is… | Load skill |
|----------------|-----------|
| KOL / influencer / creator work | **`kol-operations`** |
| New client onboarding, GTM strategy, launch planning | **`gtm-research`** |
| SEO + GEO audits, search visibility, AI search readiness | **`seo-geo`** |
| Paid ads (Google, Meta, etc.), creative analysis, budget pacing | **`paid-media`** |
| Shopify store performance, products, customers, inventory | **`shopify-commerce`** |
| Competitor teardowns, battlecards, Reddit / VoC research | **`competitor-research`** |
| Drafting marketing content, brand voice review, email sequences | **`content-brand`** |

This skill (`marketing-framework`) covers only the cross-cutting rules. Anything channel- or workflow-specific lives in the focused skills above.

## Connector Reference

See **`CONNECTORS.md`** in the repo root for:
- Full Bizkol MCP tool list with parameter schemas (campaigns, KOLs, social, email, Reddit)
- Windsor MCP `get_data` patterns + field references per platform — and the Chrome fallback when not connected
- Semrush MCP tool reference (SEO + competitor intelligence)
- Shopify MCP tool reference (commerce data)
- Canva MCP tool reference (creative generation)
- Chrome browser playbooks for competitor research and platforms without an MCP
