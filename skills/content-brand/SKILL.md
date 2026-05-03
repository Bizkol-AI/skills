---
name: content-brand
description: >
  Use this skill to draft, review, or edit marketing content — blog posts,
  social copy (LinkedIn, X / Twitter, Instagram, TikTok, Facebook), email
  newsletters, landing pages, press releases, case studies, ad copy — and to
  enforce brand voice, tone, terminology, and style rules. Trigger on phrases
  like "draft content", "write a blog", "social post", "newsletter", "landing
  page copy", "press release", "case study", "brand review", "voice check",
  "tone audit", or the `/draft-content`, `/brand-review`, `/email-sequence`
  commands.
metadata:
  version: "0.1.0"
---

# Content + Brand Voice Skill

Two tightly-linked jobs in one skill: **drafting marketing content** and **reviewing content against documented brand voice**. They live together because every draft must be checked against the brand voice, and every voice review needs a concrete piece of content to evaluate.

> **Read first:** `marketing-framework/SKILL.md` for operating principles and the **client-folder prerequisite**. **Details:**
> - `${CLAUDE_PLUGIN_ROOT}/skills/content-brand/references/content-creation-framework.md` — content templates, headline / hook / CTA formulas, channel-specific guidelines
> - `${CLAUDE_PLUGIN_ROOT}/skills/content-brand/references/brand-voice-framework.md` — voice documentation template, tone adaptation, style enforcement, terminology

## Prerequisite — Client Folder

Strongly recommended: a populated `/clients/[client-name]/client-brief.md`. The client brief carries voice attributes, terminology, and messaging pillars. If missing, the user can still get a draft, but instruct them to run `/new-client` for richer, voice-consistent output.

## When to Use This Skill

- Drafting any marketing content (blog, social, email, landing, PR)
- Designing multi-email sequences (onboarding, nurture, re-engagement, win-back, launch)
- Reviewing existing content for brand-voice compliance
- Building or updating the brand voice doc for a client
- Ad creative copy (in combination with **`paid-media`** skill)

## Brand Voice Dimensions (track per client)

Every client's brief should have these dimensions documented:

- **Voice attributes** — 3–5 defining traits (e.g., authoritative, approachable, bold)
- **Tone spectrum** — how voice adapts across channels (blog vs. social vs. email vs. sales)
- **Terminology** — preferred terms, avoided terms, product name conventions, casing
- **Style rules** — Oxford comma, contractions, casing, number formatting, em-dash usage
- **Messaging pillars** — 3–5 key themes the brand consistently communicates
- **Forbidden** — words / claims / topics that must never appear

If any of these are missing for the client, prompt the user before drafting. See `brand-voice-framework.md` for the full template.

## Supported Content Types

| Type | Reference section | Typical output |
|------|-------------------|-----------------|
| Blog post | `content-creation-framework.md#blog` | 800–2000 words with H2 / H3 structure, internal links, meta |
| Social — LinkedIn | `#linkedin` | 1300–1800 char post + 3 hook variants |
| Social — X / Twitter | `#twitter` | Single post or thread (5–10 tweets) |
| Social — Instagram / TikTok | `#instagram-tiktok` | Caption + hook + hashtag set |
| Email newsletter | `#newsletter` | Subject + preview + body |
| Email sequence | `#sequences` | 3–7 emails with timing + subjects + bodies |
| Landing page | `#landing` | Hero + sections + CTA + social proof |
| Press release | `#press` | Headline + sub + dateline + body + boilerplate |
| Case study | `#case-study` | Challenge → approach → results structure |
| Ad copy | `#ads` | Headline / primary text / CTA variants per platform |

## Brand Review Output

When reviewing content (the `/brand-review` flow), produce:

1. **Severity-rated flags** — `critical` (off-voice, off-claim), `major` (style mismatch), `minor` (preferences).
2. **Suggested fix per flag** — specific replacement text, not generic advice.
3. **Before / after example** for the strongest 1–2 issues.
4. **Score** — 0–100 brand-voice compliance.

## Drafting Workflow

1. **Pull context** — `client-brief.md`, brand voice doc, recent published content, target keyword (if SEO-driven).
2. **Confirm angle** — get user sign-off on the hook / angle before writing the full draft.
3. **Draft** — apply voice + terminology + style rules from the brief.
4. **Self-review** — run the brand-review checklist on your own draft before delivery.
5. **Save**:
   - Single drafts → `/clients/[client-name]/content/drafts/[type]-[topic]-[YYYY-MM-DD].md`
   - Multi-email sequences → `/clients/[client-name]/content/sequences/email-sequence-[type]-[YYYY-MM-DD].md`
   - Brand reviews → `/clients/[client-name]/audits/brand-review-[YYYY-MM-DD].md`

## Audience Modes

- **Agency**: voice doc lives in client's brief; drafts saved per client; never reuse copy across clients.
- **Merchant**: voice doc is your own brand brief; drafts saved under `/clients/[brand-name]/content/`.

## Related Skills / Commands

- **`marketing-framework`** — operating principles
- **`seo-geo`** — for SEO-driven content briefs (target keyword, intent match)
- **`paid-media`** — for ad copy variants
- **`competitor-research`** — for VoC language to lift into copy
- `/draft-content`, `/brand-review`, `/email-sequence`, `/campaign-plan`
