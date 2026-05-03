---
name: shopify-commerce
description: >
  Use this skill for any Shopify store performance work — orders trend, AOV
  and LTV, top / bottom SKUs, category mix, inventory health, customer
  cohorts (new vs. returning, repeat-purchase rate, geographic
  concentration), and marketing × commerce attribution (pairing Shopify
  outputs with Windsor GA4 / Meta Ads / Google Ads inputs to compute true
  CAC and ROAS). Trigger on phrases like "Shopify review", "store
  performance", "AOV", "LTV", "repeat purchase", "best sellers", "inventory
  check", "commerce review", "store revenue", "cohort", "true ROAS",
  "Shopify orders", or the `/shopify-review` command.
metadata:
  version: "0.1.0"
---

# Shopify Commerce Skill

Shopify is the commerce ground truth for any merchant on the platform. This skill covers store performance analysis, product and customer insights, inventory health, and — critically — joining Shopify outputs (orders, revenue, AOV, retention) with Windsor's marketing inputs (sessions, ad spend) to compute the metrics that actually matter: true CAC, true ROAS, contribution margin per channel.

> **Read first:** `marketing-framework/SKILL.md` for tool-priority rules, the data-first principle, and the new-client prerequisite. Shopify is **owned data only** — the same boundary as Windsor.

## Prerequisite — Client Folder

Before running, the client folder must exist at `/clients/[client-name]/` with `client-brief.md` populated. If missing, instruct the user to run `/new-client [name]` first.

## When to Use This Skill

- The user asks for a Shopify / commerce / store-performance review
- Building the commerce section of an initial audit
- Diagnosing revenue, AOV, or repeat-purchase changes
- Inventory check (low-stock alerts, slow-mover identification)
- Computing true CAC / true ROAS by joining marketing spend with order data
- Pre-launch SKU planning or restock decisions

## Tool Priority

| Need | Primary | Fallback |
|------|---------|----------|
| Orders, revenue, AOV, transactions | Shopify MCP | Chrome → `*.myshopify.com/admin/orders` |
| Products, SKU performance, inventory | Shopify MCP | Chrome → `/admin/products` |
| Customers, segments, repeat-purchase | Shopify MCP | Chrome → `/admin/customers` |
| Marketing-attributed sessions / ad spend | Windsor MCP `get_data` (`googleanalytics4`, `google_ads`, `facebook`) | Chrome → respective platform UI |
| Product feed performance (Shopping Ads) | Windsor MCP `get_data` (`google_merchant`) | Chrome → Merchant Center |
| Search demand for product categories | Semrush MCP | — |

**Owned-data only:** Shopify is for the client's / merchant's own store. Never use it to look up competitor stores.

## Standard Output Sections

Every Shopify review produces these sections (data first, analysis after):

1. **Executive summary** — revenue, orders, AOV, repeat-customer rate, with WoW or MoM delta and a one-line read.
2. **Revenue & order trend** — daily / weekly orders, revenue, AOV; flag anomalies (spikes, drops, weekend patterns).
3. **Product performance** — top 10 SKUs by revenue, top 10 by units, bottom 10 by sell-through, category mix.
4. **Inventory health** — low-stock SKUs (against current weekly sell-through), overstock SKUs (>90 days of cover), stockouts in the period.
5. **Customer analytics** — new vs returning split, repeat-purchase rate, cohort retention curve (M1 / M3 / M6), top geographies, top channels driving first orders.
6. **Marketing × commerce attribution** — for each major paid channel, pair Windsor spend with Shopify orders attributed to that channel:
   - Sessions × CVR × AOV ladder
   - True CAC = (channel ad spend) / (new customers from channel)
   - True ROAS = (channel revenue from new + returning) / (channel ad spend)
   - Contribution margin per channel (if COGS available)
7. **Anomalies** — sudden CVR drops, AOV shifts, refund spikes, fulfillment delays.
8. **Recommendations** — prioritized actions: which SKUs to restock, which to discount, which channels to scale or cut, which customer segments to target.

## Folder Output

```
/clients/[client-name]/
  /raw-data/
    /commerce/
      shopify-orders-[YYYY-MM-DD].md
      shopify-products-[YYYY-MM-DD].md
      shopify-customers-[YYYY-MM-DD].md
      shopify-inventory-[YYYY-MM-DD].md
      merchant-data-[YYYY-MM-DD].md          ← Google Merchant via Windsor (if connected)
  /audits/
    commerce-review-[YYYY-MM-DD].md          ← finished report
```

**Raw first, analysis after** — no exceptions. If Shopify MCP fails for a query, note the gap in the raw file and continue.

## Marketing × Commerce Attribution Pattern

The most valuable thing this skill does is **join sides**. Marketing data alone doesn't tell you whether spend was profitable; commerce data alone doesn't tell you which channel earned the order. Together they do.

Standard join:

```
For each channel (google_ads, facebook, organic, email, direct):
  windsor.spend       ← Windsor get_data per platform
  windsor.sessions    ← Windsor GA4 sessions for that source/medium
  shopify.orders      ← Shopify orders filtered by referring source/UTM
  shopify.revenue     ← Shopify revenue filtered same way
  shopify.new_customers
  → CAC = spend / new_customers
  → ROAS = revenue / spend
  → CVR = orders / sessions
  → AOV = revenue / orders
```

If Windsor isn't connected, the join becomes weaker — fall back to Chrome to capture top-line spend per platform, then proceed.

## Audience Modes

- **Agency**: per-client Shopify connection in their `client-brief.md` under `## Tech & Tracking → Shopify store`. Confirm which store URL to query before pulling.
- **Merchant**: single store mapped in your brand brief. Default to that store unless the user says otherwise.

## Related Skills / Commands

- **`marketing-framework`** — operating principles, folder layout, prerequisite rule
- **`paid-media`** — supplies the marketing-spend side of the attribution join
- **`seo-geo`** — supplies the organic / GSC side
- **`competitor-research`** — competitor product / price benchmarking (uses Chrome, not Shopify MCP)
- `/shopify-review` — the standalone commerce review command
