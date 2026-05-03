---
name: kol-campaign
description: End-to-end KOL/influencer campaign workflow using the Bizkol MCP — write strategic brief, create the campaign, search for relevant creators, shortlist, import by handle, and track performance. Campaign data lives in Bizkol; only the strategic brief and campaignId index are stored locally.
---

# /kol-campaign $ARGUMENTS

Run a complete KOL/influencer campaign workflow on the Bizkol platform. Campaign **runtime state** (KOL list, statuses, outreach, performance) lives in Bizkol's database — pulled live via the MCP. The only local artifacts are:
1. The **strategic brief** that informs `create_campaign` parameters → `/clients/[name]/plans/kol-campaign-brief-[campaign].md`
2. The **campaign index** in `client-brief.md` (campaign-name → Bizkol `campaignId`)

## Step 0: Prerequisite — Client Folder

Before running, the client folder must exist at `/clients/[client-name]/` with `client-brief.md`. If not:

> "No client folder found for [client-name]. Run `/new-client [client-name]` first to create the brief and folder structure, then re-run this command."

Stop here if the folder is missing.

## Prerequisites

- **Bizkol MCP** must be connected (`https://mcp.bizkol.ai/api/mcp`). On first call, Claude triggers the OAuth flow.
- A populated `/clients/[client-name]/client-brief.md` (per Step 0).

## Workflow

### Step 1 — Confirm the campaign brief

Read the client's `client-brief.md` (brand voice, ICP, positioning, budget, active channels). Confirm with the user:
- **Campaign goal** (awareness, conversions, content library, launch hype)
- **Target audience** (demographics, interests, geography)
- **Platforms** (Instagram, TikTok, YouTube, X — pick 1–3)
- **Creator tier** (nano <10k, micro 10k–100k, mid 100k–1M, macro 1M+)
- **Budget range and timeline** (start/end dates, settlement currency)
- **Content type** (in-feed posts, reels/shorts, stories, live, longform video)

Save the strategic brief to `/clients/[client-name]/plans/kol-campaign-brief-[campaign-slug].md`. Use the brief slug (lowercase-hyphenated campaign name) consistently.

### Step 2 — Create the Bizkol campaign

Use `create_campaign`:

```
Tool: create_campaign
Parameters:
  name: "<campaign-name>"
  description: "<one-line goal + target audience>"
  startDate: "<YYYY-MM-DD>"
  endDate: "<YYYY-MM-DD>"
  status: "draft"
  settlementCurrency: "USD"  # or CNY, EUR, etc.
  tags: ["<platform>", "<creator-tier>", "<vertical>"]
```

**On success, capture the returned `campaignId` and update the campaign index in `client-brief.md`:**

```markdown
## Bizkol Campaign Index
| Campaign Name | Bizkol campaignId | Status | Strategic Brief | Created |
|---|---|---|---|---|
| [campaign-name] | [campaignId] | draft | /clients/[name]/plans/kol-campaign-brief-[slug].md | [YYYY-MM-DD] |
```

This is the only place the `campaignId` lives locally — every subsequent call needs it.

### Step 3 — Discover candidate KOLs

Two paths depending on whether the user has handles in mind:

**A. Discovery from scratch** — search the Bizkol KOL DB:

```
Tool: search_kols
Parameters:
  query: "<topic / vertical keyword>"
  platform: "<instagram|tiktok|youtube|x>"
  minFollowers: <number>
  maxFollowers: <number>
  minEngagementRate: 0.01  # 2% floor for most verticals
  maxPrice: <number>
  pageSize: 25
```

For creators not yet in the Bizkol DB, also run `search_social_kols` (live platform discovery) and pick handles to import.

**B. Known handles** — skip discovery and go straight to import (Step 5).

Optional: archive the raw `search_kols` response to `/clients/[client-name]/raw-data/kol/search-[campaign-slug]-[YYYY-MM-DD].md`. Skip if the user doesn't want a snapshot — Bizkol re-runs the query freely.

### Step 4 — Shortlist (in Bizkol)

For each candidate, pull richer data:

```
Tool: get_kol
Parameters: { kolId: "..." }

Tool: get_kol_performance
Parameters: { kolId: "...", platform: "...", username: "..." }

Tool: get_kol_posts
Parameters: { kolId: "..." }
```

Score against:
- **Audience fit** (does their content/engaged audience match the brief?)
- **Engagement quality** (real engagement rate, comment depth, not just followers)
- **Price-to-reach ratio** (price per 1k engaged followers)
- **Brand safety** (recent content audit, controversy check)
- **Recent post cadence** (active in the last 30 days?)

The shortlist itself is recorded in **Bizkol via Step 5** — `add_kols_to_campaign` with `initialStatus: "shortlisted"` and `notes` carrying the scoring rationale. **Do not** write a local `kol-shortlist.md` file. If the user wants a one-off export, generate `/clients/[name]/audits/kol-discovery-[campaign-slug]-[YYYY-MM-DD].md`.

### Step 5 — Import to the campaign

For KOLs already in the Bizkol DB:

```
Tool: add_kols_to_campaign
Parameters:
  campaignId: "<id>"
  kolIds: ["<id1>", "<id2>", ...]
  initialStatus: "shortlisted"
  notes: "<scoring rationale or fit summary>"
```

For KOLs identified by social handle but not yet in the Bizkol DB (auto-scrapes):

```
Tool: import_kols_to_campaign_by_username
Parameters:
  campaignId: "<id>"
  platform: "<instagram|tiktok|youtube|x>"
  usernames: ["@handle1", "@handle2", ...]   # max 50 per call
```

This is **async**. Poll with `get_campaign_import_jobs` until complete:

```
Tool: get_campaign_import_jobs
Parameters: { campaignId: "<id>" }
```

### Step 6 — Confirm in Bizkol

Pull the live campaign view to confirm everyone is in:

```
Tool: get_campaign_kols
Parameters: { campaignId: "<id>" }
```

Verify socialHandles, status, and any quotes are populated. Note any KOLs whose import failed and re-run for those. **Do not write a local file** — this state lives in Bizkol; query it on demand.

### Step 7 — Hand off

The shortlist is staged in Bizkol. Suggest the next command:

> Next: run `/kol-outreach [campaign-name]` to triage outreach threads and draft follow-ups, or `/kol-performance [campaign-name]` once posts go live.

## Output

The only file artifacts produced by this command:
- `/clients/[name]/plans/kol-campaign-brief-[slug].md` — the strategic brief (Step 1)
- Updated `client-brief.md` campaign index (Step 2)

Optionally (only if user explicitly asks):
- `/clients/[name]/audits/kol-discovery-[slug]-[YYYY-MM-DD].md` — exported shortlist snapshot
- `/clients/[name]/raw-data/kol/search-[slug]-[YYYY-MM-DD].md` — raw `search_kols` response

Everything else (KOL list, statuses, outreach, performance) stays in Bizkol — pull live whenever needed.

## Graceful degradation

- **If Bizkol MCP isn't connected:** Inform the user — point them to `<https://bizkol.ai>` to sign in, then re-run. Do not attempt to substitute social scrapers; the workflow needs Bizkol's campaign object to function.
- **If `import_kols_to_campaign_by_username` returns partial failure:** Note which usernames failed and continue with the successful ones. Surface the failed list to the user for manual handling.
