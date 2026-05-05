---
description: Track KOL post performance over time and replay historical engagement curves. Subcommands: --add, --add-from-campaign, --capture, --replay, --list, --pause, --resume, --stop.
argument-hint: "[client-name] [--add URL | --add-from-campaign NAME | --capture | --replay POSTID | --list | --pause POSTID | --resume POSTID | --stop POSTID]"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, TodoWrite, Task
---

Read the marketing-framework skill and the kol-tracking skill before starting.

> **Connector:** Bizkol MCP (`get_social_post_info`, `get_kol_posts`, `get_campaign_kols`, `get_kol_performance`). All snapshots come from Bizkol; no scraper fallback unless Bizkol returns null/error for a specific post.

`/track-posts` maintains an append-only time-series log of KOL post performance under `/clients/[client-name]/raw-data/kol/`. It pairs with `/schedule` for unattended daily captures.

## Step 0: Prerequisite — Client Folder

Before running, the client folder must exist at `/clients/[client-name]/` with `client-brief.md`. If not:

> "No client folder found for [client-name]. Run `/new-client [client-name]` first to create the brief and folder structure, then re-run this command."

Stop here if the folder is missing.

## Step 1: Parse the Action

Parse `$ARGUMENTS` into `[client-name]` plus exactly one action flag. If no flag is given, default to `--list` and warn the user.

Valid actions:
- `--add <url-or-shortcode>` — add a single post to the registry by its platform URL
- `--add-from-campaign <campaign-name>` — find every shipped post for the campaign's KOLs and add all to the registry
- `--capture` — run a snapshot for every `active` post in the registry (this is the schedule target)
- `--replay <postId-or-url-or-shortcode>` — print the historical trajectory of one post
- `--list` — show the registry (which posts are being tracked, status, last-seen engagement)
- `--pause <postId-or-url>` — set status to `paused` (no captures, registry retained)
- `--resume <postId-or-url>` — set status back to `active`
- `--stop <postId-or-url>` — set status to `stopped` (manual retire; history preserved)

## Step 2: Load the Registry

Read `/clients/[client-name]/raw-data/kol/tracked-posts.json`. If it doesn't exist, create an empty file:

```json
{ "version": 1, "posts": [] }
```

The registry schema and lifecycle states (`active` / `paused` / `stopped` / `deleted`) are defined in the `kol-tracking` skill — follow them exactly.

## Step 3: Execute the Action

### `--add <url-or-shortcode>`

1. Resolve the URL to `(platform, shortcode, postId)` by calling `get_social_post_info`. If it returns 404, abort and tell the user the post wasn't found.
2. From the response, extract `kolId`, `handle`, and the post's `postedAt` timestamp.
3. Ask the user (one short question): which campaign should this post be associated with? Offer options from the client's `client-brief.md` Bizkol Campaign Index plus an `adhoc` option. Resolve to `campaignId` and `campaignSlug`.
4. Append a new entry to `posts[]` with `status: "active"`, `addedAt` = now, `lastCapturedAt` = null.
5. Run a first capture immediately so the registry isn't empty (call the `--capture` flow for just this one post).
6. Confirm to the user: "Tracking @handle's [platform] post — first capture saved. Use `/track-posts [client] --replay [postId]` to view history once you've run a few more captures."

### `--add-from-campaign <campaign-name>`

1. Look up `campaignId` from the client's `client-brief.md` Bizkol Campaign Index. If not indexed, call `list_campaigns` once with no status filter and let the user pick. Campaign-level status (`draft`/`active`/etc.) is unreliable in merchant code — do not filter or display it. See `assets/campaign-kol-statuses.md`.
2. Call `get_campaign_kols` for the campaign.
3. For each KOL in a Confirmed cooperation status (`assets/campaign-kol-statuses.md`: `confirmation`, `appt_booked`, `store_visited`, `content_created`, `product_sent`, `product_received`, `content_draft`, `content_approved`, `published`, `completed`), call `get_kol_posts(kolId)` and filter to posts created on or after the campaign's `startDate`.
4. For each candidate post:
   - If `postId` already exists in the registry, skip (don't double-track).
   - Otherwise add to `posts[]` with `status: "active"` and the campaign's `campaignId` / `campaignSlug`.
5. Report: "Added X new posts (skipped Y already-tracked). Run `--capture` to take the first snapshot."
6. Do **not** auto-capture here — the user may want to review the list and prune first. Show the new additions.

### `--capture`

This is the workhorse — designed to be called by `/schedule` on a daily cadence.

1. Resolve which posts to capture: every `posts[]` entry where `status === "active"`.
2. Group by `kolId` to amortize follower lookups. For each KOL × platform combination, call `get_kol_performance` once and cache `followersAtCapture` for the run.
3. For each active post, call:
   ```
   Tool: get_social_post_info
   Parameters: { platform: "<platform>", identifier: "<url or shortcode>" }
   ```
4. Handle the response:
   - **Success**: compose a snapshot row per the kol-tracking skill schema and append to `/clients/[name]/raw-data/kol/post-history-[campaignSlug].ndjson` (use `post-history-adhoc.ndjson` for `campaignSlug === "adhoc"`). Update the registry entry's `lastCapturedAt`.
   - **404 / not found**: set the entry's `status: "deleted"` and `stopReason: "platform-404"` in the registry. Don't append a row.
   - **Other error / rate limit**: leave `status: "active"` so it retries next run. Log the error in the run summary.
5. After all posts: write the registry back atomically (write to `tracked-posts.json.tmp`, then rename).
6. Report: posts captured, deleted (auto-stopped), errored (will retry next run), total NDJSON rows appended, total file size.

### `--replay <postId-or-url-or-shortcode>`

1. Resolve the identifier against the registry to find the matching `postId` and `campaignSlug`. If multiple match (e.g., user passed a partial shortcode), ask which one.
2. Stream the matching NDJSON file, filter rows where `postId === <resolved id>`, sort by `timestamp` ascending.
3. If 0 rows: "No history yet for this post. Run `--capture` first."
4. If ≥ 1 row, compute and present:
   - **Headline**: post URL, handle, platform, total captures, first-seen → last-seen window, current engagement vs. peak engagement (with peak timestamp).
   - **Trajectory table**:

     | Timestamp (UTC) | Views | Likes | Comments | Shares | Engagement | ER% | Δ Engagement (vs prev) |
     |---|---|---|---|---|---|---|---|

   - **Sparkline** (if ≥ 8 captures): a 30-character bar over engagement using `▁▂▃▄▅▆▇█`, scaled to the min-max in the window. Skip if fewer than 8 rows.
   - **Milestones** (if the timeline allows): engagement at 24h / 7d / 30d post-publish, computed from `postedAt` and the closest captures.
5. Ask: "Save this replay to `/clients/[name]/audits/post-replay-[postId]-[YYYY-MM-DD].md`?" If yes, save the same content as a markdown file.

### `--list`

Read the registry. For each entry, also pull the latest matching NDJSON row to show "last-seen engagement" and the freshness of the data.

Present:

```
Tracked posts for [client-name] — N total (X active, Y paused, Z stopped, D deleted)

| Status | Handle | Platform | Posted | Last captured | Captures | Latest engagement | Campaign |
|--------|--------|----------|--------|---------------|----------|-------------------|----------|
```

Sort by `status` (active first), then by `lastCapturedAt` descending.

### `--pause` / `--resume` / `--stop`

Resolve the identifier in the registry, set the new `status`, write back. Confirm: "Status for @handle's [platform] post set to `[status]`."

## Step 4: Idempotency & Atomicity

- Registry writes use the temp-file-rename pattern so a crash mid-write doesn't corrupt the file.
- NDJSON appends use a single write call per row; on platforms where this isn't atomic for large files, append rows one at a time rather than buffering.
- A `--capture` run that's interrupted halfway through is fine: posts already captured have rows in the NDJSON, posts not yet captured will be picked up on the next run. The registry's `lastCapturedAt` is the truth for "what got done".

## Step 5: Schedule Pairing (Recommended Follow-up)

After the first `--capture`, suggest:

> "To keep history flowing without re-running manually, set up a daily capture:
> ```
> /schedule create --name 'track-posts-[client]' --cron '0 9 * * *' --command '/track-posts [client] --capture'
> ```
> The cadence guidance in the `kol-tracking` skill recommends every 6 hours for the first 48h post-publish, daily for days 2–14, then tapering. Adjust the cron as posts age, or run multiple schedules with different cadences for different campaigns."

Do not actually create the schedule from inside this command — the user should opt in via `/schedule` so the routine is theirs to manage.

## Output Summary

Every action ends with a one-paragraph summary of what changed:
- Files touched (registry path + NDJSON path + line count delta)
- Registry counts (active / paused / stopped / deleted) before → after
- Suggested next action

## Graceful degradation

- **If Bizkol MCP isn't connected:** This command can't run — every snapshot needs `get_social_post_info`. Inform the user.
- **If a post returns 404:** Auto-set `status: "deleted"`, log it, continue with the rest. The user sees this in the run summary.
- **If `get_kol_performance` fails for one KOL:** Use the most recent cached `followersAtCapture` from the NDJSON for that handle. If no cached value exists, write the row with `followersAtCapture: null` and rely on `engagementRate = engagementTotal / reach` instead.
- **If the NDJSON file grows past ~5 MB:** Suggest archiving older entries — `mv post-history-[slug].ndjson post-history-[slug]-archive-[YYYY-MM].ndjson` and start a fresh file. Replay reads both via globbing.
