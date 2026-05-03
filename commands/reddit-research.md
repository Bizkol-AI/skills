---
description: Run deep Reddit and social voice-of-customer research for a client or topic
argument-hint: [client-name or topic]
allowed-tools: Read, Write, Glob, Grep, WebSearch, TodoWrite, Task
---

Read the marketing-framework skill and `${CLAUDE_PLUGIN_ROOT}/skills/competitor-research/references/reddit-research-methodology.md` before starting.

> **Connector reference:** See [CONNECTORS.md](../CONNECTORS.md). Primary: **Bizkol MCP** (`search_reddit`, `get_reddit_post`, `get_reddit_subreddit`, `get_reddit_user`) — falls back to standalone Reddit, Twitter, YouTube scrapers for breadth.

Run deep voice-of-customer research via Reddit and social channels for "$ARGUMENTS".

## Step 0 / Step 1: Determine Research Context

Check if `/clients/$ARGUMENTS/client-brief.md` exists.

**If client brief exists (recommended):** Extract product/service description, target customer, competitors, and industry context to guide research queries. Outputs land under `/clients/$ARGUMENTS/`.

**If no client folder (topic-only mode allowed):** Treat "$ARGUMENTS" as a topic or product category. Warn the user:

> "No client folder found for $ARGUMENTS. Running in topic-only mode — outputs will land in `/sessions/[topic-slug]-[YYYY-MM-DD]/` and brand-fit context will be limited. For richer research, run `/new-client $ARGUMENTS` first."

In topic-only mode, ask via AskUserQuestion:
1. What product/service/topic are you researching?
2. Who is the target customer?
3. Any specific questions you want answered?

For the rest of this command, `[OUTPUT-ROOT]` = `/clients/$ARGUMENTS` (with-client) or `/sessions/[topic-slug]-[YYYY-MM-DD]` (topic-only).

## Step 2: Subreddit Discovery

**Reddit scraper:**
- Search for subreddits related to the client's industry, product category, and target audience
- Identify 3-5 most relevant and active subreddits
- Note subscriber counts and posting activity levels

Save to `[OUTPUT-ROOT]/raw-data/voc/reddit-subreddits-[YYYY-MM-DD].md` (with-client) or `[OUTPUT-ROOT]/reddit-subreddits-[YYYY-MM-DD].md` (topic-only).

## Step 3: Problem and Frustration Mining

**Reddit scraper — Search across identified subreddits for:**
- "[product category] problems"
- "[product category] frustrating"
- "[product category] hate"
- "[product category] wish"
- "[product category] recommend"
- "[product category] alternative to"
- "[product category] vs"
- "[product category] review"
- "[product category] worth it"
- "best [product category]"
- "help me choose [product category]"
- "[product category] for beginners"

For each search, pull top 10 posts. For the most relevant posts, pull top 3 comments.

**Capture verbatim quotes** — exact customer language is the gold. Record:
- The exact words people use to describe their problems
- How they describe what they want
- Objections and concerns they raise
- Decision criteria they mention
- Emotional language and intensity

Save all raw posts and comments to `[OUTPUT-ROOT]/raw-data/voc/reddit-discussions-[YYYY-MM-DD].md` (with-client) or `[OUTPUT-ROOT]/reddit-discussions-[YYYY-MM-DD].md` (topic-only).

## Step 4: Competitor Sentiment

**Reddit scraper — For each known competitor:**
- Search "[competitor name]" across relevant subreddits
- Search "[competitor name] review"
- Search "[competitor name] alternative"

Record:
- What people love about competitors
- What people complain about
- Common reasons for switching
- Feature requests and wishlists

Save to `[OUTPUT-ROOT]/raw-data/voc/reddit-competitor-sentiment-[YYYY-MM-DD].md` (or directly under `[OUTPUT-ROOT]/` in topic-only mode).

## Step 5: Broader Social Listening

**Twitter scraper:**
- Search for the product category and competitor names
- Note trending conversations, complaints, and praise

**YouTube scraper:**
- Search for "[product category] review" and "[product category] comparison"
- Note top videos, view counts, and comment themes

Save to `[OUTPUT-ROOT]/raw-data/voc/social-listening-[YYYY-MM-DD].md` (or directly under `[OUTPUT-ROOT]/` in topic-only mode).

## Step 6: Generate Voice of Customer Report

Analyze all collected data and compile:

1. **Customer Language Glossary** — The exact words and phrases real customers use (for ad copy, landing pages, SEO)
2. **Pain Point Hierarchy** — Top 5 problems ranked by frequency and emotional intensity, with verbatim quotes
3. **Desire Mapping** — What customers actually want (often different from what they say they want)
4. **Decision Criteria** — What factors drive purchase decisions, in order of importance
5. **Objection Bank** — Common concerns and hesitations, with suggested responses
6. **Competitor Perception** — How customers view each competitor (strengths and weaknesses from their perspective)
7. **Content Opportunities** — Questions people ask that could become blog posts, videos, or ads
8. **Messaging Recommendations** — Specific headline and copy suggestions using actual customer language
9. **Subreddit Engagement Opportunities** — Where and how the client could authentically participate

Every insight must include at least one verbatim quote from the research.

Save to `[OUTPUT-ROOT]/audits/voc-research-[topic-slug]-[YYYY-MM-DD].md` (with-client) or `[OUTPUT-ROOT]/voc-research-[topic-slug]-[YYYY-MM-DD].md` (topic-only).

Present the report to the user.

## Follow-Up Prompts

After presenting the report, suggest relevant next steps:
- "Want me to `/draft-content $ARGUMENTS` using the customer language we discovered?"
- "Want me to `/email-sequence $ARGUMENTS` targeting the top pain points from the research?"
- "Want me to `/brand-review` to check if your current messaging aligns with how customers talk?"
