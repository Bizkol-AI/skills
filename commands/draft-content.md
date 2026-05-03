---
description: Draft marketing content — blog posts, social media, email newsletters, landing pages, press releases, and case studies for a client
argument-hint: "[client-name] <content type and topic>"
---

> If you see unfamiliar placeholders or need to check which tools are connected, see [CONNECTORS.md](../CONNECTORS.md).

Read the marketing-framework skill and `${CLAUDE_PLUGIN_ROOT}/skills/content-brand/references/content-creation-framework.md` before starting.

Generate marketing content for client "$ARGUMENTS".

## Step 0: Prerequisite — Client Folder (recommended)

Strongly recommended: a populated `/clients/[client-name]/client-brief.md` so brand voice, positioning, and ICP are applied. If missing, warn:

> "No client folder found for [name]. The draft will be generic without brand voice and ICP context. For voice-consistent output, run `/new-client [name]` first."

The user may proceed without — fall through to the AskUserQuestion path below.

## Step 1: Load Client Context

Read `/clients/[client-name]/client-brief.md` if it exists. Extract:
- Brand voice and tone preferences
- Target customer description
- Product/service details
- Positioning and differentiators
- Competitors (to ensure differentiation)

If the client brief is not found, gather inputs using AskUserQuestion:
1. What content type? (Blog post, Social media post, Email newsletter, Landing page, Press release, Case study)
2. What is the topic?
3. Who is the target audience?
4. What are the key messages (2-4 main points)?
5. What tone? (authoritative, conversational, inspirational, technical, witty)
6. What length? (target word count or format constraint)

## Step 2: Research Context (if tools available)

**If Semrush MCP is connected (for web content):**
- Pull keyword data for the topic
- Identify primary keyword and 2-3 secondary keywords
- Check what's ranking on page 1 for the primary keyword
- Note content angles that are working

**If Bizkol MCP is available (Reddit tools) or the Reddit scraper:**
- Quick search for the topic to find customer language and common questions
- Pull verbatim phrases to use in copy

**If no tools are connected:**
- Use existing client research data from raw-data folder if available
- Proceed with content creation based on provided inputs

## Step 3: Generate Content

Follow the content-creation-framework.md templates for the specified content type.

### Blog Post
- 2-3 headline options (benefit-driven, includes primary keyword, under 60 chars)
- Introduction with a hook (question, statistic, bold statement, or story)
- 3-5 organized sections with descriptive subheadings (H2)
- Supporting points, examples, or data in each section
- Conclusion with a clear call to action
- Meta description (under 160 chars)
- SEO: primary keyword in headline, first paragraph, one subheading

### Social Media Post
- Platform-appropriate format and length
- Hook in the first line
- 3-5 relevant hashtags
- Call to action or engagement prompt
- Adapt per platform: LinkedIn (professional, paragraph breaks), Twitter/X (punchy, concise), Instagram (visual-first, story-driven), Facebook (conversational, question-driven)

### Email Newsletter
- 2-3 subject line options (under 50 chars)
- Preview text (complementing, not repeating subject)
- Body sections with clear hierarchy
- One primary CTA per email
- Sign-off and unsubscribe reminder

### Landing Page Copy
- Headline and subheadline
- Hero section copy
- 3-4 value propositions (benefit-driven)
- Social proof placement suggestions
- Primary and secondary CTAs
- FAQ section suggestions
- Meta title and description

### Press Release
- Headline following press release conventions
- Dateline, lead paragraph (who, what, when, where, why)
- Supporting quotes placeholder
- Company boilerplate placeholder
- Media contact placeholder

### Case Study
- Title: "[Customer] achieves [result] with [product]"
- Customer overview, challenge, solution, results
- Customer quote placeholder
- Call to action

## Step 4: Apply Brand Voice

- If brand voice is defined in client brief, apply it consistently
- Note which voice attributes were applied
- If no brand voice is configured, use a professional tone and note this

## Step 5: Save and Present

Save to: `/clients/[client-name]/content/drafts/[content-type]-[topic-slug]-[YYYY-MM-DD].md` (with-client) or `/sessions/draft-[topic-slug]-[YYYY-MM-DD]/[content-type].md` (no client).

Present the draft with:
- A note on what brand voice and tone were applied
- SEO recommendations (for web content)
- Suggestions for next steps

## Follow-Up

Ask: "Would you like me to:
- Revise any section or adjust the tone?
- Create a variation for a different channel?
- Run a brand voice review using `/brand-review`?
- Create more content pieces for a campaign?
- Research the topic further using `/reddit-research`?"
