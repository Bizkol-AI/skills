---
description: Generate a full campaign brief with objectives, channels, content calendar, and success metrics for a client
argument-hint: "[client-name] <campaign objective>"
---

> If you see unfamiliar placeholders or need to check which tools are connected, see [CONNECTORS.md](../CONNECTORS.md).

Read the marketing-framework skill and these references before starting:
- `${CLAUDE_PLUGIN_ROOT}/skills/paid-media/references/paid-media-framework.md`
- `${CLAUDE_PLUGIN_ROOT}/skills/content-brand/references/content-creation-framework.md`

Generate a comprehensive marketing campaign brief for client "$ARGUMENTS".

## Step 0: Prerequisite — Client Folder

Before running, the client folder must exist at `/clients/$ARGUMENTS/` with `client-brief.md`. If not:

> "No client folder found for $ARGUMENTS. Run `/new-client $ARGUMENTS` first to create the brief and folder structure, then re-run this command."

Stop here if the folder is missing.

## Step 1: Load Client Context

Read `/clients/$ARGUMENTS/client-brief.md`. Extract:
- Business type and target customer
- Active marketing channels
- Budget range
- Brand voice and positioning
- Competitors
- Current performance baselines

If the client brief is not found, use AskUserQuestion to gather essentials:
1. What is the campaign goal? (drive signups, increase awareness, launch a product, generate leads, re-engage churned users)
2. Who is the target audience?
3. What is the timeline and any fixed dates?
4. What is the approximate budget?

## Step 2: Competitive Context

**If Semrush MCP is connected:**
- Pull top keywords related to the campaign topic
- Check what competitors are ranking for in this space
- Note content gap opportunities relevant to the campaign

**If Chrome browser is available:**
- Check Meta Ad Library for competitor campaigns on similar themes
- Note messaging angles and creative formats being used

**If no tools are connected:**
- Use any existing competitor data from the client's raw-data folder
- Note that competitive context is limited and recommend running `/competitor-scan` for deeper intelligence

## Step 3: Generate Campaign Brief

### 1. Campaign Overview
- Campaign name suggestion
- One-sentence campaign summary
- Primary objective with a specific, measurable goal
- Secondary objectives (if applicable)

### 2. Target Audience
- Primary audience segment with description
- Audience pain points and motivations (reference Reddit research if available)
- Where they spend time (channels, communities, publications)
- Buying stage alignment (awareness, consideration, decision)

### 3. Key Messages
- Core campaign message (one sentence)
- 3-4 supporting messages tailored to audience pain points
- Message variations by channel (if different tones are needed)
- Proof points or evidence to support each message
- Use customer language from Reddit research if available

### 4. Channel Strategy
Recommend channels based on audience, goal, and client's active channels. For each channel:
- Why this channel fits the audience and objective
- Content format recommendations
- Estimated effort level (low, medium, high)
- Budget allocation suggestion (reference paid-media-framework.md for allocation guidance)

Use the budget distribution framework from paid-media-framework.md:
- New clients (< 6 months): weight toward high-intent, fast-feedback channels
- Mature clients (6+ months): shift toward retention and brand building

### 5. Content Calendar
Create a week-by-week content calendar:

| Week | Content Piece | Channel | Format | Owner/Notes | Status |
|------|--------------|---------|--------|-------------|--------|

Include dependencies between pieces (e.g., "landing page must be live before paid ads launch").

### 6. Content Pieces Needed
List every content asset required:
- Asset name and type (blog post, email, social post, ad creative, landing page, etc.)
- Brief description
- Priority (must-have vs. nice-to-have)
- Suggested timeline for creation

### 7. Creative Strategy
Reference the paid-media-framework.md creative strategy section:
- Content types by funnel stage (TOF, MOF, BOF)
- UGC vs branded creative recommendations
- Platform-native creative guidelines (formats, aspect ratios, lengths)

### 8. Success Metrics
- Primary KPI with target number
- Secondary KPIs (3-5)
- How each metric will be tracked
- Reporting cadence recommendation
- Reference performance benchmarks from performance-benchmarks.md

### 9. Risks and Mitigations
- 2-3 potential risks (timeline, audience mismatch, channel underperformance)
- Mitigation strategy for each

### 10. Next Steps
- Immediate action items to kick off the campaign
- Decision points and approvals needed

## Step 4: Save Campaign Brief

Save to: `/clients/$ARGUMENTS/plans/campaign-plan-[YYYY-MM-DD].md`

Present the brief to the user.

## Follow-Up

Ask: "Would you like me to:
- Dive deeper into any section of this campaign plan?
- Draft specific content pieces from the calendar using `/draft-content`?
- Run a competitor analysis to sharpen the messaging using `/competitor-scan`?
- Design an email sequence for this campaign using `/email-sequence`?
- Adjust the plan for a different budget or timeline?"
