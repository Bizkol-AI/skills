---
name: kol-outreach
description: Manage email outreach for a Bizkol campaign — list active threads, surface awaiting-reply conversations, and draft follow-ups. Thread state lives in Bizkol; only the generated drafts and review summary are saved locally.
---

# /kol-outreach $ARGUMENTS

Manage and review KOL email outreach for a Bizkol campaign. This command focuses on **status review and draft follow-ups** — the actual sending happens through Bizkol's UI.

## Step 0: Prerequisite — Client Folder

Before running, the client folder must exist at `/clients/[client-name]/` with `client-brief.md`. If not:

> "No client folder found for [client-name]. Run `/new-client [client-name]` first to create the brief and folder structure, then re-run this command."

Stop here if the folder is missing.

## Workflow

### Step 1 — Identify the campaign

Look up the campaign in the client's `client-brief.md` Bizkol Campaign Index. If `$ARGUMENTS` doesn't match an indexed campaign name, list both `active` and `draft` campaigns from Bizkol — outreach often begins on a draft campaign before it's flipped to active. The `status` parameter accepts a single value, so make two calls:

```
Tool: list_campaigns
Parameters: { status: "active" }

Tool: list_campaigns
Parameters: { status: "draft" }
```

Merge both result sets and present them to the user, marking each entry with its status (draft / active) so they can pick the right one. Resolve the choice to a `campaignId`.

### Step 2 — Pull all email threads for the campaign

```
Tool: list_email_conversations
Parameters:
  campaignId: "<id>"
  status: "all"
  limit: 100
```

This is a live read against Bizkol — **do not write a local raw file** unless the user explicitly asks for an archival snapshot (which would land at `/clients/[name]/raw-data/kol/outreach-threads-[campaign-slug]-[YYYY-MM-DD].md`).

### Step 3 — Bucket threads by status

Group into:
- **Awaiting reply (us)** — KOL replied, we owe a response. **Highest priority.**
- **Active negotiation** — back-and-forth in progress
- **Awaiting reply (them)** — we sent, they haven't responded; flag for follow-up if >5 days
- **Replied** — concluded with an answer
- **Closed** — declined or completed

### Step 4 — Drill into priority threads

For threads in **Awaiting reply (us)** or **Active negotiation**, pull the full thread:

```
Tool: get_email_conversation
Parameters:
  threadId: "<id>"
  campaignId: "<id>"
  kolId: "<id>"
```

Read the latest KOL message and assess:
- Are they accepting / declining / negotiating?
- What needs to be answered or sent?
- Are there contract / pricing details to capture in `update_campaign_kol`?

### Step 5 — Draft follow-ups

For each priority thread, draft a reply in `/clients/[client-name]/content/drafts/outreach-[campaign-slug]-[YYYY-MM-DD].md`:

```markdown
## Thread: [@handle] — [topic]
**Status:** Awaiting reply from us
**Last message:** [date] — [one-line summary of their message]
**Bizkol threadId:** [id]

### Suggested reply
Hi [name],

[draft body — match the tone of the conversation]

[CTA / next step]

Best,
[Sender]

### Notes for the user
- Confirm price: $X for Y deliverables
- Update Bizkol: `update_campaign_kol` with quotes: [...]
```

> **Important:** This command only **drafts** replies — it never sends email. The user reviews drafts in Bizkol's UI and sends from there. If a KOL has agreed to a price/terms, surface the recommended `update_campaign_kol` payload so the user can persist it in one action.

### Step 6 — Stale-thread report

For **Awaiting reply (them)** older than 5 days, suggest a polite nudge. Format the same way as Step 5 drafts.

## Output

Save the outreach review summary to `/clients/[client-name]/audits/kol-outreach-[campaign-slug]-[YYYY-MM-DD].md`:

```markdown
# Outreach Review — [campaign]
**Date:** [YYYY-MM-DD] | **Bizkol campaignId:** [id]

## Summary (live from Bizkol)
- Awaiting our reply: X threads
- Active negotiation: X threads
- Awaiting their reply (>5 days): X threads
- Concluded this week: X threads

## Top priorities
1. [@handle] — [one-line: what they said + what to send back]
2. ...

## Drafts
See `/clients/[name]/content/drafts/outreach-[campaign-slug]-[YYYY-MM-DD].md`

## Suggested Bizkol updates
[list of `update_campaign_kol` payloads to apply]
```

Live thread state always lives in Bizkol — re-run this command for an updated picture.

## Graceful degradation

- **If Bizkol MCP isn't connected:** This command can't run — no thread data is available. Inform the user.
- **If a thread fails to load:** Note the threadId and skip to the next.
