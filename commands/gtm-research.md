---
description: Run GTM strategy research for an idea-stage client or new product. Topic-only mode supported when no client folder exists.
argument-hint: [client-name or topic]
allowed-tools: Read, Write, Glob, Grep, WebSearch, WebFetch, AskUserQuestion, TodoWrite, Task, mcp__reddit_scraper__Search_Posts, mcp__reddit_scraper__Posts_By_Subreddit, mcp__reddit_scraper__Post_Comments, mcp__reddit_scraper__Search_Subreddits, mcp__reddit_scraper__Subreddit_Info, mcp__twitter_scraper__Search_Twitter, mcp__twitter_scraper__Get_Trends_By_Location, mcp__tiktok_scraper__Get_Trending_Videos, mcp__tiktok_scraper__Search_Video, mcp__tiktok_scraper__Get_Trending_Hashtag, mcp__instagram_scraper__Media_by_hashtag, mcp__instagram_scraper__Search_users_by_keyword
---

Read the marketing-framework skill and these references before starting:
- `${CLAUDE_PLUGIN_ROOT}/skills/gtm-research/references/gtm-strategy-methodology.md`
- `${CLAUDE_PLUGIN_ROOT}/skills/competitor-research/references/reddit-research-methodology.md`
- `${CLAUDE_PLUGIN_ROOT}/skills/competitor-research/references/competitor-analysis-methodology.md`

> **Connector reference:** See [CONNECTORS.md](../CONNECTORS.md). Primary SEO/demand source is **Semrush** (replacing prior Ahrefs references). Voice-of-customer comes from **Bizkol MCP** Reddit tools and the standalone Reddit / Twitter / TikTok / Instagram / YouTube scrapers.

Run a full GTM Flywheel strategy research process for "$ARGUMENTS". Follow the five-stage flywheel methodology defined in `gtm-strategy-methodology.md` exactly.

---

## Step 0: Determine Research Context (with-client preferred, topic-only allowed)

Check whether `/clients/$ARGUMENTS/client-brief.md` exists.

**With-client mode (recommended):** Read the client brief. Outputs land under `/clients/$ARGUMENTS/`.

**Topic-only mode (allowed):** If no client folder exists, warn the user:

> "No client folder found for $ARGUMENTS. Running in topic-only mode — outputs will land in `/sessions/gtm-[topic-slug]-[YYYY-MM-DD]/` and the strategy will be less personalized. For a client-anchored GTM plan, run `/new-client $ARGUMENTS` first."

Then use AskUserQuestion to gather essentials:
1. What product/service are you launching?
2. Who is the target customer?
3. What problem does it solve?
4. Known competitors (if any)?
5. Approximate budget range and launch timeline?

For the rest of this command, `[OUTPUT-ROOT]` = `/clients/$ARGUMENTS` (with-client) or `/sessions/gtm-[topic-slug]-[YYYY-MM-DD]` (topic-only).

---

## Step 1: User Needs Insights

*Goal: Understand what real users actually want, fear, and complain about — in their own words.*

**Reddit Research (Reddit Scraper):**
- Search for 5-8 subreddits relevant to the problem domain
- Pull top posts from each subreddit (last 3 months)
- Pull comments from the top 10 posts — mine for exact pain-point language, workarounds people use, competitor complaints, and wish-lists
- Search Reddit for "[product category] alternatives", "[competitor name] review", "[problem keyword] help"

**Twitter/X Social Listening (Twitter Scraper):**
- Search for product category keywords and competitor brand names
- Find trending complaints and use-case discussions
- Capture verbatim quotes that reveal unmet needs

**TikTok & Instagram Trend Research (TikTok + Instagram Scrapers):**
- Search for product/problem hashtags — note which content formats get high engagement
- Look at comment sections of viral videos for raw customer language
- Identify trending hooks and pain-point framings

**SEO Demand Signals (Semrush MCP):**
- Pull keyword overview for 5-10 seed keywords
- Get matching terms and related terms to map the full demand landscape
- Note monthly volume, keyword difficulty, CPC — flag high-CPC keywords as proof of commercial intent
- Identify question-format keywords (how to / best / vs / alternative) as intent signals
- *If Semrush not connected:* fall back to Google Trends + Google Keyword Planner via Chrome.

**Web Search — Industry & Market:**
- Search "[product category] market size [year]", "[category] trends [year]"
- Search for recent customer survey reports, industry whitepapers
- Check Google News for pain-point-related headlines in the last 6 months

Save all raw data to `[OUTPUT-ROOT]/raw-data/voc/gtm-01-needs-insights-[YYYY-MM-DD].md` (with-client) or `[OUTPUT-ROOT]/gtm-01-needs-insights-[YYYY-MM-DD].md` (topic-only).

---

## Step 2: User Segmentation & Layering

*Goal: Slice the audience into distinct, actionable segments. Know who to speak to first, second, and later.*

**Segment Discovery:**
Using data from Step 1, identify 3-5 distinct user segments based on:
- **Job-to-be-done**: What are they actually trying to accomplish?
- **Pain intensity**: Who suffers most from the problem right now?
- **Buying behavior signals**: Who is actively searching, spending money, comparing options?
- **Platform presence**: Where does each segment live (Reddit, TikTok, LinkedIn, etc.)?

**For each segment, define:**
- Demographics and psychographics (draw from Reddit thread context, job titles in Twitter discussions, etc.)
- Core trigger event (what makes them search for a solution today?)
- Biggest fear / objection to buying
- Competitor they currently use (from research in Step 1)
- Estimated segment size and growth direction
- Priority tier: ICP (ideal customer profile) / Secondary / Long-term

**Audience Layering Framework:**
- **Layer 1 — Hot Demand**: Already aware of the solution, actively comparing — fastest to convert
- **Layer 2 — Problem-Aware**: Knows the pain, hasn't found the solution yet — educate to convert
- **Layer 3 — Latent Demand**: Has the pain but doesn't know it's solvable — content-led activation

**ICP Statement:**
Write a single ICP statement: "Our ideal first customer is [persona], who experiences [trigger event], currently uses [workaround/competitor], and will buy when [buying condition]."

Save to `[OUTPUT-ROOT]/raw-data/voc/gtm-02-segmentation-[YYYY-MM-DD].md` (with-client) or `[OUTPUT-ROOT]/gtm-02-segmentation-[YYYY-MM-DD].md` (topic-only).

---

## Step 3: Content & Creative Production

*Goal: Produce the actual messaging, positioning, and content ideas grounded in real data.*

**Competitor Content Intelligence:**
- Web search for competitors' blog content, case studies, landing page copy
- Use Instagram and TikTok scrapers to pull competitors' top-performing posts — note what hooks, formats, and CTAs they use
- Search Twitter for competitors' viral threads or campaign content

**Positioning & Messaging Architecture:**
Based on Segments (Step 2) and Needs Insights (Step 1):

1. **Whitespace Statement** — Where is the gap no competitor owns?
   > "[TARGET] who are frustrated by [PAIN FROM DATA] want [WISH FROM DATA] but currently [GAP FROM DATA]."

2. **Positioning Statement:**
   > "For [TARGET — use their language], [BRAND] is the [CATEGORY] that [DIFFERENTIATOR] — unlike [COMPETITOR] which [WEAKNESS FROM DATA]."

3. **Messaging Hierarchy:**
   - Primary message (one sentence — what you do and for whom)
   - 3 supporting proof points (each backed by a data point from research)
   - Top 3 objection responses (using exact fears captured in Step 1)

**Content Starter Kit — produce ready-to-use creative assets:**

For **each of the top 2 segments**, generate:
- 3 homepage hero headline options (in the customer's own language)
- 5 ad hook variations (one per awareness stage: unaware / problem-aware / solution-aware / product-aware / most-aware)
- 5 social media post concepts (with platform, format, hook, and body copy)
- 3 email subject line options for a cold outreach or launch sequence
- 1 short-form video script concept (30-45 seconds) built around a real Reddit pain-point quote

**SEO Content Plan:**
- Map the top 15 keywords from Step 1 to content types (landing page / blog post / comparison page / FAQ)
- Identify 3 "content pillars" that cover the full buyer journey
- Flag 3 quick-win content pieces (low difficulty, decent volume)

Save to `[OUTPUT-ROOT]/raw-data/voc/gtm-03-content-creative-[YYYY-MM-DD].md` (or directly under `[OUTPUT-ROOT]/` in topic-only mode).

---

## Step 4: Channel Distribution

*Goal: Decide exactly where, how, and in what order to distribute — with specific tactics and rationale from data.*

**Channel Audit:**
Evaluate each channel below using evidence from Steps 1-3:

| Channel | Evidence of Fit | Competitor Presence | Audience Layer | Priority |
|---|---|---|---|---|
| SEO / Organic Search | | | | |
| Google Ads (SEM) | | | | |
| Meta Ads (FB/IG) | | | | |
| TikTok Ads / Organic | | | | |
| YouTube | | | | |
| Reddit Organic / Ads | | | | |
| Twitter/X | | | | |
| Influencer / KOL | | | | |
| Email Marketing | | | | |
| Community / Forum | | | | |
| PR / Earned Media | | | | |
| Partnerships / BD | | | | |

**For each PRIORITIZED channel, write a channel playbook:**
- Why this channel (cite specific data: keyword volume, Reddit thread count, competitor ad spend signals, etc.)
- Target audience layer (Hot / Problem-Aware / Latent)
- Specific content format and cadence
- First 30 days tactical plan (exactly what to do week by week)
- Expected timeline to first meaningful results
- Success metric and target

**Launch Channel Priority Map:**
- **Day 1 Launch (2-3 channels):** Highest-confidence channels with fastest feedback loop
- **Month 2-3 Build (2-3 channels):** Medium-term channels requiring more lead time
- **Month 6+ Scale:** Channels to unlock once PMF is validated

**Budget Allocation Table:**

| Channel | Monthly Budget | % of Total | Primary Goal | Key Metric | Month 1 Target |
|---|---|---|---|---|---|

**90-Day GTM Roadmap:**

*Pre-Launch (Weeks 1-4):*
- Week 1: [Specific tasks]
- Week 2: [Specific tasks]
- Week 3: [Specific tasks]
- Week 4: [Specific tasks]

*Launch Week (Day-by-Day):*
- Day 1-2: [Specific tasks]
- Day 3-5: [Specific tasks]
- Day 6-7: [Specific tasks]

*Month 2 — Validate:* Goal + activities + go/no-go decision gate
*Month 3 — Optimize:* Goal + activities + go/no-go decision gate

Save to `[OUTPUT-ROOT]/raw-data/voc/gtm-04-channel-distribution-[YYYY-MM-DD].md` (or directly under `[OUTPUT-ROOT]/` in topic-only mode).

---

## Step 5: Data Collection, Feedback & Iteration

*Goal: Build the measurement and learning system that powers the flywheel — so every cycle improves the next.*

**North Star Metric:**
Define the single metric that best represents real product-market fit progress for this business. Explain why this metric over others.

**KPI Dashboard Framework:**

| Stage | Metric | Tool to Measure | Baseline | Month 1 Target | Month 3 Target |
|---|---|---|---|---|---|
| Needs Insights | Search volume trend | Semrush | | | |
| Segmentation | ICP conversion rate | CRM / Analytics | | | |
| Content | CTR / Engagement rate | Platform analytics | | | |
| Distribution | CAC per channel | Ad platform + CRM | | | |
| Flywheel health | Retention / Repeat usage | Product analytics | | | |

**Feedback Collection System:**
- **Quantitative signals**: Which metrics to monitor weekly, and what threshold triggers a strategy review
- **Qualitative signals**: Ongoing Reddit / social listening cadence; customer interview cadence (recommend 5 interviews in Month 1)
- **Competitor pulse**: Monthly check on competitor ad activity, new content, pricing changes

**Iteration Cadence:**
- **Weekly**: Review channel KPIs; pause underperforming ads; double down on top content
- **Bi-weekly**: Refresh creative based on engagement data; update audience targeting
- **Monthly**: Full flywheel review — does the needs insight still hold? Should segmentation be revised? Are new channels warranted?
- **Quarterly**: Full GTM strategy review — revisit positioning if conversion rates plateau

**A/B Testing Roadmap:**
List 5 specific tests to run in the first 90 days, each with:
- Hypothesis (drawn from a specific data point from Steps 1-4)
- What to test (headline / audience / channel / format)
- How to measure
- Decision rule (when to call a winner)

**Flywheel Loop Summary:**
Write a paragraph explaining how the data from this iteration feeds back into better needs insights for the next cycle — closing the loop.

**Strategic Risks & Assumptions:**
- Top 3 risks (evidence, likelihood, impact, mitigation)
- Top 3 assumptions to validate (with specific test method and timeline)
- The contrarian take: What would most agencies miss in this market?

Save to `[OUTPUT-ROOT]/raw-data/voc/gtm-05-data-feedback-iteration-[YYYY-MM-DD].md` (or directly under `[OUTPUT-ROOT]/` in topic-only mode).

---

## Step 6: Synthesize — GTM Flywheel Strategy Report

Compile all five stages into a single, executive-ready report. Follow the structure in `gtm-strategy-methodology.md` Section: Final Report.

Every recommendation must cite a specific data point from Steps 1-5.

Save final report to `[OUTPUT-ROOT]/plans/gtm-plan-[YYYY-MM-DD].md` (with-client) or `[OUTPUT-ROOT]/gtm-plan-[YYYY-MM-DD].md` (topic-only).

Present the strategy to the user with a brief summary of the most surprising insight from each flywheel stage.

---

## Follow-Up Prompts

After presenting the strategy, suggest relevant next steps:
- "Ready to execute? Run `/campaign-plan $ARGUMENTS` to build the first campaign."
- "Want me to `/draft-content $ARGUMENTS` for the launch content?"
- "Want me to `/email-sequence $ARGUMENTS` for the launch nurture flow?"
- "Run `/new-client $ARGUMENTS` to set up full ongoing tracking and reporting."
