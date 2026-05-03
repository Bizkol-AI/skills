# GTM Flywheel Strategy Methodology

Full Go-To-Market research and strategy process structured as a self-reinforcing flywheel. Used by `/gtm-research`.

---

## The GTM Flywheel — How It Works

Most GTM strategies are linear: research → plan → launch → hope. The flywheel is different. It is a **closed loop where every stage feeds the next**, and data from the last stage sharpens the first. Each cycle compounds:

```
Needs Insights
        ↓
Segmentation
        ↓
Content & Creative
        ↓
Channel Distribution
        ↓
Data → Feedback → Iteration
        ↓
    [back to Needs Insights, sharper]
```

**Why this structure works:**
- Needs insights ground everything in real customer language — not assumptions
- Segmentation ensures the right message reaches the right person, not everyone
- Content is created *after* understanding the audience, so it resonates immediately
- Channel selection is evidence-based, not intuition-based
- The data/feedback loop continuously improves all four upstream stages

**Operating principle:** Complete data collection for each stage before drawing conclusions. Every recommendation must reference a specific data point. Speculation must be labeled as such.

---

## Stage 1: User Needs Insights

**Purpose:** Build a ground-truth picture of what real users want, fear, and complain about — in their own unfiltered language.

### Data Collection

**Reddit (primary voice-of-customer source):**
- Identify 5-8 subreddits where the target audience discusses the problem
- Pull top posts (last 3 months) sorted by engagement
- Scrape comments from the top 10 posts per subreddit
- Mine for: exact pain-point phrases, workarounds people use, competitor complaints, product wish-lists, trigger events ("I finally decided to look for a solution when...")
- Search Reddit for: "[category] alternatives", "[competitor] review", "[problem] help", "[problem] frustrated"

**Twitter/X (trend and sentiment signals):**
- Search category keywords and competitor brand names
- Capture verbatim complaints, success stories, and use-case discussions
- Note which posts get high engagement — these reveal what resonates

**TikTok & Instagram (format and cultural signals):**
- Search product/problem hashtags
- Study comment sections of viral problem-related videos — raw, emotional customer language
- Note which content formats (tutorial, rant, transformation, comparison) drive the most engagement

**SEO / Semrush (quantified demand signals):**
- Pull keyword overview for 5-10 seed terms
- Get matching terms and related terms — map the full demand landscape
- Flag: high CPC keywords (= commercial intent), question-format keywords (= confusion/research phase), "vs" and "alternative" keywords (= comparison shopping)
- Record monthly volume + keyword difficulty for each

**Web Search / Industry Reports:**
- "[category] market size [year]", "[category] trends [year]", "[category] statistics"
- Recent customer survey reports, VC/analyst whitepapers
- Google News: pain-point headlines from last 6 months

### Analysis Output: Needs Insights Summary
- Top 5 verbatim pain-point quotes (with source)
- Top 3 recurring frustrations (pattern across platforms)
- Top 3 desires / wish-list items
- Key trigger events that drive people to search for a solution
- Demand verdict: Proven (high volume + CPC) / Emerging / New category
- AI search landscape: Is the problem/category showing up in AI overviews?

---

## Stage 2: User Segmentation & Layering

**Purpose:** Identify who the audience actually is, split them into actionable groups, and prioritize who to go after first.

### Segment Discovery Framework

Using Stage 1 data, identify 3-5 segments using these lenses:
- **Job-to-be-done**: What ultimate outcome is each group pursuing?
- **Pain intensity**: Who is suffering most acutely right now?
- **Buying readiness**: Who is actively comparing and spending?
- **Platform presence**: Where do they live — and where are they reachable?

### Per-Segment Profile (required for each segment)
- Name and 1-line description
- Demographics + psychographics (use Reddit/Twitter evidence, not assumptions)
- Core trigger event (what sparks the search for a solution?)
- Biggest objection / fear about buying
- Current solution or workaround
- Competitor they most likely use today
- Estimated segment size and growth trajectory
- Priority tier: **ICP** / Secondary / Long-term

### Audience Layering (demand temperature)

| Layer | Description | GTM Approach |
|---|---|---|
| Layer 1 — Hot Demand | Solution-aware, actively comparing options | Direct response, comparison content, trials |
| Layer 2 — Problem-Aware | Feels the pain, hasn't found the solution yet | Educational content, problem-framing ads |
| Layer 3 — Latent Demand | Has the problem but doesn't know it's solvable | Awareness content, social proof, storytelling |

### ICP Statement
One sentence: "Our ideal first customer is [persona], who experiences [trigger event], currently uses [workaround/competitor], and will buy when [buying condition is met]."

---

## Stage 3: Content & Creative Production

**Purpose:** Translate insights and segments into actual messaging and content assets — ready to deploy.

### Competitor Content Intelligence
- Pull competitors' top-performing social posts via Bizkol MCP `get_social_kol_profile` / `get_social_post_info` (Instagram, TikTok, YouTube, X) — note hooks, formats, and CTAs
- Web search for competitors' blog content, landing page copy, case studies
- Identify what's working (high engagement) and what's missing (the whitespace)

### Positioning Architecture

**1. Whitespace Statement**
> "[TARGET] who are frustrated by [PAIN FROM DATA] want [WISH FROM DATA] but currently [GAP — no one is serving this well]."

**2. Positioning Statement**
> "For [TARGET — in their language], [BRAND] is the [CATEGORY] that [DIFFERENTIATOR] — unlike [COMPETITOR] which [WEAKNESS from data]."

**3. Messaging Hierarchy**
- Primary message: One sentence — what you do and who it's for
- 3 supporting proof points — each anchored to a specific data point from Stage 1
- 3 objection responses — using exact fears captured from Reddit/social

### Content Starter Kit

For each of the top 2 ICP segments, produce:

**Conversion Copy**
- 3 homepage hero headline options (written in the customer's own language from Stage 1)
- 5 ad hook variations — one per awareness stage (unaware / problem-aware / solution-aware / product-aware / most-aware)

**Social Content**
- 5 social media post concepts: specify platform, format (reel / carousel / thread / image), hook sentence, and body copy
- 1 short-form video script (30-45 seconds) — must open with a verbatim Reddit pain-point quote

**Email**
- 3 subject line options for launch / cold outreach

**SEO Content Plan**
- Map top 15 keywords from Stage 1 to content types: landing page / blog post / comparison page / FAQ
- Identify 3 content pillars covering the full buyer journey (awareness → consideration → decision)
- Flag 3 quick-win pieces: low KD, decent volume, high buyer intent

---

## Stage 4: Channel Distribution

**Purpose:** Decide where to distribute, in what order, with what tactics — all backed by data from Stages 1-3.

### Channel Evaluation Matrix

For each channel, score based on evidence from Stages 1-3:

| Channel | Evidence of Audience Fit | Competitor Presence | Audience Layer Match | Priority |
|---|---|---|---|---|
| SEO / Organic | | | | |
| Google Ads | | | | |
| Meta Ads | | | | |
| TikTok Organic/Ads | | | | |
| YouTube | | | | |
| Reddit | | | | |
| Twitter/X | | | | |
| Influencer / KOL | | | | |
| Email | | | | |
| Community / Forum | | | | |
| PR / Earned Media | | | | |
| Partnerships | | | | |

### Per-Channel Playbook (for each prioritized channel)
- **Why this channel**: cite specific evidence (e.g., "Reddit has 12 active subreddits with 500K+ members discussing this problem")
- **Target layer**: Hot / Problem-Aware / Latent
- **Content format & cadence**: exactly what to post and how often
- **Week-by-week 30-day plan**: specific actions, not categories
- **Time to first signal**: realistic timeline to meaningful data
- **Success metric + target**

### Launch Channel Priority Map
- **Day 1 (2-3 channels):** Channels with fastest feedback loop and highest confidence — prioritize channels where the audience already congregates
- **Month 2-3 (2-3 channels):** Channels with longer build time (SEO, community, influencer)
- **Month 6+ (remaining):** Unlock once PMF signals are confirmed

### Budget Allocation Table
| Channel | Monthly Budget | % of Total | Primary Goal | Key Metric | Month 1 Target |
|---|---|---|---|---|---|

### 90-Day GTM Roadmap

**Pre-Launch (Weeks 1-4)**
Week-by-week checklist of specific tasks (not generic categories).

**Launch Week**
Day-by-day playbook.

**Month 2 — Validate**
North star goal + specific activities + go/no-go decision gate.

**Month 3 — Optimize**
North star goal + specific activities + go/no-go decision gate.

---

## Stage 5: Data Collection, Feedback & Iteration

**Purpose:** Build the measurement and learning system that powers the flywheel. Data from this stage directly improves Stage 1 of the next cycle.

### North Star Metric
Define the single metric that best represents real PMF progress. Explain why this over other candidates.

### KPI Framework

| Flywheel Stage | Metric | Measurement Tool | Baseline | Month 1 Target | Month 3 Target |
|---|---|---|---|---|---|
| Needs Insights | Search volume trend | Semrush | | | |
| Segmentation | ICP conversion rate | CRM + Analytics | | | |
| Content | CTR / Engagement rate | Platform analytics | | | |
| Distribution | CAC per channel | Ad platform + CRM | | | |
| Flywheel Health | Retention / Repeat usage | Product analytics | | | |

### Feedback Collection System

**Quantitative signals** — monitored weekly:
- Which metrics to track and in which tool
- Threshold that triggers a strategy review (e.g., "if CAC exceeds 3x LTV, pause paid and review targeting")

**Qualitative signals** — ongoing:
- Reddit / social listening cadence (weekly keyword searches, monthly subreddit reviews)
- Customer interview cadence: 5 interviews in Month 1, 3/month ongoing
- Win/loss interview template: what made them buy, what almost stopped them, what they'd change

**Competitor pulse** — monthly:
- Competitor ad activity (new creatives, new channels)
- New content or product announcements
- Pricing changes

### Iteration Cadence

| Frequency | Review Focus | Action If Off-Track |
|---|---|---|
| Weekly | Channel KPIs, ad performance | Pause underperformers, double down on winners |
| Bi-weekly | Creative performance, audience targeting | Refresh creatives, adjust targeting |
| Monthly | Full flywheel review | Revise segmentation or messaging if conversion plateaus |
| Quarterly | GTM strategy review | Revisit positioning, ICP, or channel mix |

### A/B Testing Roadmap
List 5 specific tests for the first 90 days. For each:
- Hypothesis (from a specific Stage 1-4 data point)
- Variable to test (headline / audience / channel / format)
- Measurement method
- Decision rule (minimum sample size + metric threshold to call a winner)

### Flywheel Loop Summary
Write a paragraph explaining specifically how data from this iteration improves Stage 1 of the next cycle — what new insights will be sharper, what assumptions will be replaced with facts.

### Strategic Risks & Final Take
- Top 3 risks: evidence, likelihood (H/M/L), impact (H/M/L), mitigation plan
- Top 3 assumptions to validate: specific test method + timeline
- The contrarian take: what would most agencies miss or get wrong in this specific market?

---

## Final Report Structure

The GTM Flywheel Strategy Report is one executive-ready document containing:

1. **Methodology Overview** — Explain the flywheel approach and why it applies to this client
2. **Stage 1 Summary** — Top needs insights with supporting evidence
3. **Stage 2 Summary** — Segment profiles + ICP statement
4. **Stage 3 Summary** — Positioning, messaging hierarchy, content starter kit
5. **Stage 4 Summary** — Channel priority map, budget table, 90-day roadmap
6. **Stage 5 Summary** — KPI framework, iteration cadence, A/B testing plan, flywheel loop summary
7. **Executive One-Pager** — Market opportunity, customer problem (in their words), competitive whitespace, positioning statement, top 3 launch channels, 90-day north star goal, biggest strategic bet, confidence level

Every recommendation must cite a specific data point. Label any speculation clearly.
