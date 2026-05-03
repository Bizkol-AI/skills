---
description: Run an ad-hoc competitor intelligence sweep for a client
argument-hint: [client-name]
allowed-tools: Read, Write, Glob, Grep, TodoWrite, Task
---

Read the marketing-framework skill and `${CLAUDE_PLUGIN_ROOT}/skills/competitor-research/references/competitor-analysis-methodology.md` before starting.

> **Connector reference:** See [CONNECTORS.md](../CONNECTORS.md). Primary: **Semrush** (competitor SEO), **Chrome** (Meta Ad Library, Google Ads Transparency Center, competitor sites, reviews, news), **Bizkol MCP** (competitor social + Reddit).

Run a full competitor intelligence sweep for client "$ARGUMENTS".

## Step 0: Prerequisite — Client Folder

Before running, the client folder must exist at `/clients/$ARGUMENTS/` with `client-brief.md`. If not:

> "No client folder found for $ARGUMENTS. Run `/new-client $ARGUMENTS` first to create the brief and folder structure, then re-run this command."

Stop here if the folder is missing.

## Step 1: Load Client Context

Read `/clients/$ARGUMENTS/client-brief.md`. Extract:
- All competitor names and URLs
- Client's own website and positioning
- Tracked keywords for comparison
- Industry context

## Step 2: SEO Competitor Comparison

**Semrush MCP — For each competitor:**
- Domain overview: Authority Score, organic traffic, referring domains, keyword count
- Pull the same metrics for the client for side-by-side comparison
- Keyword Gap / Domain Comparison on shared keywords
- Identify keywords where competitors rank but client doesn't (content gaps)
- Check for significant new pages indexed recently

> *If Semrush is not connected:* Skip SEO comparison. Note "Semrush not connected — competitive SEO analysis skipped." Continue with ad intelligence.

Save raw data to `/clients/$ARGUMENTS/raw-data/competitor/competitor-scan-seo-[YYYY-MM-DD].md`

## Step 3: Paid Advertising Intelligence

**Chrome — Meta Ad Library:**
- Search for each competitor by name
- Record: total active ads, ad formats used, messaging themes, offers
- Note any new campaigns launched recently
- Capture creative angles (UGC vs branded, video vs static, emotional vs rational)

**Chrome — Google Ads Transparency Center (if applicable):**
- Check if competitors are running Google Ads
- Note ad copy themes

Save raw data to `/clients/$ARGUMENTS/raw-data/paid/competitor-scan-ads-[YYYY-MM-DD].md`

## Step 4: Website and Positioning Analysis

**Chrome — Visit each competitor website:**
- Current positioning and messaging on homepage
- Any pricing changes or new offers
- New product launches or feature announcements
- Blog/content publishing activity
- Lead magnets or conversion tactics

Save raw data to `/clients/$ARGUMENTS/raw-data/competitor/competitor-scan-websites-[YYYY-MM-DD].md`

## Step 5: Social and Reputation

**Bizkol MCP — competitor social (Instagram, TikTok, YouTube, X):**
- `get_social_kol_profile` for each competitor handle on each platform — follower count, recent post engagement
- `get_social_post_info` for high-engagement recent posts — note posting frequency, content themes, viral / high-performing content

For platforms Bizkol doesn't cover (LinkedIn, Pinterest, podcasts), fall back to Chrome.

**Chrome — Reviews:**
- Check Google reviews, Trustpilot, G2 (as applicable) for each competitor
- Note rating changes, new review themes, common complaints

**Bizkol MCP — Reddit:**
- `search_reddit` for competitor brand mentions
- `get_reddit_post` on top-upvoted threads for full comment context
- Note sentiment and any trending discussions

Save raw data to `/clients/$ARGUMENTS/raw-data/voc/competitor-scan-social-[YYYY-MM-DD].md`

## Step 6: Press and PR

**Chrome — Google News:**
- Search for each competitor name
- Note any press coverage, partnerships, funding, or major announcements

Save raw data to `/clients/$ARGUMENTS/raw-data/competitor/competitor-scan-press-[YYYY-MM-DD].md`

## Step 7: Generate Competitor Intelligence Report

Compile all findings into a structured report:

1. **Competitor Matrix** — Side-by-side comparison table (Authority Score, traffic, keywords, social followers, ad activity, review scores)
2. **Positioning Map** — How each competitor positions themselves vs the client
3. **Ad Creative Intelligence** — What messaging and creative approaches competitors are using
4. **Content & SEO Gaps** — Keywords and topics where competitors are winning
5. **Reputation Signals** — What customers say about competitors (positive and negative)
6. **Threat Assessment** — Rank competitors by threat level with reasoning
7. **Opportunities** — Specific actions the client can take based on competitor weaknesses

Every finding must cite specific data.

Save to `/clients/$ARGUMENTS/audits/competitor-scan-[YYYY-MM-DD].md`

Present the report to the user.

## Follow-Up Prompts

After presenting the report, suggest relevant next steps:
- "Want me to `/draft-content $ARGUMENTS` to create content targeting competitor weaknesses?"
- "Want me to `/campaign-plan $ARGUMENTS` to build a campaign exploiting competitive gaps?"
- "Want me to `/reddit-research $ARGUMENTS` to understand how customers perceive these competitors?"
