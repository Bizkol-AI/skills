# Connectors

This plugin ships with five MCP servers wired into `.mcp.json`. Bizkol is the only one required; the others are optional and authorize independently via OAuth on first tool call. Anything outside these five connectors is handled via Chrome browser automation.

| MCP | Required | URL | Used for |
|-----|----------|-----|----------|
| `bizkol` | ✅ | `https://mcp.bizkol.ai/api/mcp` | KOL discovery, campaigns, outreach, performance, social lookups, Reddit VoC |
| `windsor` | Optional | `https://mcp.windsor.ai` | Owned analytics — GA4, GSC, Google Ads, Meta Ads, GBP, Merchant |
| `shopify` | Optional | `https://setup.shopify.com/mcp` | Commerce data — orders, products, customers, sales, inventory |
| `semrush` | Optional | `https://mcp.semrush.com/v1/mcp` | SEO + competitor intelligence — keywords, traffic, backlinks, content gaps |
| `canva` | Optional | `https://mcp.canva.com/mcp` | Creative asset generation |

---

## Bizkol MCP — the core (REQUIRED for KOL work)

**URL:** `https://mcp.bizkol.ai/api/mcp`
**Auth:** OAuth (browser-based, one-click on first call)
**Scope:** All campaigns, KOLs, social lookups, Reddit research, email threads, and billing data are scoped to the authenticated Bizkol account.

The Bizkol MCP is the **primary data and action source** for all influencer/KOL workflows in this plugin, plus Reddit voice-of-customer research.

### Bizkol MCP tools — quick reference

#### Campaigns

| Tool | Purpose |
|------|---------|
| `create_campaign` | Create a new campaign (`name` required; `description`, `startDate`/`endDate`, `status`, `settlementCurrency`, `tags[]` optional) |
| `get_campaign` | Get campaign details by `campaignId` |
| `list_campaigns` | List campaigns; optional `status` filter (`draft`/`active`/`paused`/`completed`/`archived`) |
| `update_campaign` | Patch any campaign field by `campaignId` |
| `get_campaign_import_jobs` | Track async username-based KOL imports |

#### Campaign KOLs

| Tool | Purpose |
|------|---------|
| `add_kols_to_campaign` | Add KOLs by `kolId` (use `search_kols` first); supports `initialStatus` and `notes` |
| `get_campaign_kols` | List KOLs in a campaign with status, performance, social handles, quotes |
| `import_kols_to_campaign_by_username` | Bulk add KOLs by social handle (auto-scrapes new profiles); **async** — poll via `get_campaign_import_jobs` |
| `remove_kols_from_campaign` | Remove KOLs from a campaign (does not delete from KOL DB) |
| `update_campaign_kol` | Per-KOL updates (status, notes, quotes, contact info, custom affiliate/discount); up to 100 items per call |

#### KOLs

| Tool | Purpose |
|------|---------|
| `get_kol` | Full KOL profile (social accounts, AI descriptions, pricing) |
| `get_kol_performance` | Performance metrics for a KOL on a specific platform |
| `get_kol_posts` | KOL's recent posts with engagement |
| `search_kols` | Search KOL DB with filters (`query`, `platform`, follower/engagement/price ranges) |

#### Social (live lookups)

| Tool | Purpose |
|------|---------|
| `get_social_kol_profile` | Live profile for a KOL on a platform (Instagram, TikTok, YouTube, X) |
| `get_social_post_info` | Live post details + engagement |
| `search_social_kols` | Discover creators by topic/keyword on a platform |

#### Reddit (voice-of-customer)

| Tool | Purpose |
|------|---------|
| `search_reddit` | Search Reddit posts and comments by query |
| `get_reddit_post` | Full post details including comment tree |
| `get_reddit_subreddit` | Subreddit metadata, recent posts, subscriber count |
| `get_reddit_user` | User profile and posting history |

#### Email

| Tool | Purpose |
|------|---------|
| `get_email_conversation` | Full email thread (`threadId`, `campaignId`, `kolId`) |
| `list_email_conversations` | List threads, paginated; filter by `status` (`active`/`closed`/`awaiting_reply`/`replied`) |

#### Billing & account

| Tool | Purpose |
|------|---------|
| `get_billing_status` | Current plan, trial, billing status |
| `get_invoices` | Invoice history |
| `get_usage_summary` | Usage counters and limits |
| `get_user_profile` | Account info |

### Bizkol-first rule

For anything KOL-related, **always use Bizkol MCP first** — `search_kols`, `import_kols_to_campaign_by_username`, `get_campaign_kols`, etc. For competitor / VoC research, `search_reddit`, `search_social_kols`, and `get_social_kol_profile` cover most needs without leaving Claude.

---

## Windsor.ai — owned analytics (OPTIONAL)

**URL:** `https://mcp.windsor.ai`
Windsor.ai unifies the **client's / merchant's own** performance data across Google and Meta platforms behind a single `get_data` tool.

**Critical rule — Windsor is for OWNED data only:**
- **USE Windsor** for: GA4, GSC, Google Ads, Meta Ads, Google Business Profile, Google Merchant
- **DO NOT use Windsor** for: competitor research, competitor ads, third-party brand data
- **For competitor data:** Semrush (SEO), Chrome (Meta Ad Library, Google Ads Transparency, competitor sites, reviews), Bizkol MCP (social + Reddit)

### Account mapping

Each client/brand stores its Windsor account IDs in their `client-brief.md`:

```
## Windsor Accounts
- **GA4 Account ID:** [ID]
- **GSC Account ID:** [sc-domain:xxx or URL]
- **Google Ads Account ID:** [ID or "Not connected"]
- **Meta Ads Account ID:** [ID or "Not connected"]
- **GBP Account ID:** [ID or "Not connected"]
- **Merchant Account ID:** [ID or "Not connected"]
```

Run `/new-client` to be walked through populating these.

### Windsor `get_data` pattern

```
Tool: get_data
Parameters:
  connector: "<connector_id>"       # e.g. "googleanalytics4", "searchconsole", "google_ads"
  accounts: ["<account_id>"]
  fields: ["field1", "field2", ...]
  date_from: "YYYY-MM-DD"          # optional
  date_to: "YYYY-MM-DD"            # optional
  date_preset: "last_7d"           # alternative to date_from/date_to
  filters: [["field", "op", "val"]] # optional
```

### Windsor field reference — GA4 (`googleanalytics4`)

**Traffic:** `sessions`, `users`, `newusers`, `totalusers`, `active_users`, `bounce_rate`, `engaged_sessions`, `engagement_rate`, `average_session_duration`, `screen_page_views`, `screen_page_views_per_session`

**Conversions / revenue:** `conversions`, `totalrevenue`, `transactionrevenue`, `transactions`, `ecommerce_purchases`, `purchase_revenue`, `average_purchase_revenue`, `return_on_ad_spend`, `add_to_carts`, `checkouts`, `items_purchased`

**Dimensions:** `date`, `default_channel_group`, `source`, `medium`, `source_medium`, `campaign`, `landing_page`, `page_path`, `pagetitle`, `devicecategory`, `country`, `city`, `new_vs_returning`, `browser`, `operating_system`

**Google Ads attribution (within GA4):** `google_ads_campaign_name`, `google_ads_ad_group_name`, `google_ads_keyword`, `google_ads_query`, `google_ads_campaign_type`, `advertiser_ad_clicks`, `advertiser_ad_cost`, `advertiser_ad_impressions`

### Windsor field reference — Google Search Console (`searchconsole`)

**Metrics:** `clicks`, `impressions`, `ctr`, `position`
**Dimensions:** `date`, `query`, `page`, `pagepath`, `device`, `country`, `countryname`, `search_type`, `search_appearance`, `hostname`, `branded_vs_nonbranded`
**Useful options:** `include_fresh_data: "true"`

### Windsor field reference — Google Ads (`google_ads`)

**Metrics:** `clicks`, `impressions`, `cost_micros`, `average_cpc`, `conversions`, `all_conversions`, `all_conversions_value`, `average_cost`, `cost_per_conversion`
**Dimensions:** `campaign_name`, `campaign_id`, `campaign_status`, `campaign_advertising_channel_type`, `ad_group_name`, `ad_group_id`, `ad_group_status`, `date`

> `cost_micros` is in micros — divide by 1,000,000 for actual cost. ROAS = `all_conversions_value / (cost_micros / 1000000)`.

### Windsor field reference — Meta Ads (`facebook`)

**Typical fields once connected:** `campaign_name`, `adset_name`, `ad_name`, `spend`, `impressions`, `clicks`, `ctr`, `cpc`, `cpm`, `reach`, `frequency`, `conversions`, `cost_per_conversion`, `roas`, `date`
**Connect at:** <https://onboard.windsor.ai?datasource=facebook>

### Windsor field reference — Google Business Profile (`google_my_business`)

**Typical fields:** `business_name`, `queries_direct`, `queries_indirect`, `queries_chain`, `views_maps`, `views_search`, `actions_website`, `actions_phone`, `actions_driving_directions`, `photos_views_merchant`, `photos_views_customers`, `date`
**Connect at:** <https://onboard.windsor.ai?datasource=google_my_business>

### Windsor field reference — Google Merchant Center (`google_merchant`)

**Typical fields:** `product_title`, `product_id`, `brand`, `clicks`, `impressions`, `ctr`, `date`, `offer_id`, `product_type`
**Connect at:** <https://onboard.windsor.ai?datasource=google_merchant>

---

## Semrush MCP — SEO & competitor intelligence (OPTIONAL)

**URL:** `https://mcp.semrush.com/v1/mcp`
**Use for:** All client AND competitor SEO data — domain analytics, keyword rankings, backlinks, content gaps, traffic estimates, Position Tracking.

Semrush is the **single source of truth for both client and competitor SEO** in this toolbox. It replaces what was historically Ahrefs / Similarweb usage — anywhere this docs or older commands say "Ahrefs" or `~~SEO`, read **Semrush**.

**Typical jobs:**
- Domain overview (Authority Score, organic traffic, ranking keyword count, referring domains)
- Organic keyword positions (with volume, KD, position, URL)
- Keyword Gap / Domain Comparison (client vs competitors)
- Backlink Analytics (referring domains, growth trend, top sources)
- Top Pages by traffic
- Position Tracking (if a project is set up for the client)

---

## Shopify MCP — commerce data (OPTIONAL but powerful)

**URL:** `https://setup.shopify.com/mcp`
**Use for:** Merchant store data — orders, products, customers, sales, inventory, AOV, repeat-purchase rate, top SKUs, category performance.

Provides **commerce ground truth** for merchants. Pairs with Windsor for full attribution: Windsor tells you the marketing inputs (sessions, ad spend), Shopify tells you the outputs (orders, revenue, AOV, retention).

**Powers the `shopify-commerce` skill and the `/shopify-review` command.** Typical jobs:
- Store performance overview — revenue trend, AOV, conversion rate, repeat-customer rate
- Product analytics — best/worst sellers, category mix, inventory health, price-elasticity signals
- Customer analytics — new vs returning, LTV cohorts, geographic concentration
- Marketing × commerce attribution — pair Shopify orders with Windsor GA4/Meta/Google sessions to compute true CAC/ROAS
- Inventory + restock guidance — flag low-stock SKUs against current sell-through

If Shopify is not connected, fall back to Chrome → Shopify admin (`*.myshopify.com/admin`).

---

## Canva MCP — creative generation (OPTIONAL)

**URL:** `https://mcp.canva.com/mcp`
**Use for:** Generating social graphics, ad creative, basic visual assets directly from Claude.

---

## Browser tools — Chrome (fallback for everything else)

Chrome browser automation is the **universal fallback** when there's no dedicated MCP for a job. Primary use cases:

**Competitor research (always Chrome):**
- Meta Ad Library — competitor ad creative, messaging, offers
- Google Ads Transparency Center — competitor Google Ads
- Competitor websites — positioning, pricing, content
- Review platforms (Google Reviews, Trustpilot, G2, Capterra, App Store)
- Google News — press and PR

**SERP & GEO inspection:**
- Live Google SERP for AI Overview presence
- Perplexity / ChatGPT / Gemini citation checks
- Featured snippets, People Also Ask, knowledge panels

**Owned analytics fallback (when Windsor isn't connected for that platform):**

| Platform | Chrome fallback URL | What to capture |
|---|---|---|
| GA4 | analytics.google.com | Sessions, users, conversions, channel breakdown, top landing pages — same fields as the Windsor `googleanalytics4` connector |
| Search Console | search.google.com/search-console | Clicks, impressions, CTR, position by query and page |
| Google Ads | ads.google.com | Spend, clicks, impressions, conversions per campaign |
| Meta Ads | adsmanager.facebook.com | Spend, ROAS, CPM, CTR per campaign / ad set / ad |
| Google Business Profile | business.google.com | Search queries, views, actions |
| Google Merchant Center | merchants.google.com | Product impressions / clicks / CTR |
| Shopify (if Shopify MCP unavailable) | `*.myshopify.com/admin` | Orders, products, customers, analytics |
| Email platform | Klaviyo / Mailchimp / etc. | Campaign opens, clicks, conversions, list growth |

The fallback should produce the **same data shape** as the corresponding MCP — so downstream analysis stays consistent regardless of which path the data came in by.

**Anything else without an MCP** — Chrome is the catch-all. Sign in once and Claude reads dashboards via browser automation.

> **Reminder:** Windsor must **never** be used for competitor data. Owned data only.
