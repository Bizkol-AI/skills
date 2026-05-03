---
name: kol-performance
description: Pull a Bizkol campaign's performance dashboard — aggregated metrics, platform breakdown, top posts, per-KOL performance — and produce a structured performance report. Performance data lives in Bizkol; only the snapshot report is saved locally.
---

# /kol-performance $ARGUMENTS

Generate a performance report for a Bizkol KOL campaign. Pulls aggregated metrics from Bizkol, breaks down by platform and KOL, and surfaces top-performing content + outliers.

## Step 0: Prerequisite — Client Folder

Before running, the client folder must exist at `/clients/[client-name]/` with `client-brief.md`. If not:

> "No client folder found for [client-name]. Run `/new-client [client-name]` first to create the brief and folder structure, then re-run this command."

Stop here if the folder is missing.

## Workflow

### Step 1 — Identify the campaign

Look up the campaign in the client's `client-brief.md` Bizkol Campaign Index. If `$ARGUMENTS` doesn't match an indexed campaign, list both `active` and `draft` campaigns from Bizkol. The `status` parameter accepts a single value, so make two calls:

```
Tool: list_campaigns
Parameters: { status: "active" }

Tool: list_campaigns
Parameters: { status: "draft" }
```

Merge both result sets and present them to the user, marking each entry with its status (draft / active) so they can pick the right one. Resolve the choice to a `campaignId`.

### Step 2 — Pull the per-KOL roster

Bizkol no longer exposes a server-side dashboard tool — the dashboard view is composed client-side from per-KOL data. Start with the campaign's KOL roster:

```
Tool: get_campaign_kols
Parameters: { campaignId: "<id>" }
```

This returns each KOL's status, per-platform `socialHandles`, and `quotes` (pricing per deliverable). Use it as the spine of the report. Optionally archive raw to `/clients/[client-name]/raw-data/kol/performance-roster-[campaign-slug]-[YYYY-MM-DD].md`.

### Step 3 — Pull per-KOL performance and posts

For each KOL whose status is `posted`, `live`, `confirmed`, `completed`, or `concluded`:

```
Tool: get_kol_performance
Parameters: { kolId: "<id>", platform: "<from socialHandles>", username: "<from socialHandles>" }

Tool: get_kol_posts
Parameters: { kolId: "<id>" }
```

Aggregate **client-side** to build the dashboard view:
- **Totals** across the campaign: posts, reach, engagement, average ER, settled spend (sum of `quotes` × deliverables actually shipped).
- **Per-platform breakdown**: group by the `platform` field on each KOL/post; sum reach, engagement, spend; compute ER and CPE per platform.
- **Top 5 posts**: rank all posts pulled in this step by engagement (or ER if reach is unreliable for the platform).
- **Underperformers**: posts whose ER is more than 30% below the campaign average.

For specific posts that need engagement detail:

```
Tool: get_social_post_info
Parameters: { platform: "...", identifier: "<URL or post ID>" }
```

### Step 4 — Build the performance report

Save to `/clients/[client-name]/audits/kol-performance-[campaign-slug]-[YYYY-MM-DD].md`:

```markdown
# Campaign Performance — [campaign name]
**Date:** [YYYY-MM-DD] | **Window:** [start] → [end] | **Status:** [active/paused/completed]
**Bizkol campaignId:** [id]

## Executive summary
- Total posts live: X
- Total reach: X
- Total engagement: X
- Average ER: X% (vs platform median Y%)
- Spend (settled): $X
- CPM: $X | Cost per engagement: $X

## Platform breakdown

| Platform | Posts | Reach | Engagement | ER | Spend | CPE |
|----------|-------|-------|------------|----|-------|-----|
| Instagram | ... | ... | ... | ... | ... | ... |
| TikTok | ... | ... | ... | ... | ... | ... |
| YouTube | ... | ... | ... | ... | ... | ... |

## Top 5 posts (by engagement)

1. [@handle] — [post URL] — Reach: X · Engagement: X · ER: X%
   - Why it worked: [one line]

## Underperformers (bottom 3)

1. [@handle] — [post URL] — ER: X% (vs campaign avg Y%)
   - Possible cause: [hypothesis]
   - Action: [boost / no-action / re-cut]

## Per-KOL performance

| Handle | Platform | Posts | Reach | ER | Spend | Status |
|--------|----------|-------|-------|----|-------|--------|

## Recommendations
- [actionable next step 1]
- [actionable next step 2]

## Next campaign learnings
- [what to keep doing]
- [what to change]
```

This is a **point-in-time snapshot**. Live performance always lives in Bizkol — re-run for fresh numbers.

### Step 5 — Suggest follow-ups

Suggest:
- Updating low-performer status with `update_campaign_kol` (e.g., `concluded` or `paused`)
- Repurposing top posts: usage rights, paid amplification, organic reshares
- Updating the campaign-index `Status` column in `client-brief.md` if the campaign has finished

## Graceful degradation

- **If Bizkol MCP isn't connected:** Cannot run — campaign data lives in Bizkol. Inform the user.
- **If a KOL's `get_kol_performance` or `get_kol_posts` call fails:** Note the KOL ID, exclude that KOL from the totals (don't double-count from a partial pull), and continue. Surface the missing-data list in the report's footer so the user knows the totals are conservative.
- **If the campaign has no live posts yet:** Surface that in the executive summary and recommend `/kol-outreach` to push outstanding negotiations forward.
