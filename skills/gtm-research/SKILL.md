---
name: gtm-research
description: >
  Use this skill for go-to-market strategy work and new client / brand
  onboarding — demand validation, ICP definition, channel strategy, 90-day
  launch plans, market sizing, positioning, the master client / brand brief
  template, and the initial-audit setup. Trigger on phrases like "GTM",
  "go-to-market", "launch plan", "demand validation", "ICP", "channel
  strategy", "market sizing", "positioning", "onboard a client", "new
  client", "client brief", or the `/gtm-research`, `/new-client` commands.
metadata:
  version: "0.1.0"
---

# GTM Research + Client Onboarding Skill

Two jobs that share the same backbone: **answering the GTM question** for an idea-stage product, and **onboarding a new client / brand** so all the other skills have the context they need. Both use the same client brief template and similar research moves.

> **Read first:** `marketing-framework/SKILL.md` for operating principles, folder layout, and the **client-folder prerequisite** rule. **Details:**
> - `${CLAUDE_PLUGIN_ROOT}/skills/gtm-research/references/gtm-strategy-methodology.md` — full GTM research and 90-day launch plan
> - `${CLAUDE_PLUGIN_ROOT}/skills/gtm-research/references/client-brief-template.md` — master brief template with every onboarding field

## Prerequisite — Client Folder

`/new-client` is the **one command that creates** the client folder; every other client-scoped command requires it to already exist. `/gtm-research` may run in topic-only mode (no client) and save to a session folder, but its output is much richer when a client brief exists.

## When to Use This Skill

- A new client or brand needs onboarding (brief generation, folder setup, initial audit)
- Idea-stage product needs a GTM plan (demand validation → channels → 90-day plan)
- Existing client wants to enter a new market or launch a new product line
- Updating a client brief with new positioning, ICP, or channel changes
- Pre-launch research before any campaign-plan or content-strategy work

## Onboarding Workflow (`/new-client`)

Six steps, in order:

1. **Guided Q&A** — ask only what's missing; pull anything available from the user's existing materials first.
2. **Generate `client-brief.md`** using the template in `references/client-brief-template.md`.
3. **Create the folder structure** (see "Folder Output" below).
4. **Initial audit** — light-touch SEO + competitor + social baseline so the team has a starting picture.
5. **Connector inventory** — confirm Bizkol access, Windsor accounts (per ad / analytics platform), Semrush project, Shopify store (if merchant), Canva access — note gaps.

## GTM Research Workflow (`/gtm-research`)

Five layers, sequenced:

1. **Demand validation** — search-volume signals, Reddit / forum signals, ad-existence signals, related-product traction.
2. **ICP + JTBD** — who, what they hire the product to do, current alternatives, switching triggers.
3. **Competitive landscape** — direct, indirect, status-quo. Use **`competitor-research`** skill.
4. **Channel strategy** — which channels to test, with hypothesis + budget per channel + success metric.
5. **90-day launch plan** — week-by-week milestones, content calendar, paid test plan, KOL plan, measurement plan.

See `references/gtm-strategy-methodology.md` for the full sequence, data sources per layer, and the launch-plan template.

## Client Brief — minimum viable fields

A brief is "good enough" when it has at minimum:

- Brand name + URL + one-line description
- ICP (1–3 personas) with pain points and outcomes desired
- Positioning statement + 3–5 messaging pillars
- Brand voice (3–5 attributes + tone spectrum + style rules)
- Top 3 competitors
- Active channels (with priority order)
- Windsor account IDs per platform (GA4, GSC, Google Ads, etc.)
- Bizkol account / KOL campaign goals (if KOL is a channel)
- Reporting cadence + delivery channel
- Forbidden words / claims / topics

Anything missing → flag in the brief with `TBD: [field]` rather than skipping.

## Folder Output

Onboarding creates this structure under `/clients/[client-name]/`:

```
/clients/[client-name]/
  client-brief.md              ← master brief; every other skill reads here; includes Bizkol campaign index
  /raw-data/                   ← seeded with initial audit pulls, dated
    /seo/                      keyword-data-*, gsc-data-*, seo-audit-*
    /competitor/               competitor-list.md, competitor-deep-dive-*
    /voc/
    /analytics/                ga4-baseline-*
    /paid/                     google-ads-baseline-*, meta-ads-baseline-* (if applicable)
    /commerce/                 shopify-baseline-* (if merchant)
    /kol/
  /audits/
    initial-audit-[YYYY-MM-DD].md   ← onboarding deliverable
  /plans/
    gtm-plan-[YYYY-MM-DD].md        ← if GTM workflow ran
  /content/
    /drafts/
    /sequences/
```

There is **no `/campaigns/` folder** — Bizkol KOL campaigns live in Bizkol's database. The local index is in `client-brief.md`.

## Audience Modes

- **Agency**: many folders under `/clients/`; brief uses client's brand name; multi-client portfolio common.
- **Merchant**: one folder under `/clients/` for your brand; brief uses your brand name; treat your own org as the "client".

## Related Skills / Commands

- **`marketing-framework`** — operating principles + folder layout (this skill creates the folder)
- **`seo-geo`** — initial audit's SEO portion
- **`competitor-research`** — competitive landscape layer
- **`content-brand`** — uses the brief's voice + pillars
- `/new-client`, `/gtm-research`
