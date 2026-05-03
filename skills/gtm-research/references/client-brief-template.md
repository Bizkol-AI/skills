# Client Brief Template

This is the master context file created by `/new-client`. It lives at `/clients/[client-name]/client-brief.md` and is read by Claude at the start of every report or analysis session.

**Reference path** (used by all commands): `${CLAUDE_PLUGIN_ROOT}/skills/gtm-research/references/client-brief-template.md`

## Onboarding Questions

When running `/new-client`, ask the user these questions using AskUserQuestion tool, grouped into logical batches of 2-4 questions at a time. Do not dump all questions at once.

### Batch 1: Business Basics
- Business name
- Website URL
- Business type: local service / e-commerce / both / SaaS / marketplace
- E-commerce platform: Shopify / WooCommerce / other / N/A

### Batch 2: What They Sell
- Primary product or service (2-3 sentence description)
- Price points (entry / mid / premium / average order value)
- Target customer description (who they are, where they are, what problem they're solving)

### Batch 3: Business Goals
- Primary goal for next 90 days (specific and measurable)
- North star metric (the one number that represents growth)
- Current baseline if established: monthly revenue, sessions, leads/orders, conversion rate

### Batch 4: Marketing History
- Active marketing channels (multi-select: SEO, Google Ads, Meta Ads, TikTok, Email, Organic Social, Influencer, PR, Other)
- Monthly marketing budget (total)
- What has worked before
- What has not worked

### Batch 5: Competitors
- Known direct competitors (names and URLs, up to 5)
- Who does the client most want to beat
- Where client believes they have an advantage

### Batch 6: Brand & Positioning
- Current tagline or positioning statement (if any)
- How they describe themselves
- Tone of voice preference

### Batch 7: Tech & Tracking
- GA4 Property ID
- Google Search Console: verified? (Yes/No)
- Meta Pixel installed? (Yes/No)
- Email platform (Klaviyo / Mailchimp / ActiveCampaign / other / none) and list size
- Semrush project set up? (Yes/No) and tracked keyword count
- Shopify store: connected to Shopify MCP? (Yes/No) — store URL (e.g. `mybrand.myshopify.com`)

### Batch 8: Windsor.ai Account Mapping
Map the client's platforms to Windsor.ai connected accounts. Show the user the available accounts from CONNECTORS.md and ask them to confirm which accounts belong to this client:

- **GA4 Account:** Select from available Windsor GA4 accounts or enter new ID
- **GSC Account:** Select from available Windsor GSC accounts or enter new domain
- **Google Ads Account:** Select from available Windsor Google Ads accounts or "Not connected"
- **Meta Ads Account:** Select from available Windsor Meta Ads accounts or "Not connected" (connect at: https://onboard.windsor.ai?datasource=facebook)
- **Google Business Profile Account:** Select or "Not connected" (connect at: https://onboard.windsor.ai?datasource=google_my_business)
- **Google Merchant Account:** Select or "Not connected" (connect at: https://onboard.windsor.ai?datasource=google_merchant)

### Batch 9: Reporting Preferences
- Report delivery preference: email / Slack / folder only
- What the client cares most about seeing in reports
- Meeting cadence: weekly / biweekly / monthly

### Batch 10: Cowork Configuration
- Tracked keywords list (the 20-50 keywords to monitor in Semrush)
- Any special instructions for this client

## Generated File Format

After collecting answers, generate `client-brief.md` in this exact format:

```markdown
# Client Brief: [Business Name]
*Last updated: [DATE]*
*Prepared by: Bizkol AI*

## Business Overview
- **Name:** [NAME]
- **Website:** [URL]
- **Type:** [TYPE]
- **Platform:** [PLATFORM]
- **Location:** [GEOGRAPHY]
- **Stage:** [idea / pre-launch / launched under 1 year / established]

## Products & Services
**Primary offering:** [DESCRIPTION]

**Price points:**
- Entry: $[X]
- Mid: $[X]
- Premium: $[X]
- Average order/transaction value: $[X]

## Target Customer
[DESCRIPTION — who they are, where they are, what problem they solve]

## Business Goals
- **90-day goal:** [SPECIFIC MEASURABLE GOAL]
- **North star metric:** [METRIC]
- **Current baseline:**
  - Monthly revenue: $[X]
  - Monthly sessions: [X]
  - Monthly leads/orders: [X]
  - Conversion rate: [X]%

## Marketing History
**Active channels:** [LIST]
**Monthly budget:** $[X]
**What has worked:** [DESCRIPTION]
**What has not worked:** [DESCRIPTION]

## Competitors
| Name | URL | Notes |
|------|-----|-------|
| [NAME] | [URL] | [NOTES] |

**Primary competitor to beat:** [NAME]
**Client's perceived advantage:** [DESCRIPTION]

## Brand & Positioning
- **Tagline:** [TEXT or "None yet"]
- **Self-description:** [HOW THEY DESCRIBE THEMSELVES]
- **Tone:** [TONE PREFERENCE]

## Tech & Tracking
- **GA4 Property ID:** [ID]
- **Google Search Console:** [Yes/No]
- **Meta Pixel:** [Yes/No]
- **Email platform:** [PLATFORM] — list size: [X]
- **Semrush project:** [Yes/No] — tracked keywords: [X]
- **Shopify store:** [Store URL or "Not connected"] (merchant only)

## Windsor Accounts
- **GA4 Account ID:** [ID or "Not connected"]
- **GSC Account ID:** [sc-domain:xxx or URL or "Not connected"]
- **Google Ads Account ID:** [ID or "Not connected"]
- **Meta Ads Account ID:** [ID or "Not connected"]
- **Google Business Profile Account ID:** [ID or "Not connected"]
- **Google Merchant Account ID:** [ID or "Not connected"]

## Reporting
- **Delivery:** [PREFERENCE]
- **Client cares about:** [WHAT MATTERS MOST]
- **Meeting cadence:** [FREQUENCY]

## Tracked Keywords
[LIST — one per line or comma-separated]

## Competitor URLs to Monitor
- [URL1]
- [URL2]
- [URL3]

## Bizkol Campaign Index

This section is the local index of KOL campaigns created in Bizkol. Update it whenever `/kol-campaign` creates a new campaign, or whenever Bizkol returns a new `campaignId`. Live state (KOLs, outreach, performance) always comes from the Bizkol MCP — never duplicate it here.

| Campaign Name | Bizkol campaignId | Status | Strategic Brief | Created |
|---|---|---|---|---|
| [name] | [id] | draft / active / paused / completed | `/clients/[name]/plans/kol-campaign-brief-[name].md` | [YYYY-MM-DD] |

## Special Instructions
[ANY CLIENT-SPECIFIC NOTES]
```
