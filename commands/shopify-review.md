---
description: Run a full Shopify store performance review — orders, products, customers, inventory, plus marketing × commerce attribution
argument-hint: "[client-name]"
allowed-tools: Read, Write, Glob, Grep, TodoWrite, Task
---

Read the marketing-framework skill and the shopify-commerce skill before starting.

> **Connector reference:** See [CONNECTORS.md](../CONNECTORS.md). Primary: **Shopify MCP** for orders / products / customers / inventory. Pairs with **Windsor MCP** for paid + analytics inputs (GA4, Google Ads, Meta Ads, Merchant). Falls back to Chrome → `*.myshopify.com/admin` if Shopify MCP isn't connected, and Chrome → respective platform UIs if Windsor isn't connected.

Run a comprehensive commerce review for client "$ARGUMENTS".

## Step 0: Prerequisite — Client Folder

Before running, the client folder must exist at `/clients/$ARGUMENTS/` with `client-brief.md`. If not:

> "No client folder found for $ARGUMENTS. Run `/new-client $ARGUMENTS` first to create the brief and folder structure, then re-run this command."

Stop here if the folder is missing.

## Step 1: Load Client Context

Read `/clients/$ARGUMENTS/client-brief.md`. Extract:
- Shopify store URL (under Tech & Tracking)
- Windsor account IDs (GA4, Google Ads, Meta Ads, Merchant) — for the attribution join
- Business goals + north star metric — to anchor the read
- Current baselines (revenue, sessions, conversion rate) — for delta comparison
- Active marketing channels — to scope the attribution table

If Shopify is not connected and store URL is empty: note this and run a Chrome-fallback version (lighter — only what the Shopify admin exposes).

## Step 2: Orders & Revenue Trend

**Shopify MCP — last 90 days vs previous 90 days:**
- Daily orders, daily revenue, AOV
- Total period revenue, orders, AOV — with period-over-period delta
- Refund / return rate
- Order status breakdown (paid, fulfilled, cancelled)

Save raw to `/clients/$ARGUMENTS/raw-data/commerce/shopify-orders-[YYYY-MM-DD].md`.

## Step 3: Product Performance

**Shopify MCP:**
- Top 10 SKUs by revenue (last 90 days)
- Top 10 SKUs by units sold
- Bottom 10 SKUs by sell-through (in-stock but not selling)
- Category / product-type mix (% of revenue per category)
- New SKUs launched in the period and their first-90-days performance

Save to `/clients/$ARGUMENTS/raw-data/commerce/shopify-products-[YYYY-MM-DD].md`.

## Step 4: Inventory Health

**Shopify MCP:**
- Current inventory by SKU
- Compute weeks-of-cover = inventory / weekly sell-through (last 30 days)
- Flag low-stock: <2 weeks of cover on top-20 SKUs
- Flag overstock: >12 weeks of cover
- Stockout events in the period

Save to `/clients/$ARGUMENTS/raw-data/commerce/shopify-inventory-[YYYY-MM-DD].md`.

## Step 5: Customer Analytics

**Shopify MCP:**
- New vs returning customer split (orders + revenue)
- Repeat-purchase rate (% of customers with 2+ orders in last 12 months)
- Cohort retention curve — for monthly cohorts in the last 12 months, compute % of customers placing a 2nd order by M1, M3, M6
- Top 10 geographies (country / region) by revenue
- Top 5 channels driving first orders (where Shopify attributes them)
- Customer LTV estimate (avg revenue per customer over their lifetime to date)

Save to `/clients/$ARGUMENTS/raw-data/commerce/shopify-customers-[YYYY-MM-DD].md`.

## Step 6: Marketing × Commerce Attribution

This is the high-value section. Pair Windsor spend with Shopify orders per channel.

**For each channel the client runs (from client-brief Active Channels):**

| Channel | Windsor connector | Shopify attribution |
|---|---|---|
| Google Ads | `google_ads` | filter orders by `referring_site = google` or UTM `source=google` + `medium=cpc` |
| Meta Ads | `facebook` | filter orders by UTM `source=facebook` or `instagram` + `medium=cpc/paid_social` |
| Organic search | (Windsor `googleanalytics4` + `default_channel_group=Organic Search`) | filter orders by UTM `source=google` + `medium=organic` |
| Email | (Windsor GA4 `default_channel_group=Email`) | filter orders by UTM `source=email` or `medium=email` |
| Direct / branded | (GA4 `default_channel_group=Direct`) | unattributed orders + branded sessions |

For each channel, compute and report:
- Spend (from Windsor)
- Sessions (from Windsor GA4)
- Orders (from Shopify, attributed)
- Revenue (from Shopify, attributed)
- New customers (from Shopify)
- **CVR** = orders / sessions
- **AOV** = revenue / orders
- **True CAC** = spend / new customers
- **True ROAS** = revenue / spend
- **Contribution margin** if COGS available in Shopify

> *If Windsor is not connected:* fall back to Chrome → respective ad platform UIs to grab top-line spend per channel for the period. Note the fallback in the raw file. The attribution remains directional, not exact.

> *If a channel isn't running:* skip that row.

Save to `/clients/$ARGUMENTS/raw-data/commerce/attribution-[YYYY-MM-DD].md`.

## Step 7: Google Merchant Center (if connected)

**Windsor MCP `get_data` — `google_merchant`:**
- Fields: `product_title`, `product_id`, `clicks`, `impressions`, `ctr`, `date`
- Top 20 products by clicks
- Bottom 20 products by CTR (need feed optimization)
- Cross-reference with Shopify SKU performance — products with high impressions but low Shopify sales = listing or pricing issue

> *If Merchant not connected:* skip.

Save to `/clients/$ARGUMENTS/raw-data/commerce/merchant-data-[YYYY-MM-DD].md`.

## Step 8: Generate Commerce Review Report

Compile findings into the structured report. Save to `/clients/$ARGUMENTS/audits/commerce-review-[YYYY-MM-DD].md`:

1. **Executive summary** — revenue, orders, AOV, repeat rate, with WoW + MoM delta and a one-line read of what's driving it.
2. **Revenue & order trend** — chart description (daily orders / revenue), anomalies flagged.
3. **Product performance** — top sellers, bottom sellers, category mix, new-SKU performance.
4. **Inventory health** — low-stock list (urgent reorders), overstock list (discount candidates), stockout cost estimate.
5. **Customer analytics** — new vs returning, repeat rate, cohort retention, geo concentration, LTV.
6. **Marketing × commerce attribution table** — channel-by-channel CAC / ROAS / CVR / AOV with deltas.
7. **Google Merchant insights** (if connected).
8. **Anomalies** — anything surprising in the data, with possible cause.
9. **Quick wins (30 days)** — concrete actions: SKUs to restock, channels to cut, retargeting opportunities, AOV-lift bundles.
10. **Strategic priorities (90 days)** — where to invest growth budget based on the attribution view.

Every recommendation cites a specific data point. Present the report to the user.

## Follow-Up Prompts

After presenting:
- "Want me to `/campaign-plan $ARGUMENTS` focused on the under-served channels we found?"
- "Want me to `/draft-content $ARGUMENTS` for the bestseller category?"
- "Want me to `/competitor-scan $ARGUMENTS` to benchmark pricing on the underperforming SKUs?"
- "Want me to `/email-sequence $ARGUMENTS` for repeat-purchase reactivation based on the cohort drop-off?"

## Graceful Degradation

- **Shopify MCP not connected:** Prompt the user to connect at `https://setup.shopify.com/mcp` (one-click OAuth). If they can't, fall back to Chrome on their `*.myshopify.com/admin` and capture the same data shape — note the fallback in the raw file.
- **Windsor not connected:** Skip the precise attribution table; provide a directional read using Shopify's first-touch attribution + Chrome-grabbed ad spend totals. Note that true CAC/ROAS will be approximate.
- **Merchant not connected:** Skip Step 7.
