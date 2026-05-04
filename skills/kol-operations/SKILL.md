---
name: kol-operations
description: >
  Use this skill for any KOL / influencer / creator workflow run through the
  Bizkol MCP — creator discovery, campaign creation and management, importing
  KOLs by handle, email outreach tracking, and per-campaign performance
  review. Trigger on phrases like "find creators", "search KOLs", "shortlist
  influencers", "create a campaign", "import handles", "outreach status",
  "campaign performance", "creator dashboard", or any of the `/kol-*`
  commands.
metadata:
  version: "0.2.0"
---

# KOL Operations — Bizkol MCP Playbook

The single source of truth for **all** creator/influencer/KOL workflows in this toolbox. Anything involving creator discovery, campaign management, outreach, or KOL performance must use the **Bizkol MCP** first — it's faster, structured, and persists state across sessions.

> **Read first:** `marketing-framework/SKILL.md` for cross-cutting operating principles (tool priority, data-first, graceful degradation, raw-data rule, **client-folder prerequisite**, **Bizkol-native campaign rule**).

## Prerequisite — Client Folder

Before any KOL command runs, the client folder must exist at `/clients/[client-name]/` with `client-brief.md`. If missing, stop and instruct the user to run `/new-client [name]` first. This applies to `/kol-campaign`, `/kol-discovery`, `/kol-outreach`, and `/kol-performance`.

## Bizkol-Native Campaign Rule

**Campaigns live in Bizkol, not on disk.** When the user works with a campaign, the source of truth is the Bizkol MCP. Never create local `/campaigns/[name]/` folders mirroring KOL list, outreach state, or performance — pull live every time.

What lives on disk:
- **Campaign index** in `/clients/[name]/client-brief.md` mapping `campaign-name → Bizkol campaignId`. Update this whenever a new campaign is created via `create_campaign`.
- **Strategic brief** written *before* the Bizkol campaign exists → `/clients/[name]/plans/kol-campaign-brief-[campaign].md`. This is the human-written intent (goals, audience, tier, budget, content angles) that informs `create_campaign` parameters.
- **One-off generated artifacts** when the user runs a command:
  - Performance snapshots → `/clients/[name]/audits/kol-performance-[campaign]-[YYYY-MM-DD].md`
  - Outreach reply drafts → `/clients/[name]/content/drafts/outreach-[campaign]-[YYYY-MM-DD].md`
  - Discovery shortlists → `/clients/[name]/audits/kol-discovery-[topic]-[YYYY-MM-DD].md`
- **Optional raw API archival** if explicitly requested → `/clients/[name]/raw-data/kol/`

When asked "what's the status of campaign X", **always** call `get_campaign` / `get_campaign_kols` — never read a local file.

## Bizkol-First Rule

For any KOL-related task, **start with the Bizkol MCP** at `https://mcp.bizkol.ai/api/mcp` (OAuth). Bizkol covers the full surface — KOL DB, live social lookups across Instagram / TikTok / YouTube / X, Reddit research, campaigns, and outreach. Only fall back to Chrome browser automation for platforms or surfaces Bizkol doesn't cover yet.

| Surface | Bizkol tools | Use For |
|---------|--------------|---------|
| **Campaigns** | `create_campaign`, `get_campaign`, `list_campaigns`, `update_campaign`, `get_campaign_import_jobs` | Spin up, update, and track campaigns. Dashboard-style aggregates are composed client-side from `get_campaign_kols` + per-KOL calls (see Performance Review workflow). |
| **Campaign KOLs** | `add_kols_to_campaign`, `get_campaign_kols`, `import_kols_to_campaign_by_username`, `remove_kols_from_campaign`, `update_campaign_kol` | Build the shortlist, import by handle (auto-scrapes new profiles), apply per-KOL status / notes / quotes |
| **KOL DB** | `search_kols`, `get_kol`, `get_kol_performance`, `get_kol_posts` | Search the Bizkol creator DB; pull profile, performance, recent posts |
| **Live social** (IG / TikTok / YouTube / X) | `get_social_kol_profile`, `get_social_post_info`, `search_social_kols` | Discover or look up creators not yet in the Bizkol DB |
| **Reddit** | `search_reddit`, `get_reddit_post`, `get_reddit_subreddit`, `get_reddit_user` | Find Reddit-native creators and high-influence users; surface category-relevant threads, audiences, and VoC for campaign briefs |
| **Email** | `list_email_conversations`, `get_email_conversation` | Read KOL outreach threads (sending happens in Bizkol's UI) |
| **Account** | `get_billing_status`, `get_invoices`, `get_usage_summary`, `get_user_profile` | Plan, usage, invoices |

## Audience Modes

This skill serves both audiences without forking — the only difference is how many client folders exist under `/clients/`:

- **Agency mode** (default): One folder per client. Strategic briefs and generated artifacts live under `/clients/[client-name]/`.
- **Merchant mode**: Single folder for your own brand under `/clients/[brand-name]/`. The "client" is your own brand profile.

The Bizkol MCP itself is identical in both modes.

## Standard Workflows

### Discovery → Shortlist → Import
1. Define the brief: topic, geos, follower band, engagement floor, content style.
2. **Bizkol DB first:** `search_kols` with the brief. Save raw response (optional) to `/clients/[name]/raw-data/kol/`.
3. **Live social discovery** for niches Bizkol's DB doesn't cover yet: `search_social_kols` across Instagram / TikTok / YouTube / X.
4. **Reddit discovery** for community-led categories (B2B, hobbies, niche DTC): `search_reddit` to find category subreddits, then `get_reddit_subreddit` for top contributors and `get_reddit_user` for posting history.
5. Triage the list against the brand fit criteria from `client-brief.md`.
6. Write strategic brief to `/clients/[name]/plans/kol-campaign-brief-[campaign].md` (if not already done).
7. `create_campaign` (or `get_campaign` if continuing).
8. **Add the new `campaignId` to the campaign index in `client-brief.md`.**
9. `import_kols_to_campaign_by_username` for handles not in the DB — Bizkol auto-scrapes.
10. `add_kols_to_campaign` for KOLs already in the DB.
11. The shortlist now lives in Bizkol — confirm via `get_campaign_kols`. Optionally export a snapshot to `/clients/[name]/audits/kol-discovery-[topic]-[YYYY-MM-DD].md`.

### Outreach Tracking
- `/kol-outreach [client] [campaign]` is the single stateful command for the email inbox of a campaign. Idempotent — safe to run ad-hoc or daily.
- Paginates `list_email_conversations` to handle 500+ KOL campaigns; only fetches `get_email_conversation` bodies for threads with new inbound since the last run.
- Persistent state at `/clients/[name]/raw-data/kol/outreach-state-[campaign-slug].json` tracks per-thread `lastDraftedForMessageId`, `draftStatus` (`pending_send | sent | superseded | skipped`), and `decision` (`negotiating | agreed | declined | no_response`) — so reruns only redraft when the inbound has actually advanced.
- Drafts → `/clients/[name]/content/drafts/outreach-[campaign-slug]-[YYYY-MM-DD].md`.
- Review report → `/clients/[name]/audits/kol-outreach-[campaign-slug]-[YYYY-MM-DD].md`.
- Sending always happens in Bizkol's UI; this skill never sends. Pair with `/schedule` for a daily cron.

### Performance Review
- `get_campaign_kols` for the per-KOL roster — status, `socialHandles`, `quotes`. This is the spine of the report.
- For each shipped KOL (status `posted` / `live` / `confirmed` / `completed` / `concluded`): `get_kol_performance` per platform/handle, plus `get_kol_posts`.
- Aggregate **client-side** to produce totals, per-platform breakdown, top posts, and underperformers.
- Use `get_social_post_info` for any single-post deep-dive.
- Report saved to `/clients/[name]/audits/kol-performance-[campaign]-[YYYY-MM-DD].md`.

## Folder Layout Reminder

```
/clients/[client-name]/
  client-brief.md                  ← campaign index lives here
  /plans/
    kol-campaign-brief-[name].md   ← strategic brief (pre-creation)
  /audits/
    kol-performance-[name]-[YYYY-MM-DD].md
    kol-discovery-[topic]-[YYYY-MM-DD].md
    kol-outreach-[name]-[YYYY-MM-DD].md
  /content/drafts/
    outreach-[name]-[YYYY-MM-DD].md
  /raw-data/kol/                   ← optional API archival only
```

There is **no `/campaigns/` folder.** Live campaign data is always pulled from Bizkol.

## Fallback: Chrome browser

When Bizkol MCP doesn't cover a surface (e.g., LinkedIn creators, Pinterest accounts, podcast hosts) or when a creator isn't returned by `search_social_kols`, fall back to Chrome browser automation:

- Visit the platform / profile directly.
- Capture the relevant metrics manually (followers, engagement, recent content, contact info).
- Once a useful creator is identified, prefer **`import_kols_to_campaign_by_username`** to bring them into Bizkol so they're tracked centrally going forward.

## Related Commands

- `/kol-discovery` — topic/handle-driven creator search
- `/kol-campaign` — end-to-end campaign workflow (writes the strategic brief, calls `create_campaign`, updates the index)
- `/kol-outreach` — stateful email inbox triage for a campaign at scale (500+ KOLs); incremental drafts with per-thread state, idempotent on any cadence
- `/kol-performance` — per-KOL roster + posts pull, aggregated client-side into a dashboard view + analysis
