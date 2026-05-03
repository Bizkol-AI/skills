---
name: seo-geo
description: >
  Use this skill for SEO and Generative Engine Optimization (GEO) audits and
  research — domain health, keyword rankings, content gaps, technical
  spot-checks, backlink analysis, SERP / AI Overview review, llms.txt and
  schema readiness, citability for ChatGPT / Perplexity / Gemini / Google AIO.
  Trigger on phrases like "SEO audit", "search rankings", "GEO audit",
  "AI search visibility", "content gap", "keyword research", "Search Console
  data", "SERP review", "AI Overviews", or the `/seo-audit` command.
metadata:
  version: "0.1.0"
---

# SEO + GEO Audit Skill

Search optimization for both **traditional SEO** (Google, Bing) and **Generative Engine Optimization** (ChatGPT, Claude, Perplexity, Gemini, Google AI Overviews). Optimize for where traffic is actually going, not where it was.

> **Read first:** `marketing-framework/SKILL.md` for tool-priority rules and the data-first principle. **Detail:** `references/seo-audit-methodology.md` for the full step-by-step audit process.

## When to Use This Skill

- The user asks for an SEO audit, GEO audit, or search visibility review
- Generating the SEO sections of weekly / monthly reports
- Investigating ranking drops, traffic anomalies, or AI search citation gaps
- Pre-launch search readiness for new content or sites
- Competitor SEO comparison (combine with **`competitor-research`** skill)

## Prerequisite — Client Folder

Before any SEO/GEO command runs, the client folder must exist at `/clients/[client-name]/` with `client-brief.md`. If missing, stop and instruct the user to run `/new-client [name]` first.

## Tool Priority

| Need | Primary | Fallback if not connected |
|------|---------|---------------------------|
| Client's own GSC data (clicks, impressions, CTR, position) | Windsor MCP `get_data` (`searchconsole`) | Chrome → `search.google.com/search-console` — capture same fields manually |
| Client's own GA4 organic data | Windsor MCP `get_data` (`googleanalytics4`, filter `default_channel_group=Organic Search`) | Chrome → `analytics.google.com` |
| Domain authority, keyword ranks, backlinks, content gaps (client + competitors) | Semrush MCP | Chrome → Google SERP for spot-checks |
| Competitor organic traffic estimates | Semrush MCP | — |
| Live SERP inspection + AI Overview presence | Chrome | — |
| Schema / llms.txt / robots / meta validation | WebFetch + manual checks | — |

**CRITICAL:** Windsor is for the client's *own* GSC / GA4 data only. All competitor SEO data comes through Semrush; Windsor must never be used for competitor research. **If Windsor is not connected**, fall back to Chrome on the Search Console / GA4 UI for the same data — never block the audit.

## Audit Output Structure

Every SEO+GEO audit produces these sections (in order — data first, analysis after):

1. **Executive summary** — top 3 wins, top 3 risks, top 3 actions.
2. **Domain health** — Authority Score, organic traffic estimate, top pages, top keywords (Semrush).
3. **Keyword performance** — ranking distribution, MoM movement, branded vs non-branded (Semrush + Windsor GSC).
4. **Search Console deep-dive** — top queries, top pages, CTR vs position curve, query intent gaps (Windsor GSC).
5. **Content gaps** — keywords competitors rank for that the client doesn't (Semrush).
6. **Technical spot-check** — index coverage, core web vitals, crawlability, mobile, sitemaps, robots.
7. **Backlink profile** — referring domains trend, lost links, toxic-link signals, link velocity (Semrush).
8. **GEO readiness** — AI Overview / SGE presence, citability score, llms.txt status, schema coverage, mention presence on AI-cited platforms.
9. **Prioritized action plan** — issue × impact × effort, sequenced 0–30 / 30–90 / 90+ days.

See **`references/seo-audit-methodology.md`** for the complete step-by-step process, data templates, and per-section rubrics.

## Folder Output

Save audits under the client folder:

```
/clients/[client-name]/
  /raw-data/seo/
    seo-audit-domain-[YYYY-MM-DD].md
    seo-audit-keywords-[YYYY-MM-DD].md
    seo-audit-backlinks-[YYYY-MM-DD].md
    seo-audit-content-gaps-[YYYY-MM-DD].md
    seo-audit-top-pages-[YYYY-MM-DD].md
    seo-audit-technical-[YYYY-MM-DD].md
    seo-audit-serp-[YYYY-MM-DD].md
    seo-audit-gsc-[YYYY-MM-DD].md      ← Windsor GSC (or Chrome fallback)
    seo-audit-ga4-[YYYY-MM-DD].md      ← Windsor GA4 (or Chrome fallback)
  /audits/
    seo-audit-[YYYY-MM-DD].md          ← finished report
```

**Raw data first, analysis after** — no exceptions. If a tool fails, note the gap in the file and continue.

## GEO-Specific Checks

Beyond classic SEO, run these for AI search readiness:

- **llms.txt presence + accuracy** — root-level file matching the [llms.txt spec](https://llmstxt.org).
- **Schema coverage** — JSON-LD for `Organization`, `Person`, `Article`, `FAQPage`, plus `sameAs` linking and `speakable` where relevant.
- **AI crawler access** — `GPTBot`, `ClaudeBot`, `PerplexityBot`, `Google-Extended` (and others) not blocked by robots.txt or firewall.
- **Citability** — passages structured for AI extraction: clear definitions, specific facts with dates, attributable stats, short authoritative paragraphs.
- **Brand mention presence** — coverage on Wikipedia, Reddit, Quora, industry publications, podcasts, YouTube — the platforms AI models lean on for entity recognition.

## Audience Modes

- **Agency**: audit per client; folder per client; report saved as `seo-audit-[YYYY-MM-DD].md` under their reports.
- **Merchant**: audit your own domain; folder is your brand root; same report cadence.

## Related Skills / Commands

- **`marketing-framework`** — operating principles
- **`competitor-research`** — for competitor organic comparison
- `/seo-audit` — the standalone audit command
