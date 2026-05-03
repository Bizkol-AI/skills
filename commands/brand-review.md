---
description: Review content against a client's brand voice, style guide, and messaging pillars
argument-hint: "[client-name] <content to review>"
---

> If you see unfamiliar placeholders or need to check which tools are connected, see [CONNECTORS.md](../CONNECTORS.md).

Read the marketing-framework skill and `${CLAUDE_PLUGIN_ROOT}/skills/content-brand/references/brand-voice-framework.md` before starting.

Review marketing content against brand voice and messaging standards for client "$ARGUMENTS".

## Step 0: Prerequisite — Client Folder (recommended)

Strongly recommended: a populated `/clients/[client-name]/client-brief.md` with brand voice attributes, terminology, and messaging pillars. Without it, the review falls back to a generic clarity / consistency / professionalism check.

If missing and the user wants a brand-specific review:

> "No client folder found for [name]. For a brand-specific review, run `/new-client [name]` first to document the voice. Otherwise I'll run a generic clarity/consistency check — proceed?"

## Step 1: Load Client Context

Read `/clients/[client-name]/client-brief.md` if it exists. Extract:
- Brand voice attributes (tone, personality, voice descriptors)
- Tagline and positioning
- Target customer description
- Key messaging pillars
- Any style guide preferences

If the client brief is not found, ask:
"Do you have a brand style guide or voice guidelines I should review against? You can paste them, share a file, or describe your brand voice. Otherwise, I'll do a general review for clarity, consistency, and professionalism."

## Step 2: Receive Content

Accept content in any form:
- Pasted directly into the conversation
- A file path
- A URL to a published page (use Chrome to visit)
- Multiple pieces for batch review

## Step 3: Review Process

### With Brand Guidelines Configured

Evaluate against each dimension:

#### Voice and Tone
- Does the content match the defined brand voice attributes?
- Is the tone appropriate for the content type and audience?
- Are there inconsistent shifts in voice?
- Flag specific sentences that deviate with explanation

#### Terminology and Language
- Are preferred brand terms used correctly?
- Are any "avoid" terms present?
- Is jargon level appropriate for the target audience?
- Are product names and branded terms used correctly?

#### Messaging Pillars
- Does the content align with defined messaging pillars?
- Are claims consistent with approved messaging?
- Is the content reinforcing or contradicting brand positioning?

#### Style Guide Compliance
- Grammar and punctuation per style guide (Oxford comma, case conventions)
- Formatting conventions (headers, lists, emphasis)
- Number and date formatting
- Acronym usage (defined on first use?)

### Without Brand Guidelines (Generic Review)

#### Clarity
- Is the main message clear within the first paragraph?
- Are sentences concise and easy to understand?
- Is the structure logical?

#### Consistency
- Is the tone consistent throughout?
- Are terms used consistently?
- Is formatting consistent?

#### Professionalism
- Free of typos, grammatical errors, awkward phrasing?
- Tone appropriate for the intended audience?
- Claims supported or substantiated?

### Legal and Compliance Flags (Always Checked)

Regardless of brand guidelines, flag:
- **Unsubstantiated claims** — superlatives ("best", "fastest", "only") without evidence
- **Missing disclaimers** — financial, health, or guarantee claims that may need legal review
- **Comparative claims** — comparisons to competitors that could be challenged
- **Regulatory language** — content that may need compliance review
- **Testimonial issues** — quotes without attribution or disclosure
- **Copyright concerns** — closely paraphrased content from other sources

## Step 4: Generate Review Report

### Summary
- Overall assessment: how well the content aligns with brand standards
- 1-2 sentence summary of biggest strengths
- 1-2 sentence summary of most important improvements

### Detailed Findings

| Issue | Location | Severity | Suggestion |
|-------|----------|----------|------------|

Severity levels:
- **High** — contradicts brand voice, compliance risk, or significantly undermines messaging
- **Medium** — inconsistent with guidelines but not damaging
- **Low** — minor style or preference issue

### Revised Sections
For the top 3-5 highest-severity issues, provide before/after showing original text and suggested revision.

### Legal/Compliance Flags
List any legal or compliance concerns separately with recommended actions.

## Step 5: Save and Present

Save to: `/clients/[client-name]/audits/brand-review-[YYYY-MM-DD].md` (with-client) or `/sessions/brand-review-[YYYY-MM-DD]/review.md` (no client).

Present the review to the user.

## Follow-Up

Ask: "Would you like me to:
- Revise the full content with these suggestions applied?
- Focus on fixing just the high-severity issues?
- Review additional content against the same guidelines?
- Help document the brand voice more thoroughly in the client brief?
- Draft new content that follows the brand voice using `/draft-content`?"
