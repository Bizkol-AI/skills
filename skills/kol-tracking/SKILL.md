---
name: kol-tracking
description: >
  Use this skill to track the performance of specific KOL/influencer posts
  over time and replay their historical engagement curves. Maintains an
  append-only time-series log per campaign so you can answer "how did this
  post grow over the first 14 days?", "when did engagement peak?", "which
  posts are still earning a long tail?", and "what was the day-over-day
  delta on this post last week?". Pairs with `/schedule` for unattended
  daily captures. Trigger on phrases like "track this post", "post
  performance over time", "replay post history", "engagement curve",
  "long-tail performance", "day-over-day post growth", or the
  `/track-posts` command.
metadata:
  version: "0.1.0"
---

# KOL Post Tracking — Time-Series Performance Skill

Bizkol's MCP returns **point-in-time** snapshots: `get_social_post_info`, `get_kol_posts`, and `get_kol_performance` all show whatever the platform reports right now. There is no built-in history. This skill solves that — it captures repeated snapshots into an append-only log and lets you replay the trajectory of any tracked post.

> **Read first:** `marketing-framework/SKILL.md` for the client-folder prerequisite, the data-first rule, and the Bizkol-native campaign rule. **Related:** `kol-operations/SKILL.md` for the underlying Bizkol tools used here.

## Prerequisite — Client Folder

Before any `/track-posts` action, the client folder must exist at `/clients/[client-name]/` with `client-brief.md`. If missing, stop and instruct the user to run `/new-client [name]` first.

## Mental Model

Two files per client power the whole skill:

```
/clients/[name]/raw-data/kol/
  tracked-posts.json                      ← registry: which posts, status, metadata
  post-history-[campaign-slug].ndjson     ← append-only time-series log per campaign
  post-history-adhoc.ndjson               ← log for posts tracked outside any campaign
```

- The **registry** records *intent*: which posts to capture, current state (`active` / `paused` / `stopped` / `deleted`), and identifiers needed for the next call.
- The **NDJSON log** records *observations*: one line per capture, append-only, never rewritten. This is the source of truth for replay.

Splitting intent from observations means you can pause a post without losing its history, resume it later, or stop it entirely while keeping the curve intact for analysis.

## File Schemas

### `tracked-posts.json`

```json
{
  "version": 1,
  "posts": [
    {
      "postId": "<bizkol or platform-native id>",
      "platform": "instagram|tiktok|youtube|x",
      "url": "https://...",
      "shortcode": "<platform shortcode if available>",
      "kolId": "<bizkol kolId>",
      "handle": "@username",
      "campaignId": "<bizkol campaignId or null>",
      "campaignSlug": "<lowercase-hyphenated or 'adhoc'>",
      "addedAt": "2026-05-02T10:00:00Z",
      "lastCapturedAt": "2026-05-09T08:00:00Z",
      "status": "active",
      "stopReason": null,
      "notes": ""
    }
  ]
}
```

`status` values: `active` (capture every run), `paused` (skip for now, keep in registry), `stopped` (manually retired — keep in registry but never capture), `deleted` (the platform 404'd the post — auto-set; never capture).

### `post-history-[campaign-slug].ndjson`

One JSON object per line. Append-only — never edit existing rows; corrections are new rows with a `correction: true` marker.

```jsonl
{"timestamp":"2026-05-02T10:00:00Z","postId":"abc","platform":"tiktok","url":"https://...","kolId":"k1","handle":"@user","campaignId":"camp1","postedAt":"2026-05-01T18:30:00Z","views":12450,"likes":890,"comments":42,"shares":17,"saves":63,"engagementTotal":1012,"reach":12450,"engagementRate":0.0813,"followersAtCapture":245000,"sourceTool":"get_social_post_info"}
{"timestamp":"2026-05-03T10:00:00Z","postId":"abc","platform":"tiktok","url":"https://...","kolId":"k1","handle":"@user","campaignId":"camp1","postedAt":"2026-05-01T18:30:00Z","views":48200,"likes":3140,"comments":189,"shares":76,"saves":214,"engagementTotal":3619,"reach":48200,"engagementRate":0.0751,"followersAtCapture":245100,"sourceTool":"get_social_post_info"}
```

**Field rules:**
- `timestamp` — ISO 8601 UTC, when the snapshot was captured.
- `postedAt` — the post's own creation timestamp from the platform (same across all rows for a given post; redundant by design so each row is self-contained for replay).
- `views` and `reach` — both included; on platforms that don't separate them (TikTok/IG video), set both equal to the view count. On X, `views` = impressions if available; `reach` may be null.
- `engagementTotal` — sum of likes + comments + shares + saves (omit fields that are null when summing).
- `engagementRate` — `engagementTotal / reach` if reach is positive, otherwise `engagementTotal / followersAtCapture`. Note which denominator was used implicitly via whether reach is null.
- `followersAtCapture` — pulled from `get_kol_performance` or last-known cached value to avoid an extra call per post; refresh weekly.
- `sourceTool` — which Bizkol tool produced the snapshot, for debugging.

NDJSON is the right shape because: append-only writes are atomic-enough for this use case, every line is self-contained (so partial corruption only loses one row), and standard tools (`jq`, `wc -l`, `grep`) replay it without parsing the whole file.

## Capture Workflow

1. **Load the registry** at `/clients/[name]/raw-data/kol/tracked-posts.json`. If it doesn't exist, create an empty `{"version":1,"posts":[]}`.
2. **Resolve which posts to capture** — every `posts[]` entry where `status === "active"`.
3. **Group by `kolId`** to amortize follower-count lookups: for each KOL, call `get_kol_performance` once per platform (cache `followersAtCapture` for that run).
4. **For each active post**, call `get_social_post_info` with its `platform` and `url` (or `shortcode`). On 404 / "not found", set the registry entry's `status: "deleted"` and `stopReason: "platform-404"` and skip the append.
5. **Compose a snapshot row** per the schema above and append it to `post-history-[campaignSlug].ndjson` (or `post-history-adhoc.ndjson` if `campaignSlug === "adhoc"`).
6. **Update the registry's `lastCapturedAt`** for each successfully captured post.
7. **Report** at the end: posts captured, posts deleted (auto-stopped), posts that errored (kept active for retry next run), total rows appended.

## Replay Workflow

Given a `postId`, `url`, or `shortcode`:

1. Find the matching entry in `tracked-posts.json` to discover its `campaignSlug`.
2. Stream the matching `post-history-[slug].ndjson`, filter rows where `postId` matches.
3. Sort by `timestamp` ascending.
4. Compute deltas vs. the previous row: `Δviews`, `Δengagement`, `ΔER`.
5. Output:
   - **Headline**: post URL, total captures, first-seen → last-seen window, current vs. peak engagement.
   - **Trajectory table**: `timestamp | views | likes | comments | engagement | ER% | Δengagement (24h)`.
   - **Simple ASCII curve** (engagement over time) — see "ASCII chart" rule below.
   - **Milestones**: 24h / 7d / 30d post-publish numbers if the timeline allows.
6. If the user asked for a written summary, save to `/clients/[name]/audits/post-replay-[postId-or-shortcode]-[YYYY-MM-DD].md`. Otherwise present in chat.

**ASCII chart rule:** when fewer than 8 data points, just show the table. With 8+ points, render a 30-column-wide sparkline using `▁▂▃▄▅▆▇█` mapped to the min-max engagement range across the captured window. Don't try to draw axes — the table already has the numbers.

## When to Use This Skill

- Tracking a launch campaign's posts daily for the first two weeks
- Diagnosing whether a post had a one-day spike or a sustained tail
- Comparing two campaigns' post curves side by side
- Justifying a usage-rights / paid-amplification decision (post still gaining engagement = boost)
- Building a "best long-tail performer" leaderboard across a quarter
- Verifying that a KOL's reported reach matches what the platform actually showed

## Cadence Recommendation

Engagement growth is heavily front-loaded. The recommended capture schedule per post:

| Post age | Cadence | Rationale |
|---|---|---|
| 0–48 hours | every 6 hours | The bulk of platform-distribution activity happens here; coarser sampling misses peaks |
| 2–14 days | daily | Long-tail growth visible at this resolution |
| 14–30 days | every 3 days | Most posts plateau; rare comeback posts still detectable |
| 30+ days | weekly, then auto-stop | Auto-stop when 3 consecutive captures show < 1% engagement growth |

Implement via `/schedule` — for example, a routine that runs `/track-posts [client] --capture` daily at 09:00 UTC. The skill itself is stateless w.r.t. cadence; the registry decides what gets captured each run.

## Tool Priority

| Need | Primary | Fallback |
|------|---------|----------|
| Live post engagement snapshot | Bizkol MCP `get_social_post_info` | Chrome on the post URL — read counts manually |
| Discover campaign posts to track | Bizkol MCP `get_campaign_kols` + `get_kol_posts` per shipped KOL | — |
| Refresh follower count for ER denominator | Bizkol MCP `get_kol_performance` | Cached value from last successful refresh |

**Bizkol-first rule** applies — only fall back to Chrome if Bizkol returns null/error for a specific post.

## Folder Layout

```
/clients/[client-name]/
  /raw-data/kol/
    tracked-posts.json                       ← registry (intent)
    post-history-[campaign-slug].ndjson      ← time series (observations)
    post-history-adhoc.ndjson                ← when no campaign
  /audits/
    post-replay-[postId-or-shortcode]-[YYYY-MM-DD].md  ← optional written replay
```

Both files live under `/raw-data/kol/` because they're raw, dated observations — same category as other archival pulls. Replay summaries are interpretation, so they go to `/audits/`.

## Audience Modes

- **Agency**: per-client tracking; each client has their own registry and history files. Cadence may differ per client (premium clients get tighter capture).
- **Merchant**: single brand registry. Default to all your shipped posts being tracked unless you explicitly stop one.

## Related Skills / Commands

- **`marketing-framework`** — operating principles, folder layout, prerequisite rule
- **`kol-operations`** — campaign + KOL management; `/kol-performance` is a *snapshot*, this skill is a *time series*
- **`/track-posts`** — the standalone command (add, capture, replay, list, pause, stop)
- **`/schedule`** — recommended pairing for unattended daily captures
