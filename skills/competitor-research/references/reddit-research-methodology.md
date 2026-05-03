# Reddit & Social Voice-of-Customer Research Methodology

Step-by-step process for deep Reddit and social discussion research. Used by `/reddit-research`, `/gtm-research`, and feeds into competitor and market analysis.

## Purpose

Reddit is one of the most valuable sources of unfiltered customer voice — real people describing real problems in their own words, without marketing language. The goal is to collect exact language real people use so it can inform positioning, messaging, ad copy, and content strategy.

## Data Collection Phase

Do NOT summarize or paraphrase during collection. Copy content verbatim. Analysis comes later.

### Step 1: Subreddit Discovery (Bizkol MCP `search_reddit` + `get_reddit_subreddit`)

Use `search_reddit` to find threads, then `get_reddit_subreddit` for details on candidate subreddits. Search queries:
- [primary product/service type]
- [problem the business solves]
- [industry/category name]
- [top competitor name 1]
- [top competitor name 2]

Record every relevant subreddit found:

| Subreddit | Subscribers | Topic Focus | Relevance (High/Med/Low) |
|-----------|------------|-------------|--------------------------|

### Step 2: Discussion Scraping (Bizkol MCP `search_reddit` + `get_reddit_post`)

Run each of these searches and collect the top 10 posts + top 3 comments per post:

1. "[product/service type] recommendation"
2. "[product/service type] advice"
3. "best [product/service type]"
4. "[problem it solves] help"
5. "[product/service type] alternative"
6. "[product/service type] worth it"
7. "[competitor 1] review"
8. "[competitor 2] review"
9. "[competitor 1] problems"
10. "frustrated [category/problem]"
11. "wish there was [product/service type]"
12. "why is [category] so [expensive/bad/complicated]"

For each post, record:
- Subreddit
- Title
- Upvote count
- Top 3 comments (verbatim — do not paraphrase)

### Step 3: Competitor Reddit Research (Bizkol MCP `search_reddit`)

For each top competitor:
- "[competitor name]"
- "[competitor name] review"
- "[competitor name] problems"
- "cancel [competitor name]" or "leaving [competitor name]"
- "[competitor name] vs"

Record the most upvoted threads and comments verbatim — both positive and negative.

### Step 4: Problem & Frustration Threads (Bizkol MCP `search_reddit`)

These threads are gold for positioning and messaging. Search:
- "frustrated with [category]"
- "why is [category] so [expensive/complicated/etc]"
- "wish there was a [product/service type] that"
- "looking for [product/service type] that actually"
- "[category] sucks" or "[category] terrible"

Record the most upvoted complaints and wishes verbatim.

### Step 5: Other Social Platforms (Bizkol MCP + Chrome)

**YouTube comments** (Chrome):
- Search "[product/service type] review" on YouTube
- Pull comments from the 2-3 most viewed videos
- Record the 10 most upvoted comments per video

**Twitter/X** (Bizkol MCP `search_social_kols` for relevant accounts; Chrome for keyword search):
- Search "[product/service type]" and "[problem it solves]" via Chrome → x.com search
- Look for complaint threads and recommendation threads
- Record notable tweets with high engagement

**Quora** (Chrome):
- Search "[problem this solves]" on Quora
- Record the top 3 questions and most upvoted answers

**Facebook Groups** (Chrome):
- Search "[industry/category] group" on Facebook
- Note what groups exist and their size
- If publicly visible, note common discussion themes

### Step 6: Amazon/Marketplace Reviews (Chrome — if physical product)

Search Amazon for the closest equivalent product category:
- Record the top 3 best-selling products
- Read and collect the top 10 reviews for each
- Focus on 3-star reviews (most balanced and honest)
- Record verbatim the most common praise and complaints

## Analysis Phase

After ALL raw discussion data is collected:

### Voice of Customer Synthesis

**The Real Problem (in customer language):**
Pull the most recurring phrases people use to describe their pain. Minimum 10 verbatim quotes with source:
- "[exact quote]" — Reddit, r/[subreddit], [upvote count] upvotes
- "[exact quote]" — Amazon review, [product], [star rating]

**What They've Already Tried (and why it failed):**
List every solution/competitor mentioned with the exact complaint about each.

**What They Wish Existed:**
Verbatim "I wish..." or "Why doesn't someone just..." statements. Minimum 5 quotes.

**Purchase Trigger:**
Based on post context — what situation prompted them to seek a solution?

**Objections & Fears:**
What hesitations appear repeatedly?

**Where They Spend Time Online:**
Which subreddits, communities, platforms, content types are most active?

### Customer Language Glossary
List the specific words and phrases this audience uses. These should appear in the brand's copy, ads, and content.

### Reddit-Specific Strategic Insight
- What does the community actually want that nobody is giving them?
- What would make a brand genuinely loved in these communities?
- Is there a community-led growth strategy here?
- Should the brand be active on Reddit? Which subreddits? What type of contribution?
