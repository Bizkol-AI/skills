# SEO & GEO Audit Methodology

Step-by-step process for conducting a full SEO and GEO audit. Used by `/seo-audit`, `/brand-audit`, and the SEO section of `/monthly-report`.

## Data Collection Phase

Complete ALL steps before any analysis.

### Step 1: Domain Overview (Semrush)

Pull from Semrush for the client domain (include subdomains where relevant):

- Authority Score (AS)
- Estimated monthly organic traffic
- Total ranking keywords
- Referring domains count
- Backlinks count

Save raw data before proceeding.

### Step 2: Keyword Rankings (Semrush)

Pull organic keywords for the client domain:

- All keywords ranking in top 100
- Select: keyword, position, volume, traffic estimate, URL, keyword difficulty
- Order by traffic descending
- Note the top 20 keywords driving the most traffic

If the client has a Semrush project with tracked keywords (Position Tracking), also pull rank tracker data for the current date compared to the previous period.

### Step 3: Keyword Ranking Distribution (Semrush)

Pull keywords history to get distribution:
- Keywords in positions 1-3
- Keywords in positions 4-10
- Keywords in positions 11-20
- Keywords in positions 21-50
- Keywords in positions 51-100

Compare against previous month if running monthly report.

### Step 4: Backlink Profile (Semrush)

Pull referring domains:
- Total count
- New referring domains this month
- Lost referring domains this month
- Top referring domains by DR

Pull backlinks stats for current date.

### Step 5: Content Gap Analysis (Semrush)

For each competitor URL in the client brief, run organic keywords comparison:
- Find keywords where competitors rank in top 10 but client does NOT rank in top 100
- Filter for keywords with volume > 200 and Keyword Difficulty < 40 (realistic opportunities)
- Order by volume descending
- Present top 20 content gap keywords

### Step 6: Top Pages Analysis (Semrush)

Pull top pages by organic traffic for the client domain:
- URL, organic traffic, number of keywords, top keyword, position for top keyword
- Top 20 pages

Also pull for each competitor to compare content strategy.

### Step 7: Technical SEO Spot-Check (Chrome)

Visit the client's website with Chrome and check:
- Homepage load time (visual assessment)
- Mobile responsiveness (resize viewport)
- Schema markup presence (view page source or use structured data testing)
- Robots.txt accessibility
- Sitemap.xml existence
- HTTPS enforcement
- Core navigation structure
- Internal linking from homepage
- Blog/content section existence and recency

### Step 8: SERP & GEO Analysis (Chrome)

For the client's top 5 target keywords:
- Search each in Google
- Note: client's position (if visible on page 1), AI Overview presence, featured snippets, People Also Ask questions, number of paid ads
- If AI Overview appears, note which sources are cited and whether the client appears
- Search the same keywords in Perplexity.ai and note which brands are recommended

### Step 9: Local SEO (if local business — Chrome)

- Search "[business type] near [location]" and note local pack results
- Check Google Business Profile: rating, review count, completeness
- Search "[business name]" and note what appears in the knowledge panel
- Check NAP consistency (Name, Address, Phone) across top directories

## Analysis Phase

After ALL data is collected, produce the analysis:

### SEO Health Assessment
- Overall health score: Strong / Moderate / Needs Attention / Critical
- Evidence for the score (cite specific metrics)
- Domain authority trajectory (improving / stable / declining)

### Quick Wins (executable in 30 days)
- Keywords in positions 4-15 with high volume — optimize existing pages
- Pages with high impressions but low CTR — rewrite title tags and meta descriptions
- Internal linking gaps — key pages not linked from navigation or content
- Missing schema markup — add appropriate structured data

### Content Priorities (next 90 days)
- Top 10 content pieces to create based on content gap analysis
- For each: target keyword, search volume, KD, search intent, suggested content format
- Topic cluster opportunities

### GEO Readiness
- Current AI search visibility (from SERP analysis)
- Content structure assessment: does content answer questions in ways AI can excerpt?
- Brand mention authority: how many external sources reference the brand?
- Specific recommendations for improving GEO visibility

### Technical Issues
- List any issues found during spot-check with severity and fix recommendation
