---
name: kol-performance
description: Pull a Bizkol campaign's performance dashboard — aggregated metrics, platform breakdown, top posts, per-KOL performance, and the email outreach funnel (reply rate, hot replies, stale outbound) — and produce a structured performance report. Performance data lives in Bizkol; only the snapshot report is saved locally.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, TodoWrite, mcp__bizkol__list_campaigns, mcp__bizkol__get_campaign, mcp__bizkol__get_campaign_kols, mcp__bizkol__get_kol_performance, mcp__bizkol__get_kol_posts, mcp__bizkol__get_social_post_info, mcp__bizkol__list_email_conversations, mcp__bizkol__get_email_conversation
---

# /kol-performance $ARGUMENTS

Generate a performance report for a Bizkol KOL campaign. Pulls aggregated metrics from Bizkol, breaks down by platform and KOL, surfaces top-performing content + outliers, **and reports the email outreach funnel** — reply rate, awaiting-reply count, hot replies, stale outbound. The funnel matters even (especially) when no posts are live yet: for a pre-launch / mid-outreach campaign it's the only leading indicator of campaign health.

**Output is a self-contained HTML report** at `/clients/[name]/audits/kol-performance-[campaign-slug]-[YYYY-MM-DD].html` — same daily-dashboard format as `/kol-outreach`'s review report. Open in a browser, scan, act. Hot-reply rows embed any existing draft inline (from per-KOL profiles) with copy buttons.

## Step 0: Prerequisite — Client Folder

Before running, the client folder must exist at `/clients/[client-name]/` with `client-brief.md`. If not:

> "No client folder found for [client-name]. Run `/new-client [client-name]` first to create the brief and folder structure, then re-run this command."

Stop here if the folder is missing.

## Workflow

### Step 1 — Identify the campaign

Look up the campaign in the client's `client-brief.md` Bizkol Campaign Index. If `$ARGUMENTS` doesn't match an indexed campaign, call `list_campaigns` once with no status filter (or `status: "all"` if required by the tool):

```
Tool: list_campaigns
Parameters: {}
```

**Do not filter by campaign-level status** — campaign status (`draft / active / paused / completed / archived / deleted`) is not used reliably in the merchant code today, so filtering or surfacing it is misleading. List every campaign returned and ask the user to pick. Do not display the campaign-status field. Resolve the choice to a `campaignId`.

Then call `get_campaign(campaignId)` and capture `workflowTemplate` (one of `local_store`, `ecommerce_product`, or `null`). The valid `CampaignKOL.status` enum depends on this — see `assets/campaign-kol-statuses.md` for the canonical list. The performance funnel must use real status values, not invented ones.

### Step 2 — Pull the per-KOL roster

Bizkol no longer exposes a server-side dashboard tool — the dashboard view is composed client-side from per-KOL data. Start with the campaign's KOL roster:

```
Tool: get_campaign_kols
Parameters: { campaignId: "<id>" }
```

This returns each KOL's status, per-platform `socialHandles`, and `quotes` (pricing per deliverable). Use it as the spine of the report. Optionally archive raw to `/clients/[client-name]/raw-data/kol/performance-roster-[campaign-slug]-[YYYY-MM-DD].md`.

### Step 3 — Pull per-KOL performance and posts

For each KOL whose status is in the **Confirmed cooperation set** from `assets/campaign-kol-statuses.md` — i.e., one of: `confirmation`, `published`, `completed`, `appt_booked`, `store_visited`, `content_created`, `product_sent`, `product_received`, `content_draft`, `content_approved`. These are the post-negotiation statuses where shipped/published content may exist. Skip everything else (still in early-negotiation or `cancelled`).

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

### Step 3.5 — Pull the email outreach funnel (always run)

Posts are a *lagging* indicator. The email thread is the *leading* indicator — it's where you see whether the campaign is actually progressing. Run this every time, including when zero posts are live.

```
Tool: list_email_conversations
Parameters: { campaignId: "<id>", status: "all", limit: 100 }
```

Paginate via `nextToken` until exhausted. **Metadata only — do not call `get_email_conversation` for bodies here.** Body-level triage and reply drafting is `/kol-outreach`'s job; this command just counts and surfaces signals.

For each conversation, keep these fields: `threadId`, `kolId`, `kolData.name`, `kolData.socialAccounts[0].username`, `status`, `messageCount`, `unreadCount`, `lastMessageSender` (`"kol"` or `"user"`/`"sender"`), `lastMessageAt`, `lastMessagePreview` (truncate to ~120 chars).

Compute these funnel buckets (use `now = today UTC`):

| Bucket | Definition |
|---|---|
| **Contacted** | Any conversation exists for this KOL on this campaign |
| **Replied** | At least one inbound from the KOL — proxy: `messageCount > 1` OR `lastMessageSender = "kol"` OR `status` ∈ {`replied`, `awaiting_reply`} |
| **Awaiting your reply** | `lastMessageSender = "kol"` AND no thread `status = "closed"` — these are the threads the user owes a response on |
| **Awaiting their reply** | `lastMessageSender = "user"`/`"sender"` AND `status = "awaiting_reply"` |
| **Unread inbound** | `unreadCount > 0` — surface as a hot-signal count |
| **Stale outbound** | `lastMessageSender = user`, `lastMessageAt` >7 days ago, no inbound after — candidates for nudges |
| **Closed** | `status = "closed"` |
| **Hot replies (last 7 days)** | Bucket = "Awaiting your reply" AND `lastMessageAt >= now - 7 days` |

Cross-reference against the roster from Step 2:

- **Reply rate** = Replied ÷ Contacted (key health metric — typical cold KOL outreach: 5–15%)
- **Conversion rate** = (KOLs in any Confirmed cooperation status — see `assets/campaign-kol-statuses.md`: `confirmation`, `appt_booked`, `store_visited`, `content_created`, `product_sent`, `product_received`, `content_draft`, `content_approved`, `published`, `completed`) ÷ Replied
- For each KOL row in the per-KOL performance table, attach an `Email` column: `awaiting-you` / `awaiting-them` / `replied` / `not-contacted` / `closed`.

Optionally archive the slim list to `/clients/[client-name]/raw-data/kol/email-conversations-[campaign-slug]-[YYYY-MM-DD].jsonl` (one conversation per line, metadata only — no message bodies). Useful for diffing against a future run.

### Step 3.6 — Load per-KOL profiles for context

`/kol-outreach` writes per-KOL profile files at `/clients/[client-name]/raw-data/kol/profiles/[handle].md` capturing quoted price, payment terms, usage rights, exclusivity, timeline, special conditions, and a timestamped negotiation log.

For every KOL in the roster, check whether a profile file exists (filename = lowercased handle, no `@`). If yes, read its `Current state` block (skip the negotiation log here — too verbose for a performance report) and join it onto the per-KOL row:

- **Quoted price** → join into the per-KOL table's `Spend` column when the KOL is in any Confirmed cooperation status but the post isn't yet `published`/`completed` (use it as *committed* spend), and into the Hot replies / Awaiting-your-reply tables so the user sees the negotiation context inline.
- **Payment terms / usage rights / exclusivity** → only surface in the report if non-trivial (e.g., flag a "30-day exclusivity in B2B sales tools" as a campaign-wide constraint worth knowing).

If a profile file is missing for a KOL who has replied, mention it in Recommendations: "Run `/kol-outreach` to extract terms from these threads."

### Step 4 — Build the performance report (HTML)

**Path:** `/clients/[client-name]/audits/kol-performance-[campaign-slug]-[YYYY-MM-DD].html`

Same self-contained HTML format as `/kol-outreach`'s daily dashboard — single file, inline CSS+JS, no external assets. Reuse the CSS / status-badge / copy-button / HTML-escape conventions documented in `commands/kol-outreach.md` Step 7. The only differences are the section content. Order of sections (action-relevant first):

1. **Header** — campaign name, date, window, workflowTemplate, Bizkol campaignId. Do **not** display campaign-level status.
2. **At-a-glance stat strip** — Total posts live · Total reach · Total engagement · Average ER · Settled spend · Reply rate · Awaiting your reply.
3. **Outreach funnel table** — Contacted / Replied / Awaiting your reply / Awaiting their reply / Unread inbound / Stale outbound / Closed.
4. **Hot replies — KOLs awaiting your reply, last 7 days** — embed each row as a `.action` block (same structure as `/kol-outreach`'s action queue) with the KOL's last message preview, quoted price (from profile if present), and a Bizkol "Open →" link. If a per-KOL profile contains a draft from a prior `/kol-outreach` run, embed that draft inline with a copy button so the user can act directly from this report. If no draft exists, surface a small note: *"Run `/kol-outreach [client] [campaign]` to draft a reply"*.
5. **Stale outbound** — table; one row per stale thread with Bizkol "Open →" link.
6. **Platform breakdown** — table.
7. **Top 5 posts** — list, each with Open-on-platform link + a one-liner of why it worked.
8. **Underperformers (bottom 3)** — list with hypothesis + suggested action.
9. **Per-KOL performance** — table: Handle · Platform · Status (with badge) · Email (with badge: `awaiting-you` / `awaiting-them` / `replied` / `not-contacted` / `closed`) · Posts · Reach · ER · Spend.
10. **Recommendations** — bulleted list.
11. **Coverage diagnostics** — collapsed `<details>`. Same content as in `/kol-outreach`: pagination status, profiles missing for replied KOLs, etc.

Status badge classes: only use values from `assets/campaign-kol-statuses.md` — fall back to plain `<span class="badge">` for unknown values and log to Errors. Email badge classes: `badge-email-awaiting-you` (red), `badge-email-awaiting-them` (gray), `badge-email-replied` (green), `badge-email-not-contacted` (light gray), `badge-email-closed` (muted).

Add these badge styles to the inline CSS (alongside the kol-outreach styles):

```css
.badge-email-awaiting-you { background: #ffebee; color: #c62828; }
.badge-email-awaiting-them { background: #f1f3f5; color: #555; }
.badge-email-replied { background: #e8f5e9; color: #2e7d32; }
.badge-email-not-contacted { background: #f8f9fa; color: #888; }
.badge-email-closed { background: #ececec; color: #555; }
```

This is a **point-in-time snapshot**. Live performance always lives in Bizkol — re-run for fresh numbers.

### Step 5 — Suggest follow-ups

Suggest:
- For low-performing posts past `published`: surface a recommendation to revisit content (boost, reshare, etc.) — but do **not** propose status downgrades; once a KOL is at `published`/`completed`, status changes belong with the merchant, not this skill.
- For KOLs who never moved past `contacted` and the campaign is winding down: propose `cancelled` (the only valid "closed-without-success" terminal status — see `assets/campaign-kol-statuses.md`).
- Repurposing top posts: usage rights, paid amplification, organic reshares.
- **If "Awaiting your reply" > 0:** explicitly recommend `/kol-outreach [client] [campaign]` — that command drafts replies for every new inbound, with persistent per-thread state so reruns only act on net-new activity.
- **If outreach is ongoing (any contacted KOLs at all):** suggest scheduling `/kol-outreach` daily via `/schedule` so new replies are triaged within 24h.

Validate every status value mentioned in a recommendation against the canonical enum in `assets/campaign-kol-statuses.md`. Never invent.

## Graceful degradation

- **If Bizkol MCP isn't connected:** Cannot run — campaign data lives in Bizkol. Inform the user.
- **If a KOL's `get_kol_performance` or `get_kol_posts` call fails:** Note the KOL ID, exclude that KOL from the totals (don't double-count from a partial pull), and continue. Surface the missing-data list in the report's footer so the user knows the totals are conservative.
- **If `list_email_conversations` fails or pagination breaks mid-run:** capture what loaded, note the gap in the report's "Errors" / footer, and continue. Don't fail the whole report — post metrics can still be reported.
- **If the campaign has no live posts yet:** the report becomes a *funnel report* — emphasize the email outreach funnel section, treat reply-rate and "awaiting your reply" as the top-line metrics, and recommend `/kol-outreach` (and `/schedule` for daily cadence) to push outstanding negotiations forward. Do **not** skip Step 3.5 in this case — it's the only meaningful data the campaign has.
