---
description: Design and draft multi-email sequences for nurture flows, onboarding, drip campaigns, and more for a client
argument-hint: "[client-name] <sequence type>"
---

> If you see unfamiliar placeholders or need to check which tools are connected, see [CONNECTORS.md](../CONNECTORS.md).

Read the marketing-framework skill and `${CLAUDE_PLUGIN_ROOT}/skills/content-brand/references/content-creation-framework.md` before starting.

Design and draft a complete email sequence for client "$ARGUMENTS".

## Step 0: Prerequisite — Client Folder (recommended)

Strongly recommended: a populated `/clients/[client-name]/client-brief.md`. Without it the sequence won't reflect brand voice or ICP. If missing, warn:

> "No client folder found for [name]. The sequence will be generic without brand voice. For voice-consistent output, run `/new-client [name]` first."

The user may proceed without — fall through to AskUserQuestion below.

## Step 1: Load Client Context

Read `/clients/[client-name]/client-brief.md` if it exists. Extract:
- Brand voice and tone preferences
- Target customer description
- Product/service details
- Email platform used (if noted)
- Current email metrics (if available)

If the client brief is not found, gather inputs using AskUserQuestion:
1. What sequence type? (Onboarding, Lead nurture, Re-engagement, Product launch, Event follow-up, Upgrade/upsell, Win-back, Educational drip)
2. What is the goal? (activate new users, convert leads, reduce churn, etc.)
3. Who is the audience? (stage, segmentation details)
4. How many emails? (or let me recommend based on type)
5. What timing/cadence? (every 3 days, weekly, aggressive then taper)
6. Any specific offers, incentives, or content assets to include?

## Step 2: Sequence Strategy

Before drafting any emails, define:

- **Narrative arc** — what story does this sequence tell? What is the emotional and logical progression?
- **Journey mapping** — map each email to a stage (awareness, consideration, decision, activation, expansion)
- **Escalation logic** — how does intensity/urgency/value build?
- **Success definition** — what action signals the sequence has done its job?

### Sequence Type Templates

Use these as starting frameworks:

**Onboarding (5-7 emails, 14-21 days):**
Welcome and set expectations → Quick win to demonstrate value → Core feature deep dive → Advanced feature/integration → Social proof and community → Check-in and feedback → Upgrade prompt

**Lead Nurture (4-6 emails, 3-4 weeks):**
Value-first educational content → Pain point identification → Solution positioning with proof → Social proof and results → Soft CTA → Direct CTA

**Re-engagement (3-4 emails, 10-14 days):**
"We miss you" with compelling reason → Value reminder → Incentive/exclusive offer → Last chance with deadline

**Win-back (3-5 emails, 30 days):**
Friendly check-in → What's new since they left → Special offer → Feedback request → Final goodbye with door open

**Product Launch (4-6 emails, 2-3 weeks):**
Teaser/pre-announcement → Launch announcement → Feature spotlight → Social proof/early results → Limited-time offer → Reminder

**Educational Drip (5-8 emails, 4-6 weeks):**
Introduction → Lesson 1 (foundational) → Lesson 2 (intermediate) → Lesson 3 (advanced) → Practical application → Resource roundup → Graduation/next steps

## Step 3: Draft Individual Emails

For each email in the sequence, produce:

### Subject Line
- 2-3 options per email
- Vary approaches: curiosity, benefit-driven, urgency, personalization, question-based
- Under 50 characters where possible

### Preview Text
- 40-90 characters complementing (not repeating) the subject line

### Email Purpose
- One sentence: why this email exists and what it moves the recipient toward

### Body Copy
- Full draft ready to use
- Clear hierarchy: hook, body, CTA
- Short paragraphs (2-3 sentences max)
- Personalization tokens where relevant
- Apply client's brand voice

### Primary CTA
- Button text and destination
- One primary CTA per email

### Timing
- Days after trigger or previous email
- Notes on engagement-based timing adjustments

### Segment/Condition Notes
- Who receives vs. who skips
- Behavioral or attribute-based conditions

## Step 4: Sequence Logic

Define:
- **Branching conditions** — alternate paths based on engagement
- **Exit conditions** — when a recipient converts, remove from sequence
- **Re-entry rules** — can someone re-enter? Under what conditions?
- **Suppression rules** — do not send if in another sequence, unsubscribed, or contacted support recently

### Sequence Flow Diagram

```
[Trigger] --> Email 1 (Day 0)
                |
          Opened? --Yes--> Email 2 (Day 3)
                |              |
                No        Clicked CTA? --Yes--> [EXIT: Converted]
                |              |
                v              No
          Email 1b (Day 2)     |
                |              v
                +--------> Email 3 (Day 7)
```

## Step 5: Performance Benchmarks

| Metric | Onboarding | Lead Nurture | Re-engagement | Win-back |
|--------|-----------|--------------|---------------|----------|
| Open rate | 50-70% | 20-30% | 15-25% | 15-20% |
| Click-through rate | 10-20% | 3-7% | 2-5% | 2-4% |
| Conversion rate | 15-30% | 2-5% | 3-8% | 1-3% |
| Unsubscribe rate | <0.5% | <0.5% | 1-2% | 1-3% |

## Step 6: Tool Integration

### If ~~email marketing is connected (e.g., Klaviyo, Mailchimp)
- Reference how to set up the sequence as a flow/automation in the platform
- Map branching logic to the platform's visual flow builder
- Note platform-specific features (smart send time, conditional splits, A/B testing)

### If ~~marketing automation is connected (e.g., HubSpot)
- Reference lead scoring data to inform segmentation and exit conditions
- Use lifecycle stage data to tailor messaging

### If no tools are connected
- Deliver all email content in copy-paste-ready format
- Include a setup checklist for any email platform:
  1. Create the automation/flow
  2. Set the enrollment trigger
  3. Add each email with specified delays
  4. Configure branching and exit conditions
  5. Set up tracking for recommended metrics

## Step 7: Save and Present

Save to: `/clients/[client-name]/content/sequences/email-sequence-[type]-[YYYY-MM-DD].md` (with-client) or `/sessions/email-sequence-[type]-[YYYY-MM-DD]/sequence.md` (no client).

Present with:
- Sequence overview table
- Full email drafts
- Flow diagram
- Branching logic notes
- A/B test suggestions (2-3 recommended tests)
- Metrics to track

## Follow-Up

Ask: "Would you like me to:
- Revise the copy or tone for any specific email?
- Add a branching path for a specific scenario?
- Create a variation for a different audience segment?
- Draft the A/B test variants for the subject lines?
- Build a companion sequence (e.g., post-purchase follow-up)?
- Review the copy against brand voice using `/brand-review`?"
