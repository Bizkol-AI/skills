---
name: kol-outreach
description: Manage email outreach for a Bizkol campaign — paginated pull of every conversation (scales to 500+ KOLs), incremental triage of new replies since last run, draft follow-ups for new inbound, and persistent per-thread state so reruns only act on net-new activity. Suitable for both ad-hoc reviews and a scheduled daily routine.
argument-hint: [client-name] [campaign-name-or-id]
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, TodoWrite, mcp__bizkol__list_campaigns, mcp__bizkol__get_campaign, mcp__bizkol__get_campaign_kols, mcp__bizkol__list_email_conversations, mcp__bizkol__get_email_conversation, mcp__bizkol__update_campaign_kol
---

# /kol-outreach $ARGUMENTS

Review and progress KOL email outreach for one Bizkol campaign. Pulls every conversation (paginated, designed for 500+ KOL campaigns), focuses on **what's new since the last run**, drafts replies for new inbound, and persists per-thread state so the next run only acts on net-new activity. Sending always happens in Bizkol's UI — this command never sends.

The same command works for **ad-hoc** reviews and **daily** cadence. State makes it idempotent: running twice in a row produces the second run's much shorter "what changed" view, not a duplicate of yesterday's drafts.

## Step 0 — Prerequisite: Client Folder

Client folder must exist at `/clients/[client-name]/` with `client-brief.md`. If missing:

> "No client folder found for [client-name]. Run `/new-client [client-name]` first, then re-run this command."

Stop if missing.

## Step 1 — Resolve client and campaign

Parse `$ARGUMENTS`:

- **Two args** (`[client] [campaign]`): use as given.
- **One arg**: ambiguous. Try matching it as a client folder name under `/clients/`; if it's a single client with exactly one campaign indexed in their `client-brief.md`, use that campaign. Otherwise ask which client + campaign.
- **No args**: ask.

Resolve the campaign argument to a Bizkol `campaignId`:

1. Look up in `/clients/[client-name]/client-brief.md` Bizkol Campaign Index. If found, use the index's `campaignId`.
2. If not in the index, call `list_campaigns` twice (status `active` and `draft` — the param accepts a single value), merge results, mark each with its status, and ask the user to pick.

Derive `campaign-slug` from the campaign name (lowercased, spaces → hyphens) — used only for filenames.

## Step 2 — Load prior state

State file: `/clients/[client-name]/raw-data/kol/outreach-state-[campaign-slug].json`

Schema:

```json
{
  "campaignId": "<bizkol-id>",
  "campaignName": "<name>",
  "campaignSlug": "<slug>",
  "lastRunAt": "2026-05-02T18:30:00Z",
  "threads": {
    "<threadId>": {
      "kolHandle": "@foo",
      "kolId": "<id>",
      "lastInboundMessageId": "<id>",
      "lastInboundAt": "2026-05-02T15:00:00Z",
      "lastOutboundAt": "2026-04-30T10:00:00Z",
      "lastDraftedAt": "2026-05-01T18:30:00Z",
      "lastDraftedForMessageId": "<id>",
      "lastDraftFile": "outreach-acme-launch-2026-05-01.md",
      "draftStatus": "pending_send | sent | superseded | skipped",
      "decision": "negotiating | agreed | declined | no_response",
      "notes": "..."
    }
  }
}
```

If the file doesn't exist: this is the first run for this campaign. Initialize with `lastRunAt: null` and `threads: {}` and note **cold-start** in the report — the first day will produce drafts for the entire active inbox, which is normal.

If the user passes `--fresh`: archive the existing state file to `outreach-state-[slug]-archived-[timestamp].json` and start cold. (For when they want a clean re-triage.)

## Step 3 — Paginated pull of every conversation

`list_email_conversations` returns one page per call. For a 500-KOL campaign that's 5+ pages.

```
Tool: list_email_conversations
Parameters:
  campaignId: "<id>"
  status: "all"
  limit: 100
  # plus cursor/offset as the API exposes
```

Loop until a page returns fewer than `limit` items. Accumulate metadata in memory only — **do not write raw pages to disk** unless the user explicitly asks for an archival snapshot (which would land at `/clients/[name]/raw-data/kol/outreach-threads-[slug]-[YYYY-MM-DD].md`).

**Incremental optimization for non-cold-start runs:** if the API supports a `since` / `updatedAfter` filter, prefer that. Otherwise, if results are sorted by `lastMessageDate` descending, you may stop paginating once a page's oldest entry is older than `state.lastRunAt`. When sort behavior is uncertain, paginate fully — list calls are cheap relative to per-thread bodies.

For each conversation, extract the metadata you need to bucket without fetching a body: `threadId`, `kolId`, `kolHandle`, `lastMessageDate`, `lastMessageDirection`, `status`.

## Step 4 — Bucket against state

For each thread, compare to `state.threads[threadId]`:

| Bucket | Condition | Action |
|---|---|---|
| **New inbound** | `lastMessageDirection = inbound` AND (`lastMessageDate > state.lastRunAt` OR thread is brand new OR cold-start) | Fetch body, draft reply |
| **Drafted, awaiting your send** | `state.threads[id].lastDraftedForMessageId === current lastInboundMessageId` AND `draftStatus === "pending_send"` | **Do not redraft.** Add to carryover list. |
| **Drafted but inbound advanced** | `state.threads[id].lastDraftedAt` exists BUT `current lastInboundMessageId !== state.lastDraftedForMessageId` | Fetch body, redraft. Mark prior draft `superseded`. |
| **Stale outbound** | `lastMessageDirection = outbound`, `lastMessageDate` >5 days ago, no inbound since, AND no nudge drafted in last 3 days per state | Optional nudge draft |
| **Concluded** | `decision in (agreed, declined)` OR thread `status` indicates closure | Update state; no draft |
| **Quiet** | Everything else | No action; no fetch |

**Only fetch full thread bodies for "New inbound", "Drafted but inbound advanced", and "Stale outbound" threads** — that's the expensive call. On a normal day this is typically 20–80 threads even for a 500-KOL campaign, not 500.

## Step 5 — Fetch bodies for action threads (parallel batches)

Run `get_email_conversation` calls in parallel batches of ~5–10:

```
Tool: get_email_conversation
Parameters:
  threadId: "<id>"
  campaignId: "<id>"
  kolId: "<id>"
```

If a thread fails to load, log the threadId in the report's "Errors" section and continue. Do **not** update state for failed threads — tomorrow's run will retry them.

## Step 6 — Draft replies

Single drafts file per run:

`/clients/[client-name]/content/drafts/outreach-[campaign-slug]-[YYYY-MM-DD].md`

For each "New inbound" and "Drafted but inbound advanced" thread, append a section:

```markdown
## [@handle] — [topic / last subject]
**Bizkol threadId:** [id] | **kolId:** [id]
**Their last message:** [timestamp] — [one-line summary]
**Flag:** [new since last run | redraft — inbound advanced since [prior draft date]]

> [2–4 sentences of their actual message, quoted, so the user has context without opening Bizkol]

### Suggested reply
Hi [name],

[draft body — match the thread's tone, address what they actually asked, include the next concrete step]

Best,
[Sender]

### Action notes
- [What to confirm before sending: price, deliverables, deadline]
- [If they agreed to terms, recommended `update_campaign_kol` payload:]
  ```json
  {
    "campaignId": "...",
    "kolId": "...",
    "status": "agreed",
    "quotes": [...],
    "notes": "..."
  }
  ```
```

For "Stale outbound" threads, append a shorter "Nudge" section (no full quote needed — a 2-line polite check-in is enough).

**Carryover threads** (already drafted, still pending send): do **not** redraft. List them in the report with a pointer to the original draft file.

> **Sending happens in Bizkol's UI.** This command only drafts. If a KOL has committed to terms, surface the `update_campaign_kol` payload so the user can persist it in one action.

## Step 7 — Review report

`/clients/[client-name]/audits/kol-outreach-[campaign-slug]-[YYYY-MM-DD].md`:

```markdown
# Outreach Review — [campaign]
**Date:** [YYYY-MM-DD] | **campaignId:** [id]
**Run window:** [state.lastRunAt → now] | **Cold start:** [yes / no]

## At a glance
- Total threads in campaign: [N]
- New inbound since last run: [N]
- New drafts produced today: [N]
- Carryover drafts (drafted prior runs, still pending send): [N]
- Stale outbound (>5 days no reply): [N]
- Concluded since last run: [N agreed / N declined]

## Action queue (do these first)
1. **[@handle]** — [one-line: what they said + what to send] → see draft in `outreach-[slug]-[date].md`
2. ...

## Carryover — drafted previously, awaiting your send
| KOL | Drafted on | Days waiting | Draft file |
|---|---|---|---|
| @foo | 2026-04-30 | 3 | outreach-[slug]-2026-04-30.md |

## Stale outbound — consider nudging
| KOL | Last sent | Days silent | Nudge draft? |
|---|---|---|---|
| @bar | 2026-04-25 | 8 | yes (in today's drafts) |

## Concluded since last run
- **@baz** — agreed: $X for Y deliverables. Suggested `update_campaign_kol` payload below.
- **@qux** — declined.

## Suggested Bizkol updates
[For threads where the KOL committed to terms, list the `update_campaign_kol` payloads to apply.]

## Errors
[threadIds that failed to load, if any]
```

## Step 8 — Persist updated state

Rewrite `/clients/[client-name]/raw-data/kol/outreach-state-[campaign-slug].json` (pretty-printed, 2-space indent so it's diffable):

For every thread surfaced in Step 3:

- Update `lastInboundMessageId`, `lastInboundAt`, `lastOutboundAt` from the latest list response.
- For threads drafted this run: set `lastDraftedAt = now`, `lastDraftedForMessageId = current lastInboundMessageId`, `lastDraftFile = outreach-[slug]-[YYYY-MM-DD].md`, `draftStatus = "pending_send"`.
- For threads where state had `pending_send` AND `lastOutboundAt > state.lastDraftedAt`: the user sent it — flip `draftStatus` to `"sent"`.
- For threads where the inbound advanced past a prior draft: mark prior `draftStatus = "superseded"` (the new draft entry overwrites with `pending_send`).
- Update `decision` if concluded (`agreed` / `declined`).
- Set top-level `lastRunAt = now` (ISO 8601 UTC).

For threads that failed to fetch in Step 5: do **not** update their state — tomorrow's run will retry.

## Step 9 — Print summary

After writing files:

```
✅ /kol-outreach — [campaign name] — [YYYY-MM-DD]

[N] new inbound, [N] drafts produced, [N] carryovers awaiting your send.
Top priority: [@handle] — [one-line]

Drafts: clients/[name]/content/drafts/outreach-[slug]-[date].md
Report: clients/[name]/audits/kol-outreach-[slug]-[date].md
```

## Cadence

This command is idempotent and safe to run on any cadence:

- **Ad-hoc**: run whenever you want a snapshot of what needs your attention.
- **Daily routine**: register on a cron via `/schedule` (e.g., 8am weekdays). State ensures you only see net-new work each morning.
- **Multiple runs in one day**: safe — the second run uses the first run's `lastRunAt` as its cutoff, so it only surfaces what arrived between runs.

## Graceful degradation

- **Bizkol MCP not connected:** stop and tell the user; nothing else is possible.
- **State file corrupted / unreadable:** rename to `outreach-state-[slug]-broken-[timestamp].json` and start cold. Note in the report.
- **A page of `list_email_conversations` fails:** retry once; if still failing, capture what you have, note the gap in the report's "Errors" section, and continue. Don't lose state from threads already processed.
- **`get_email_conversation` fails for a single thread:** skip it, log to "Errors", leave its state untouched so the next run retries.

## Output paths summary

| File | Purpose |
|---|---|
| `/clients/[name]/raw-data/kol/outreach-state-[slug].json` | Persistent per-thread state |
| `/clients/[name]/content/drafts/outreach-[slug]-[YYYY-MM-DD].md` | Today's drafts (new + redrafts + nudges) |
| `/clients/[name]/audits/kol-outreach-[slug]-[YYYY-MM-DD].md` | Today's review report |

## Related

- `/kol-campaign [campaign]` — create/manage the campaign
- `/kol-performance [campaign]` — performance dashboard for shipped KOLs
- `/schedule` — register `/kol-outreach` on a daily cron
