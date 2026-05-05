---
name: kol-outreach
description: Manage email outreach for a Bizkol campaign — paginated pull of every conversation (scales to 500+ KOLs), incremental triage of new replies since last run, draft follow-ups for new inbound, and persistent per-thread state so reruns only act on net-new activity. Suitable for both ad-hoc reviews and a scheduled daily routine.
argument-hint: [client-name] [campaign-name-or-id]
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, TodoWrite, mcp__bizkol__list_campaigns, mcp__bizkol__get_campaign, mcp__bizkol__get_campaign_kols, mcp__bizkol__list_email_conversations, mcp__bizkol__get_email_conversation, mcp__bizkol__update_campaign_kol
---

# /kol-outreach $ARGUMENTS

Review and progress KOL email outreach for one Bizkol campaign. Pulls every conversation (paginated, designed for 500+ KOL campaigns), focuses on **what's new since the last run**, drafts replies for new inbound, and persists per-thread state so the next run only acts on net-new activity. Sending always happens in Bizkol's UI — this command never sends.

**Designed as a daily decision dashboard.** The primary output is a self-contained HTML report at `/clients/[name]/audits/kol-outreach-[slug]-[YYYY-MM-DD].html` that the user opens once per morning, scans, and acts from. Every action item embeds the suggested reply inline with a copy button and a click-through link to the Bizkol thread, so the user does not need to flip between files to act.

The same command works for **ad-hoc** reviews and **daily** cadence. State makes it idempotent: running twice in a row produces the second run's much shorter "what changed" view, not a duplicate of yesterday's drafts.

## Bizkol thread URL — MUST be a real, clickable link

Every output that mentions a `threadId` (drafts, review report, per-KOL profile, summary, Coverage diagnostics) MUST include a fully-substituted, clickable markdown link to the Bizkol email UI. The user reads these reports inline and clicks the link to jump to the thread — a placeholder string is useless and counts as a bug.

**URL template (with literal placeholders for documentation only):**

```
https://biz.bizkol.ai/en/emails?threadId={THREAD_ID}&campaignId={CAMPAIGN_ID}&kolId={KOL_ID}
```

**Substitution is mandatory.** Replace `{THREAD_ID}`, `{CAMPAIGN_ID}`, `{KOL_ID}` with the actual values from `list_email_conversations` / `get_email_conversation` *before* writing the URL to any file. Never emit `<threadId>`, `{THREAD_ID}`, `...`, or any other placeholder text in a final output — those forms exist only inside this skill document as templates.

**Worked example.** For the conversation `{ threadId: "thread_bbd72ad9-4594-411f-9066-6f97ac58d983_20c2dfb8-9013-45db-a670-1cea4d2e4f7d_1777665204810", campaignId: "bbd72ad9-4594-411f-9066-6f97ac58d983", kolId: "20c2dfb8-9013-45db-a670-1cea4d2e4f7d" }`, the rendered link must be:

```markdown
[Open in Bizkol →](https://biz.bizkol.ai/en/emails?threadId=thread_bbd72ad9-4594-411f-9066-6f97ac58d983_20c2dfb8-9013-45db-a670-1cea4d2e4f7d_1777665204810&campaignId=bbd72ad9-4594-411f-9066-6f97ac58d983&kolId=20c2dfb8-9013-45db-a670-1cea4d2e4f7d)
```

That is what the user actually sees in the report and clicks.

**Render conventions:**
- Markdown link text: `Open in Bizkol →` (or `Open →` inside table cells where space is tight).
- Inline next to a threadId, separate with ` · `.
- In tables, give the link its own column titled `Bizkol thread`.

**Fallback rules:**
- All three IDs present: render the full link.
- Any ID missing or null: do **not** invent values and do **not** emit a broken URL. Print just the available `threadId` (or `kolId` if that's all you have) as plain text and add `· no-link` so the user knows why no link rendered.

**Self-check before saving any output file:** the rendered text must contain zero occurrences of `<threadId>`, `<campaignId>`, `<kolId>`, `{THREAD_ID}`, `{CAMPAIGN_ID}`, `{KOL_ID}`, or `...` inside any URL. If any of those slip through, the substitution failed — fix and re-render.

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
2. If not in the index, call `list_campaigns` once with no status filter (or `status: "all"` if required by the tool). **Do not filter by campaign-level status** — campaign status is not used reliably in the merchant code today. List every result and ask the user to pick. Do not surface the campaign-status field to the user.

After resolving, also call `get_campaign(campaignId)` and capture `workflowTemplate` (one of `local_store`, `ecommerce_product`, or `null`). The valid `CampaignKOL.status` enum depends on this — see `assets/campaign-kol-statuses.md` for the canonical list. The skill MUST consult that file before proposing any status change in Step 9.

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
      "outreachOutcome": "no_reply | replied | replied_with_conditions | committed | declined",
      "notes": "..."
    }
  }
}

> `outreachOutcome` is **local-only state** describing what the skill detected in the email thread. It is NOT a Bizkol KOL status. Mapping from `outreachOutcome` to a real Bizkol KOL status is done in Step 9 using the rules in `assets/campaign-kol-statuses.md` and depends on the campaign's `workflowTemplate`.
```

Per-thread state must include enough to detect new activity by **three independent signals** (Step 4 uses all three). Add these fields when writing state:

- `lastInboundMessageId` (existing)
- `lastInboundAt` (existing) — same as `lastMessageAt` on the most recent inbound message
- `messageCount` — total messages in the thread at the time state was last written
- `processedMessageIds` — array of every messageId already extracted into the per-KOL profile (used by Step 6.5 for per-message idempotency)
- `skillVersion` — version of the skill that wrote this state (single string, e.g., `"2.0"`)

Top-level state must include:
- `lastRunAt` (existing)
- `lastFullPaginationAt` — last time we walked the entire `list_email_conversations` (every run, but recorded explicitly for diagnostics)
- `skillVersion`

If the file doesn't exist: this is the first run for this campaign. Initialize with `lastRunAt: null`, `threads: {}`, and the current `skillVersion`. Note **cold-start** in the report — the first day will produce drafts for the entire active inbox, which is normal.

### Run modes

| Flag | Behavior |
|---|---|
| (none) | Default incremental run. Full pagination + 6h overlap window + body-fetch only for threads showing new activity. |
| `--fresh` | Archive existing state to `outreach-state-[slug]-archived-[timestamp].json` and start cold. Use when you want a clean re-triage. |
| `--full-rescan` | Keep existing state for idempotency, but ignore the bucket filter in Step 4 — fetch bodies for **every** thread and re-run profile extraction. Use after the skill is upgraded or you suspect older threads have terms that weren't extracted. Expensive; expect 5–10× the body-fetch cost of a normal run. |
| `--rescan-since YYYY-MM-DD` | Like `--full-rescan` but limited to threads with `lastMessageAt >= date`. Cheaper middle ground. |

### Coverage guarantee (what this skill promises)

> **Each new message in each thread gets at least one explicit profile-extraction pass.** Quiet threads (no new activity since last run) are not re-analyzed unless `--full-rescan` or `--rescan-since` is passed. State tracks `processedMessageIds` per thread so that re-runs do not produce duplicate negotiation-log entries and never silently skip a message that arrived during a previous run's failure.

## Step 3 — Paginated pull of every conversation (always full)

`list_email_conversations` returns one page per call. For a 500-KOL campaign that's 5+ pages.

```
Tool: list_email_conversations
Parameters:
  campaignId: "<id>"
  status: "all"
  limit: 100
  nextToken: "<cursor from prior page>"
```

**Always paginate until exhausted** — loop until a page returns `nextToken: null` or fewer than `limit` items. **Do not** use a `lastRunAt` early-exit shortcut: list ordering can race with new messages mid-run, and missing a thread because we trusted server-side sort is the worst possible failure mode for this skill. List calls are cheap; body fetches are the expensive ones, and we're already filtering those in Step 4.

Accumulate metadata in memory only — **do not write raw pages to disk** unless the user explicitly asks for an archival snapshot (which would land at `/clients/[name]/raw-data/kol/outreach-threads-[slug]-[YYYY-MM-DD].md`).

For each conversation, extract: `threadId`, `kolId`, `kolHandle`, `lastMessageAt`, `lastMessageSender`, `messageCount`, `unreadCount`, `status`.

If a page-fetch fails, retry once with exponential backoff. If still failing, capture what loaded so far, stop paginating, and write a top-level `paginationIncomplete: true` flag to state plus a "Coverage diagnostics" entry to the report. Do not silently truncate — if pagination broke, the user must know.

## Step 4 — Bucket against state (triple-check freshness)

A thread counts as having **new activity** if **any** of these differ from prior state — belt + suspenders + zipper, so a single ID-format change or clock skew can't silently hide a new message:

1. `lastInboundMessageId` differs from `state.threads[id].lastInboundMessageId`
2. `messageCount` is greater than `state.threads[id].messageCount`
3. `lastMessageAt` is greater than `state.threads[id].lastInboundAt` AND `lastMessageSender = "kol"`

Plus, regardless of state, treat as new activity any thread whose `lastMessageAt` falls inside the **6-hour overlap window** before `now`. This absorbs eventual-consistency races where a message arriving seconds before the run hasn't fully propagated to `list_email_conversations` and would otherwise be missed until tomorrow. Cheap insurance — typically only a handful of threads.

For each thread, compare to `state.threads[threadId]` and bucket:

| Bucket | Condition | Action |
|---|---|---|
| **New inbound** | "new activity" (above) AND `lastMessageSender = "kol"` | Fetch body, draft reply, run extraction |
| **Drafted, awaiting your send** | `state.threads[id].lastDraftedForMessageId === current lastInboundMessageId` AND `draftStatus === "pending_send"` AND no new activity | **Do not redraft.** Add to carryover list. |
| **Drafted but inbound advanced** | `state.threads[id].lastDraftedAt` exists BUT current `lastInboundMessageId !== state.lastDraftedForMessageId` | Fetch body, redraft. Mark prior draft `superseded`. Run extraction. |
| **Stale outbound** | `lastMessageSender = "user"`/`"sender"`, `lastMessageAt` >5 days ago, no inbound since, AND no nudge drafted in last 3 days per state | Optional nudge draft |
| **Overlap window** | `lastMessageAt >= now - 6h` AND not already in another bucket | Fetch body (eventual-consistency safety net) |
| **Concluded** | `outreachOutcome in (committed, declined)` OR conversation `status = "closed"` | Update state; no draft |
| **Quiet** | Everything else | No action; no fetch |

**Body-fetch list** = New inbound ∪ Drafted-but-inbound-advanced ∪ Stale outbound ∪ Overlap window. Plus, if `--full-rescan` or `--rescan-since` is set, expand the body-fetch list per Step 2's rules (overrides Quiet for matching threads).

On a normal day the body-fetch list is typically 20–80 threads even for a 500-KOL campaign, not 500.

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

**Long-thread guard.** If a fetched thread has `messageCount > 15`, do not pass it to extraction in one shot — attention budget compresses long threads and quietly drops middle messages. Process the thread in **batches of 10 messages each**, oldest first, and treat the per-message extraction in Step 6.5 as if each batch were its own thread (same protocol, same idempotency key on `messageId`). Merge findings before composing the profile update.

## Step 6 — Draft replies

Single drafts file per run:

`/clients/[client-name]/content/drafts/outreach-[campaign-slug]-[YYYY-MM-DD].md`

For each "New inbound" and "Drafted but inbound advanced" thread, append a section:

```markdown
## [@handle] — [topic / last subject]
**Bizkol threadId:** [id] | **kolId:** [id] · [Open in Bizkol →](https://biz.bizkol.ai/en/emails?threadId=<threadId>&campaignId=<campaignId>&kolId=<kolId>)
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
- [If a status change is warranted, propose an `update_campaign_kol` payload using the canonical enum. Map per `assets/campaign-kol-statuses.md`:]
  - KOL replied positively → `interested`
  - KOL agreed to specific terms with conditions → `conditionally_approved`
  - KOL fully agreed and merchant approved → `approved`
  - KOL declined → `cancelled`
- [If you're not certain which status applies, omit `status` from the payload and update only `quotes` and `notes`.]
  ```json
  {
    "campaignId": "...",
    "kolId": "...",
    "status": "interested",
    "quotes": [...],
    "notes": "..."
  }
  ```
```

For "Stale outbound" threads, append a shorter "Nudge" section (no full quote needed — a 2-line polite check-in is enough).

**Carryover threads** (already drafted, still pending send): do **not** redraft. List them in the report with a pointer to the original draft file.

> **Sending happens in Bizkol's UI.** This command only drafts. If a KOL has committed to terms, surface the `update_campaign_kol` payload so the user can persist it in one action.

## Step 6.5 — Update per-KOL profile files (enumerate-then-extract)

For every thread whose body was fetched in Step 5, extract negotiation facts and persist them to a per-KOL profile. **Use the explicit per-message protocol below — do not summarize the thread holistically. Holistic summarization silently drops mid-thread terms.**

**Path:** `/clients/[client-name]/raw-data/kol/profiles/[kol-handle].md`

(Filename uses the KOL's primary social handle, lowercased, no `@`. If multiple platforms, use the campaign's primary platform handle. The file is per-client; the same KOL working with two clients gets two files — that's intentional, scope stays clean.)

### Extraction protocol (every fetched thread)

For each thread body, run these substeps **in order** — do not collapse or skip:

**6.5a — Enumerate every message.** Build a numbered table of every message in the thread:

| # | messageId | timestamp | sender (kol/user) | direction (inbound/outbound) | has_attachment | ≤80-char summary |
|---|---|---|---|---|---|---|

If `messageCount > 15`, batch by 10 messages oldest-first per Step 5's long-thread guard, completing 6.5a–6.5d on each batch before merging.

**6.5b — Per-message extraction pass.** For each row in the enumerated table, ask explicitly: *"Does this message introduce, change, or confirm any of: price · deliverables · payment terms · usage rights · exclusivity · timeline · special conditions · decision (agree/decline/counter)?"* Capture each finding keyed to that message's `messageId`. A single message may produce zero or multiple findings.

**6.5c — Idempotency check.** Read the existing profile file (if any) and collect the set of `messageId`s that already appear in the Negotiation log. For every finding from 6.5b: if its `messageId` is already in the log, **skip** appending a new entry (the existing one is the source of truth). Only append entries for `messageId`s not yet logged. This makes re-runs and `--full-rescan` safe.

**6.5d — Unreadable content.** If a message contains an attachment / inline image / external doc that you cannot read (PDF, PNG rate card, linked Notion doc), do **not** silently skip. Append a log entry tagged `extraction: unreadable` with what is known (sender, timestamp, attachment count, any caption text). The user will see it in the report and can open it manually.

**6.5e — Recompute Current state.** After all log entries are appended, walk the full Negotiation log newest-to-oldest and compute the `Current state` block by picking the first non-null value per field (latest-wins for state; first-mention is preserved in the log). Always update `Last updated`. Do not let `--full-rescan` or new findings from older messages downgrade a more recent outcome (e.g., a re-extracted "interested" finding from message #3 must not overwrite an "approved" outcome from message #18).

**6.5f — Coverage assertion.** Emit a one-line summary of the extraction for this thread, to be included in the daily review report's Coverage diagnostics:

> Thread `<threadId>` — walked N/N messages · L log entries appended · S already logged (skipped) · U unreadable · profile updated: yes/no.

If `walked < messageCount` (i.e., 6.5a missed messages), flag it loudly in the report — that's a bug, not a feature.

### What to look for in each message (used during 6.5b)

| Field | Look for |
|---|---|
| **Quoted price** | Numeric values with currency near words like "rate", "fee", "price", "cost", "$", "USD", "€", "£" — both KOL-stated rates and your offers |
| **Deliverables** | Number + format (e.g., "1 dedicated video", "2 IG reels + 3 stories", "8-min long-form review") |
| **Payment terms** | "50% upfront", "net-30", "on publish", "PayPal", "wire", "invoice required" |
| **Usage rights** | "whitelisting", "paid amplification", "30-day exclusive", "perpetual", "organic only" |
| **Exclusivity** | "no competing brands for X days", category exclusions |
| **Timeline** | turnaround days, publish date asks, deadlines |
| **Special conditions** | brand asks/refusals, legal requirements, content restrictions, contract preferences |
| **Decision events** | KOL agrees / declines / counters / goes quiet |

When in doubt, capture it — better to log a maybe-relevant detail than lose it.

### Profile file schema

If the file doesn't exist, create it with this template. If it exists, **update the "Current state" block in place** and **append a new dated entry to the Negotiation log** — never overwrite history.

```markdown
# KOL Profile — @[handle] ([Display Name])

**kolId:** `<id>`
**Platforms:** [primary platform + handle, plus any others from kolData.socialAccounts]
**First contacted:** [YYYY-MM-DD — earliest message in any thread]
**Last updated:** [YYYY-MM-DD — this run]
**Active campaigns:** [Campaign Name (`<campaignId>`), …]

## Current state
- **Bizkol status:** [exact value from `CampaignKOL.status`, validated against the workflowTemplate's enum in `assets/campaign-kol-statuses.md`. Examples: `identified`, `contacted`, `interested`, `approved`, `conditionally_approved`, `cancelled`, `published`, `completed`. Do NOT invent values.]
- **Outreach outcome (local):** no_reply | replied | replied_with_conditions | committed | declined *(local interpretation; not a Bizkol field)*
- **Quoted price:** $X for [deliverables] *(as of YYYY-MM-DD)*
- **Payment terms:** [e.g., 50% upfront, 50% on publish]
- **Usage rights:** [e.g., 30-day whitelisting allowed; organic only after]
- **Exclusivity:** [e.g., 60-day category exclusivity in B2B sales tools]
- **Timeline:** [e.g., 14-day turnaround once contract signed]
- **Special conditions:** [free-form: legal, content, scheduling]
- **Next step:** [what's blocking — e.g., "awaiting their countersign", "you owe a reply"]

## Negotiation log

### YYYY-MM-DD HH:mm — [event headline, e.g., "KOL countered with $1,200"]
- **messageId:** `<id>` *(idempotency key — one log entry per messageId, ever)*
- **Source:** thread `<threadId>` · campaign `[campaign name]` · message direction: inbound|outbound · [Open in Bizkol →](https://biz.bizkol.ai/en/emails?threadId=<threadId>&campaignId=<campaignId>&kolId=<kolId>)
- **Quoted:** $X for [deliverables]
- **Payment:** [terms]
- **Usage rights:** [...]
- **Exclusivity:** [...]
- **Timeline:** [...]
- **Conditions:** [...]
- **Excerpt:** "[1–3 sentence quote from the message that's the source of truth for this entry]"

### [earlier entries below — never delete or rewrite older entries]
```

### Update rules

1. **First time you write a profile for a KOL:** populate `First contacted` from the earliest message timestamp in the enumerated table; populate `Active campaigns` with this campaign.
2. **Negotiation log is per-message, not per-run.** A single run may append zero, one, or many log entries for a given thread depending on how many messages produced findings in 6.5b. The `messageId` is the idempotency key — one log entry per messageId, ever. Do not edit prior entries.
3. **Current state recompute (6.5e):** rebuild the block every run by walking the log newest-to-oldest. Latest non-null per field wins. Always update `Last updated`.
4. **Multi-campaign KOLs:** if a profile already exists from another campaign and this campaign isn't yet listed under `Active campaigns`, add it. Log entries always tag which campaign they came from, so cross-campaign history stays readable in one file.
5. **Status field:** the profile's `Bizkol status` field mirrors what's currently in `CampaignKOL.status` (read from `get_campaign_kols`). The profile's `Outreach outcome (local)` field reflects the latest message-based interpretation. Never let a re-extracted older message downgrade a more recent outcome — log entries are timestamped per message, and the latest interpretation wins. Both fields are validated against the canonical enum in `assets/campaign-kol-statuses.md` before write.
6. **Privacy:** quote excerpts must be ≤3 sentences. Don't paste full message bodies — those stay in Bizkol.
7. **State sync:** after all log appends, update `state.threads[id].processedMessageIds` to the union of (prior set) ∪ (messageIds processed this run). Step 8 persists this; future runs skip these messageIds.

### When extraction is ambiguous

If a price or term is implied but not explicit (e.g., "we usually charge our standard rate" with no number), record it as a free-form line in `Special conditions` rather than guessing a number. Use `[unspecified]` sparingly and only where the user would benefit from knowing the gap exists.

## Step 7 — Review report (HTML, daily decision dashboard)

**Path:** `/clients/[client-name]/audits/kol-outreach-[campaign-slug]-[YYYY-MM-DD].html`

The report is the user's daily morning artifact. They open it in a browser, scan, and act — without flipping between files. Design principles:

1. **Action-first layout.** Action queue at the top, informational sections below. Coverage diagnostics is collapsed by default (debug-only).
2. **Self-contained file.** Inline CSS + minimal inline JS for copy-to-clipboard. No external assets, no build step, no network requests at view time. The file works opened from disk in any modern browser.
3. **Embed every draft inline.** Each action queue card contains the KOL's last message (quoted), the full suggested reply (in a `<pre>` block with a Copy button), action notes, and the suggested `update_campaign_kol` payload (collapsible, with a Copy button). Reading the report is sufficient — no need to open the drafts file.
4. **One-click affordances.** Every Bizkol thread link is fully substituted (per the "Bizkol thread URL" section above). Every draft and payload has a Copy button using `navigator.clipboard.writeText`. Failure path: button shows `Copy failed — select & copy manually` for 2s.
5. **Status badges use canonical enum only** (`assets/campaign-kol-statuses.md`). Never invent.

### HTML template

Render this template by substituting the `{{...}}` placeholders. Iterate the action / carryover / stale / concluded / profile blocks as needed. Keep the CSS and JS exactly as written below — they are tested for inline-rendering and clipboard reliability.

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Outreach Review — {{campaignName}} — {{runDate}}</title>
<style>
  :root { color-scheme: light; }
  * { box-sizing: border-box; }
  body { font: 14px/1.55 -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; max-width: 1024px; margin: 32px auto; padding: 0 24px 64px; color: #1a1a1a; background: #fff; }
  h1 { font-size: 24px; margin: 0 0 4px; }
  h2 { font-size: 18px; margin: 36px 0 12px; padding-bottom: 6px; border-bottom: 1px solid #e5e5e5; }
  h3 { font-size: 15px; margin: 0 0 8px; }
  h4 { font-size: 13px; margin: 12px 0 6px; color: #555; text-transform: uppercase; letter-spacing: 0.04em; }
  .meta { color: #666; font-size: 13px; margin-bottom: 20px; }
  code { background: #f3f3f3; padding: 1px 6px; border-radius: 4px; font: 12.5px/1.4 ui-monospace, "SF Mono", Menlo, monospace; }
  pre { background: #f7f7f8; border: 1px solid #ececef; padding: 12px 14px; border-radius: 6px; overflow-x: auto; white-space: pre-wrap; word-break: break-word; font: 13px/1.5 ui-monospace, "SF Mono", Menlo, monospace; margin: 6px 0; }
  table { border-collapse: collapse; width: 100%; margin: 8px 0; font-size: 13px; }
  th, td { text-align: left; padding: 8px 10px; border-bottom: 1px solid #eee; vertical-align: top; }
  th { background: #fafafa; font-weight: 600; }
  a { color: #0b5fff; text-decoration: none; }
  a:hover { text-decoration: underline; }
  .glance { display: flex; flex-wrap: wrap; gap: 10px 24px; padding: 14px 18px; background: #f7f9fc; border-radius: 8px; margin-bottom: 8px; }
  .glance .stat { display: flex; flex-direction: column; }
  .glance .stat-label { font-size: 11px; color: #666; text-transform: uppercase; letter-spacing: 0.04em; }
  .glance .stat-value { font-size: 18px; font-weight: 600; }
  .badge { display: inline-block; padding: 2px 8px; border-radius: 12px; font-size: 11px; font-weight: 500; line-height: 1.4; vertical-align: middle; }
  .badge-new { background: #e8f5e9; color: #2e7d32; }
  .badge-redraft { background: #fff8e1; color: #b07c00; }
  .badge-stale { background: #ffebee; color: #c62828; }
  .badge-status-identified { background: #f1f3f5; color: #555; }
  .badge-status-shortlisted { background: #e0f7fa; color: #00838f; }
  .badge-status-contacted { background: #e3f2fd; color: #1565c0; }
  .badge-status-interested { background: #e1f5fe; color: #0277bd; }
  .badge-status-approved { background: #ede7f6; color: #5e35b1; }
  .badge-status-conditionally_approved { background: #fff8e1; color: #b07c00; }
  .badge-status-cancelled { background: #ffebee; color: #c62828; }
  .badge-status-confirmation { background: #e0f2f1; color: #00695c; }
  .badge-status-published { background: #e8f5e9; color: #2e7d32; }
  .badge-status-completed { background: #d0f4d8; color: #1b5e20; }
  .action { border-left: 3px solid #0b5fff; padding: 14px 18px; margin: 14px 0; background: #fbfcfe; border-radius: 6px; }
  .action.priority-1 { border-left-color: #d32f2f; background: #fffaf9; }
  .action h3 { margin-top: 0; font-size: 16px; }
  .action .chip { font-size: 12px; color: #555; }
  .their-msg { color: #333; border-left: 2px solid #ddd; padding: 6px 12px; margin: 8px 0; font-style: italic; background: #fafafa; }
  .copy-wrap { position: relative; }
  .copy-btn { position: absolute; top: 6px; right: 6px; padding: 3px 10px; font-size: 11px; background: #fff; border: 1px solid #d0d0d4; border-radius: 4px; cursor: pointer; color: #333; }
  .copy-btn:hover { background: #f3f3f3; }
  .copy-btn.copied { background: #e8f5e9; border-color: #2e7d32; color: #2e7d32; }
  details { margin: 8px 0; }
  details > summary { cursor: pointer; padding: 4px 0; user-select: none; color: #0b5fff; font-size: 13px; }
  ul.tight { margin: 6px 0; padding-left: 22px; }
  ul.tight li { margin: 2px 0; }
  .small { font-size: 12px; color: #666; }
  .empty { color: #999; font-style: italic; padding: 10px 0; }
</style>
</head>
<body>

<h1>Outreach Review — {{campaignName}}</h1>
<p class="meta">{{runDate}} · campaignId <code>{{campaignId}}</code> · workflowTemplate <code>{{workflowTemplate or "null"}}</code> · run window {{lastRunAt}} → {{now}} · cold-start: {{yes/no}}</p>

<div class="glance">
  <div class="stat"><span class="stat-label">Awaiting your reply</span><span class="stat-value">{{newInbound}}</span></div>
  <div class="stat"><span class="stat-label">Drafts ready</span><span class="stat-value">{{draftsToday}}</span></div>
  <div class="stat"><span class="stat-label">Carryover (pending send)</span><span class="stat-value">{{carryover}}</span></div>
  <div class="stat"><span class="stat-label">Stale outbound (&gt;5d)</span><span class="stat-value">{{stale}}</span></div>
  <div class="stat"><span class="stat-label">Concluded today</span><span class="stat-value">{{nApproved}} / {{nCancelled}}</span></div>
  <div class="stat"><span class="stat-label">Total threads</span><span class="stat-value">{{totalThreads}}</span></div>
</div>

<h2>Action queue — do these first</h2>
<p class="small">Drafts are embedded inline. <strong>Sending happens in Bizkol</strong> — copy from the "Suggested reply" block, then open the thread.</p>

<!-- Repeat one .action block per item. The first one carries .priority-1 if there's a clear top priority. -->
<div class="action priority-1" id="action-1">
  <h3>1. @{{handle}} — {{topicLine}} <span class="badge badge-status-{{currentStatus}}">{{currentStatus}}</span> <span class="badge badge-new">new since last run</span></h3>
  <p class="chip">threadId <code>{{threadId}}</code> · kolId <code>{{kolId}}</code> · <a href="https://biz.bizkol.ai/en/emails?threadId={{threadId}}&amp;campaignId={{campaignId}}&amp;kolId={{kolId}}">Open in Bizkol →</a></p>

  <h4>Their last message — {{messageTimestamp}}</h4>
  <blockquote class="their-msg">{{kolMessageQuote}}</blockquote>

  <h4>Suggested reply</h4>
  <div class="copy-wrap">
    <button class="copy-btn" data-copy-target="#draft-1">Copy</button>
    <pre id="draft-1">Hi {{kolName}},

{{draftBody}}

Best,
{{senderName}}</pre>
  </div>

  <h4>Action notes</h4>
  <ul class="tight">
    <li>{{actionNote1}}</li>
    <li>{{actionNote2}}</li>
  </ul>

  <details>
    <summary>Suggested update_campaign_kol payload</summary>
    <div class="copy-wrap">
      <button class="copy-btn" data-copy-target="#payload-1">Copy</button>
      <pre id="payload-1">{
  "campaignId": "{{campaignId}}",
  "kolId": "{{kolId}}",
  "status": "{{proposedStatus}}",
  "quotes": {{quotesJsonOrNull}},
  "notes": "{{notesString}}"
}</pre>
    </div>
    <p class="small">Status is validated against <code>assets/campaign-kol-statuses.md</code>. If validation skipped this KOL, only <code>quotes</code> / <code>notes</code> are included above.</p>
  </details>
</div>

<!-- ... more .action blocks, one per item in priority order ... -->

<h2>Carryover — drafted previously, awaiting your send</h2>
<table>
  <thead><tr><th>KOL</th><th>Drafted</th><th>Days waiting</th><th>Draft file</th><th>Bizkol thread</th></tr></thead>
  <tbody>
    <tr><td>@{{handle}}</td><td>{{draftedOn}}</td><td>{{daysWaiting}}</td><td><code>outreach-{{slug}}-{{draftDate}}.md</code></td><td><a href="https://biz.bizkol.ai/en/emails?threadId={{threadId}}&amp;campaignId={{campaignId}}&amp;kolId={{kolId}}">Open →</a></td></tr>
  </tbody>
</table>

<h2>Stale outbound — consider nudging</h2>
<p class="small">Threads where you sent &gt;5 days ago and the KOL hasn't replied. Nudge drafts (if any) are embedded inline.</p>

<!-- Repeat one .action block per stale thread that has a nudge draft, same structure as the main action queue but with .priority-stale class. Threads without nudge drafts go in the table below. -->
<div class="action" id="stale-1">
  <h3>@{{handle}} — last sent {{lastSentDate}} · {{daysSilent}} days silent <span class="badge badge-stale">stale</span></h3>
  <p class="chip">threadId <code>{{threadId}}</code> · kolId <code>{{kolId}}</code> · <a href="https://biz.bizkol.ai/en/emails?threadId={{threadId}}&amp;campaignId={{campaignId}}&amp;kolId={{kolId}}">Open in Bizkol →</a></p>
  <h4>Suggested nudge</h4>
  <div class="copy-wrap">
    <button class="copy-btn" data-copy-target="#nudge-1">Copy</button>
    <pre id="nudge-1">{{nudgeBody}}</pre>
  </div>
</div>

<h2>Concluded since last run</h2>
<ul class="tight">
  <li>@{{handle}} — moved to <span class="badge badge-status-approved">approved</span> · ${{price}} for {{deliverables}}. <a href="https://biz.bizkol.ai/en/emails?threadId={{threadId}}&amp;campaignId={{campaignId}}&amp;kolId={{kolId}}">Open →</a></li>
  <li>@{{handle}} — moved to <span class="badge badge-status-cancelled">cancelled</span> (declined). <a href="https://biz.bizkol.ai/en/emails?threadId={{threadId}}&amp;campaignId={{campaignId}}&amp;kolId={{kolId}}">Open →</a></li>
</ul>

<h2>Profiles updated this run</h2>
<ul class="tight">
  <li>@{{handle}} — quoted price changed: ${{prev}} → ${{new}} · payment terms set · <code>raw-data/kol/profiles/{{handle}}.md</code></li>
</ul>

<details>
  <summary>Coverage diagnostics (open if anything looks off)</summary>
  <ul class="tight">
    <li>Run mode: <code>{{runMode}}</code></li>
    <li>Pagination: {{paginationStatus}}</li>
    <li>Threads in API: {{apiCount}} · in state before run: {{stateBeforeCount}} · processed this run: {{processedCount}}</li>
    <li>Bucket counts: new-inbound {{n1}} · drafted-pending {{n2}} · inbound-advanced {{n3}} · stale-outbound {{n4}} · overlap-window {{n5}} · concluded {{n6}} · quiet-skipped {{n7}}</li>
    <li>Threads with <code>walked &lt; messageCount</code> (red flag — should be empty): {{redFlagList}}</li>
    <li>Threads with unreadable content: {{unreadableList}}</li>
  </ul>
  <details>
    <summary>Per-thread extraction lines</summary>
    <pre>{{perThreadExtractionLines}}</pre>
  </details>
</details>

<h2>Errors</h2>
<p>{{errorsListOrEmpty}}</p>

<script>
document.querySelectorAll('.copy-btn').forEach(btn => {
  btn.addEventListener('click', async () => {
    const target = document.querySelector(btn.dataset.copyTarget);
    if (!target) return;
    const text = target.innerText;
    try {
      await navigator.clipboard.writeText(text);
      const original = btn.textContent;
      btn.textContent = 'Copied';
      btn.classList.add('copied');
      setTimeout(() => { btn.textContent = original; btn.classList.remove('copied'); }, 1500);
    } catch (e) {
      btn.textContent = 'Copy failed — select & copy manually';
      setTimeout(() => { btn.textContent = 'Copy'; }, 2000);
    }
  });
});
</script>

</body>
</html>
```

### Composition rules

- **Order action items by priority.** Top of the list = the single most leveraged action (e.g., a KOL who fully agreed and just needs an `approved` confirmation; or the highest-fit creator with a fresh inbound). The first action carries the `priority-1` CSS class. The Top priority surfaced in Step 9's print summary should match the `#action-1` block in the report.
- **Embed nudge drafts for stale outbound** as `.action` blocks too if the run produced a nudge draft for that thread. Threads with no nudge draft go in a follow-up table at the bottom of the Stale section. Don't double-list.
- **Status badges:** apply the `badge-status-{value}` class only with values from the canonical enum (`identified`, `shortlisted`, `contacted`, `interested`, `approved`, `conditionally_approved`, `cancelled`, `confirmation`, `appt_booked`, `store_visited`, `content_created`, `product_sent`, `product_received`, `content_draft`, `content_approved`, `published`, `completed`). Any other value is a bug — fall back to plain `<span class="badge">{{value}}</span>` and log to Errors.
- **HTML escaping is mandatory.** All `{{...}}` substitutions involving user-generated content (KOL message body, draft body, KOL name, notes, excerpts) MUST be HTML-escaped before insertion. At minimum: `&` → `&amp;`, `<` → `&lt;`, `>` → `&gt;`, `"` → `&quot;`, `'` → `&#39;`. URL parameters in `href` use `&amp;` rather than `&`. A KOL message containing `<script>` or HTML must not break the report.
- **Empty sections** render `<p class="empty">No items today.</p>` instead of disappearing entirely — keeps the layout stable run-to-run.
- **Drafts file (`content/drafts/outreach-[slug]-[date].md`) is still produced**, unchanged. The HTML report duplicates the drafts inline for review-and-act, but the markdown drafts file remains for clean copy-paste in environments where browser clipboard is restricted.

## Step 8 — Persist updated state (atomic write)

Rewrite `/clients/[client-name]/raw-data/kol/outreach-state-[campaign-slug].json` (pretty-printed, 2-space indent so it's diffable):

**Atomic write protocol** — never write directly to the state file. A run interrupted mid-write would leave corrupted JSON and force tomorrow's run to cold-start.

```bash
write to outreach-state-[slug].json.tmp
fsync
rename outreach-state-[slug].json.tmp → outreach-state-[slug].json
```

If the rename fails or the temp file is missing afterwards, log to "Errors" but do **not** delete the prior state file — better to repeat one day's work than lose all idempotency.

For every thread surfaced in Step 3:

- Update `lastInboundMessageId`, `lastInboundAt`, `lastOutboundAt`, `messageCount` from the latest list response.
- Merge `processedMessageIds` per Step 6.5 rule 7 (union with prior set).
- For threads drafted this run: set `lastDraftedAt = now`, `lastDraftedForMessageId = current lastInboundMessageId`, `lastDraftFile = outreach-[slug]-[YYYY-MM-DD].md`, `draftStatus = "pending_send"`.
- For threads where state had `pending_send` AND `lastOutboundAt > state.lastDraftedAt`: the user sent it — flip `draftStatus` to `"sent"`.
- For threads where the inbound advanced past a prior draft: mark prior `draftStatus = "superseded"` (the new draft entry overwrites with `pending_send`).
- Update `outreachOutcome` based on detected events — `committed` if the KOL fully agreed, `declined` if they declined, `replied_with_conditions` if there are open terms, `replied` if they responded but no commitment yet, otherwise `no_reply`. This is local-only state and is NOT the Bizkol KOL status.

Top-level fields:
- `lastRunAt = now` (ISO 8601 UTC).
- `lastFullPaginationAt = now` (or leave at prior value if `paginationIncomplete: true` was set this run).
- `skillVersion` — current.
- `paginationIncomplete` — `true` if Step 3 retries failed; cleared on next clean run.

For threads that failed to fetch in Step 5 OR whose extraction asserts fired in 6.5f: do **not** update their `processedMessageIds`, `lastInboundMessageId`, or `messageCount` — tomorrow's run must retry. Keep their prior state values intact so the freshness check still flags them as "new activity" next time.

## Step 9 — Print summary and ask to apply changes

After writing files, print the summary block, then **always end the run with an explicit prompt asking the user whether they want to apply this run's findings back into Bizkol**. Do not skip this — it's how negotiated terms make it from drafts into the campaign source of truth.

```
✅ /kol-outreach — [campaign name] — [YYYY-MM-DD]

[N] new inbound, [N] drafts produced, [N] carryovers awaiting your send.
Top priority: [@handle] — [one-line]

Report (open in your browser): clients/[name]/audits/kol-outreach-[slug]-[date].html
Drafts (markdown, for plain copy-paste): clients/[name]/content/drafts/outreach-[slug]-[date].md
Profiles updated: [N] (clients/[name]/raw-data/kol/profiles/)

---

📋 **Please review the report before doing anything else** — it lists the action queue, draft replies, and any KOLs who responded with terms this run.

Want me to apply any of these to Bizkol now? I can:
  1. Add quote information to KOLs (`update_campaign_kol` with `quotes: [...]`) — for the [N] KOLs who quoted a price this run: [@handle ($X), @handle ($X), …]
  2. Append notes to KOLs in Bizkol (`update_campaign_kol` with `notes: "..."`) — to record special conditions, payment terms, usage rights, or decision context surfaced in the threads
  3. Update KOL status (`update_campaign_kol` with `status: "<value>"`) — values must come from the campaign's `workflowTemplate` enum in `assets/campaign-kol-statuses.md`. Per-KOL proposals below; never propose a status outside the canonical enum:
       - [@handle] (currently `<current>`) → propose `<proposed>` because [reason from thread]
  4. Skip — I'll review and apply changes manually in the Bizkol UI

Reply with the numbers you want, or "all", or "skip".
```

**Status proposal rules (enforced before showing option 3):**

1. Read each KOL's current `status` from the Step 2 roster.
2. Look up the campaign's `workflowTemplate`. If `null`, the only safe proposable values are the seven Common Early Statuses from `assets/campaign-kol-statuses.md`: `identified`, `shortlisted`, `contacted`, `interested`, `approved`, `conditionally_approved`, `cancelled`. Do **not** propose template-specific values like `product_sent` or `appt_booked` on a null-template campaign.
3. Map outreach outcomes to statuses per the table in `assets/campaign-kol-statuses.md` (positive reply → `interested`, conditions → `conditionally_approved`, full agreement → `approved`, decline → `cancelled`). Never invent.
4. If the natural mapping doesn't have a unique answer, omit the status proposal for that KOL and only propose `quotes` / `notes` updates. Tell the user why.
5. Validate every proposed status string against the enum file before rendering option 3. If validation fails, treat it as a skill bug — log to Errors and skip the proposal.

When the user picks one or more options, build the corresponding `update_campaign_kol` calls — one per KOL — and execute them. After each batch, confirm what was written back to Bizkol and which KOLs (if any) failed. If a KOL profile file already has the data (price, terms, conditions), use it as the source so the Bizkol record stays in sync with the local profile. **Never include a `status` value in the call that isn't in the canonical enum** — if the API rejects a status string, treat it as a skill bug and surface the rejection plus the valid set.

If the user picks "skip" or doesn't reply: stop here. The drafts file already contains ready-to-paste `update_campaign_kol` payloads they can apply later by hand or on the next run.

## Cadence

This command is idempotent and safe to run on any cadence:

- **Ad-hoc**: run whenever you want a snapshot of what needs your attention.
- **Daily routine**: register on a cron via `/schedule` (e.g., 8am weekdays). State ensures you only see net-new work each morning.
- **Multiple runs in one day**: safe — the second run uses the first run's `lastRunAt` as its cutoff, so it only surfaces what arrived between runs.

## Graceful degradation

- **Bizkol MCP not connected:** stop and tell the user; nothing else is possible.
- **State file corrupted / unreadable:** rename to `outreach-state-[slug]-broken-[timestamp].json` and start cold. Note in the report.
- **A page of `list_email_conversations` fails:** retry once with backoff. If still failing, set top-level `paginationIncomplete: true`, capture what loaded, and surface prominently in the Coverage diagnostics section. Do not silently truncate.
- **`get_email_conversation` fails for a single thread:** skip it, log to "Errors", leave its state untouched so the next run retries (per Step 8).
- **Extraction asserts fire (`walked < messageCount` in 6.5f):** flag in Coverage diagnostics, do not update `processedMessageIds` for that thread so the next run retries the whole thread.
- **State file write fails (atomic rename fails):** log to "Errors", keep the prior state file. The next run will redo today's work — duplicates are prevented by the `processedMessageIds` idempotency in 6.5c.

## Output paths summary

| File | Purpose |
|---|---|
| `/clients/[name]/audits/kol-outreach-[slug]-[YYYY-MM-DD].html` | **Today's review report — daily decision dashboard.** Open in any browser. Action queue at top with embedded draft replies and copy buttons. The user's primary artifact. |
| `/clients/[name]/content/drafts/outreach-[slug]-[YYYY-MM-DD].md` | Today's drafts in plain markdown (new + redrafts + nudges). Backup for clean copy-paste in environments where browser clipboard is restricted. |
| `/clients/[name]/raw-data/kol/outreach-state-[slug].json` | Persistent per-thread state (run-to-run idempotency) |
| `/clients/[name]/raw-data/kol/profiles/[handle].md` | Persistent per-KOL profile — quoted price, payment terms, usage rights, exclusivity, timeline, special conditions; append-only negotiation log with timestamps and threadId source. Survives across campaigns. |

## Related

- `/kol-campaign [campaign]` — create/manage the campaign
- `/kol-performance [campaign]` — performance dashboard for shipped KOLs
- `/schedule` — register `/kol-outreach` on a daily cron
