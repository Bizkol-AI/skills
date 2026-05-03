# Competitor Analysis Methodology

Step-by-step process for competitor research. Used by `/competitor-scan`, `/brand-audit`, and the competitor sections of `/weekly-report` and `/monthly-report`.

## Data Collection Phase

Complete ALL steps for EACH competitor before any analysis. Work through one competitor fully before moving to the next.

### Step 1: Organic Presence (Semrush)

For each competitor domain (include subdomains where relevant):

- Authority Score
- Estimated monthly organic traffic
- Total ranking keywords
- Keywords in top 3 / 4-10 / 11-20
- Referring domains count
- Top 5 pages by organic traffic (URL, traffic estimate, top keyword)
- Top 10 ranking keywords (keyword, position, volume)

### Step 2: Head-to-Head Keyword Comparison (Semrush)

Use Semrush's keyword gap / domain comparison for the client vs each competitor:
- Common keywords count
- Keywords where competitor ranks higher than client
- Keywords where client ranks higher than competitor

Pull content gap: keywords where competitor ranks top 10 but client doesn't rank at all.

### Step 3: Backlink Intelligence (Semrush)

For each competitor:
- Referring domains count and growth trend
- New referring domains this month (count and notable sources — indicates PR activity)
- Top referring domains by Authority Score
- Compare against client's backlink profile

### Step 4: Website & Positioning (Chrome)

Visit each competitor's website and record:
- **Hero headline** (exact words)
- **Subheadline** (exact words)
- **Primary CTA** (exact words and offer)
- **Pricing** (exact figures if visible)
- **Trust signals:** review counts, press logos, certifications, guarantees, customer counts
- **Navigation structure:** main menu items
- **Blog:** exists? Last publish date? Approximate post count?
- **Email capture:** present? What lead magnet or offer?
- **Noteworthy elements:** anything distinctive about their UX, offer, or positioning

### Step 5: Paid Advertising (Chrome)

**Meta Ad Library** (facebook.com/ads/library):
- Search each competitor's brand name
- Record: total active ads, earliest active ad date
- Formats used (image / video / carousel — approximate split)
- Copy the headline and primary text of up to 5 prominent ads verbatim
- Note recurring themes, offers, pain points addressed
- Note any UGC-style creative vs branded creative

**Google Ads** (Chrome SERP):
- Search competitor's brand name — are they running branded search ads?
- Search the top 3 shared keywords — does competitor appear in paid results?
- Record any ad copy found verbatim

### Step 6: Social Presence (Bizkol MCP + Chrome)

For each competitor's social handles, pull profile + recent posts via Bizkol MCP:

- `get_social_kol_profile` — profile metrics (follower count, post count, bio) per platform (Instagram, TikTok, YouTube, X)
- `get_social_post_info` — engagement on individual recent posts
- Note content themes, formats, posting frequency, engagement levels

For platforms Bizkol doesn't cover (e.g., LinkedIn company page, Pinterest), use Chrome:
- Visit the company / brand page
- Note follower count, posting cadence, content focus

### Step 7: Review & Reputation (Chrome)

Search "[competitor name] reviews" and check:
- **Google reviews:** rating and count
- **Trustpilot / G2 / Capterra / Amazon** (whichever applies): rating and count
- Top 5 most common **positive** themes from reviews (use verbatim phrases)
- Top 5 most common **negative** themes from reviews (use verbatim phrases)
- Most helpful negative review (paste or summarize)
- Most helpful positive review (paste or summarize)

### Step 8: Reddit Sentiment (Bizkol MCP)

For each competitor, use Bizkol MCP `search_reddit` with these queries:
- "[competitor name]"
- "[competitor name] review"
- "[competitor name] problems"
- "[competitor name] vs"

For high-signal threads, drill in with `get_reddit_post` to read full comment trees. Record the most upvoted threads and comments — both positive and negative sentiment.

## Analysis Phase

After ALL competitor data is collected:

### Competitor Matrix
Build a comparison table:

| Metric | Client | Comp 1 | Comp 2 | Comp 3 |
|--------|--------|--------|--------|--------|
| Authority Score | | | | |
| Est. Monthly Traffic | | | | |
| Referring Domains | | | | |
| Primary Positioning | | | | |
| Price Point | | | | |
| Active Meta Ads | | | | |
| Social Following (primary) | | | | |
| Review Rating | | | | |
| Biggest Strength | | | | |
| Biggest Weakness | | | | |

### Positioning Map
Describe where each competitor sits relative to the client on relevant axes (e.g., premium vs budget, broad vs niche, simple vs feature-rich).

### Ad Creative Intelligence
- What hooks/angles are oversaturated across competitors?
- What pain points are competitors addressing in ads?
- What angles are NOT being used that research suggests would resonate?
- What offers/CTAs dominate? What's underused?

### Gap Analysis
Cross-reference competitor marketing claims vs what customers say in reviews and Reddit:
- Where does competitor marketing contradict customer reality?
- What customer complaints appear across multiple competitors? (universal frustration = opportunity)
- Which channels are underutilized by the competition?

### Threat Assessment
Rank competitors by overall threat level (High / Medium / Low) with rationale.

### Actionable Opportunities
List 3-5 specific things the client should do based on competitive intelligence.
