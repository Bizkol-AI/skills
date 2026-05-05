# Campaign KOL Statuses — canonical reference

This is the source-of-truth for valid `CampaignKOL.status` values. Skills MUST consult this file before proposing any `update_campaign_kol` payload. Do not invent statuses; do not rely on values copied from informal docs or chat history.

**Source:** `bizkol-merchant/lib/constants/campaign-workflows.ts` (re-verify whenever the merchant code changes — this file is a cached mirror).

## How statuses work

- `CampaignKOL.status` is stored in DynamoDB as a free `string` (no DB-level enum). Validation lives in application code at `lib/constants/campaign-workflows.ts` and the dropdown components that consume it.
- The valid set of statuses depends on the campaign's `workflowTemplate`. There are exactly two templates today:
  - `local_store` — local store / offline business campaigns
  - `ecommerce_product` — online product / e-commerce campaigns
- A campaign's `workflowTemplate` may be `null` (early-stage / draft campaigns where the user hasn't picked a flow yet). In that case only the *common early statuses* below are safe to propose. Do not propose template-specific statuses (`product_sent`, `appt_booked`, etc.) on a campaign with a null `workflowTemplate`.

## Common early statuses (valid for either template, including `workflowTemplate = null`)

These are the seven statuses where the workflow template is still mutable, per `EARLY_WORKFLOW_STATUSES` in the merchant code.

| Value | Label | Meaning |
|---|---|---|
| `identified` | Identified | Sourced into the roster; no contact yet. **Initial status.** |
| `shortlisted` | Shortlisted | Client / merchant flagged as interesting |
| `contacted` | Contacted | Initial outreach sent |
| `interested` | Interested | KOL replied with interest |
| `approved` | Approved | KOL approved by merchant for the campaign |
| `conditionally_approved` | Conditionally Approved | Approved with conditions (e.g., pricing adjustment needed) |
| `cancelled` | Cancelled | Campaign cancelled or KOL declined. **Final status.** |

## `local_store` — full enum (in order)

`identified` → `shortlisted` → `contacted` → `interested` → `approved` → `conditionally_approved` → `confirmation` → `appt_booked` → `store_visited` → `content_created` → `published` → `completed` (final) — plus `cancelled` (final, available at any point).

## `ecommerce_product` — full enum (in order)

`identified` → `shortlisted` → `contacted` → `interested` → `approved` → `conditionally_approved` → `product_sent` → `product_received` → `content_draft` → `content_approved` → `published` → `completed` (final) — plus `cancelled` (final, available at any point).

## Confirmed cooperation statuses (across both templates)

Per `CONFIRMED_COOPERATION_STATUSES` in the merchant code — these mark "KOL has confirmed participation":

`confirmation`, `published`, `completed`, `appt_booked`, `store_visited`, `content_created`, `product_sent`, `product_received`, `content_draft`, `content_approved`.

Use this set when computing the funnel "converted" count in `/kol-performance` — these are the post-negotiation statuses.

## Mapping outreach intent → status

When a `/kol-outreach` run detects an outcome in a thread, surface the **closest valid status** for the campaign's `workflowTemplate`. Never invent. If unsure, leave `status` out of the proposed `update_campaign_kol` payload and only update `notes` / `quotes`.

| Outcome detected in thread | Proposed status (early stage / null template) | Notes |
|---|---|---|
| KOL replied positively, no terms locked | `interested` | Safest default after first reply |
| KOL replied with conditions / pricing tweaks | `conditionally_approved` | Surface the conditions in `notes` |
| KOL fully agreed to the proposed terms | `approved` | Merchant's own approval also needed in some flows; double-check with user |
| KOL declined / unsubscribed / negative reply | `cancelled` | Final state |
| KOL went silent past the nudge cutoff | (leave status unchanged) | "no response" is not a status — track in local outreach state, not Bizkol |

For status changes past `approved` (template-specific: `confirmation`, `appt_booked`, `product_sent`, etc.), do not auto-propose from email outreach — those transitions usually involve operational steps (sending a product, booking a visit) that the user owns. Surface them only when the user explicitly asks.

## Forbidden / non-existent statuses (DO NOT use)

These appear in older skill drafts or LLM-generated content but **do not exist** in Bizkol. Never write them to `update_campaign_kol`:

`agreed`, `declined`, `paused`, `posted`, `live`, `concluded`, `negotiating`, `no_response`, `confirmed`, `pending`, `rejected`, `accepted`, `failed`, `done`.

## Campaign-level status — do NOT use

The Campaign entity has a `status` field with values `['draft', 'active', 'paused', 'completed', 'archived', 'deleted']` per the Amplify schema, but it is **not used reliably in the merchant code today**. Skills must NOT:

- Filter campaigns by status (e.g., don't query for only `active` campaigns)
- Treat status as a signal of campaign health / progress
- Display campaign status as a meaningful field in reports
- Make decisions based on campaign status

When listing campaigns for the user to pick from, list everything returned regardless of status, and do not surface the status field. If the underlying MCP tool requires a status filter and `"all"` isn't supported, fetch all known values and merge — but never expose the filter to the user.
