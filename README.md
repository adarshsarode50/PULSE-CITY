# PulseCity AI

> **Understand Your City. Improve Your Community.**

PulseCity AI is an AI-powered civic intelligence platform. Citizens report local
problems, the AI analyses each report the moment it is submitted, and the
platform turns those reports into city-level patterns, insights and actionable
work queues for community organisations and municipal authorities.

PulseCity AI is a proposed open-source AI-powered civic intelligence platform that will be implemented during the final hackathon. An existing prototype was used to validate the product concept and user experience.

---

## 1.Proposed Solution.

PulseCity AI is a proposed AI-powered civic intelligence platform

### Core workflow
- **Report an Issue** — validated form (title, category, description, location,
  severity, optional photo) with a **live AI preview** as you type.
- **AI analysis on submit** — category, severity, priority, summary, possible
  impact, recommended action, keywords and a confidence score.
- **Automatic duplicate detection** — the new report is compared against every
  existing report by text similarity + geographic distance, and similar reports
  are linked and grouped into a signal.
- **City Pulse** — health score, community activity, issue trends, category and
  severity distributions, neighbourhood activity table, resolution performance.
- **AI Insights** — a "Generate New AI Insights" action analyses the live report
  set and produces ranked, actionable city insights.
- **Smart Map** — Leaflet + OpenStreetMap, severity/status-coloured markers,
  critical markers pulse, popups with the AI summary, severity/category/area
  filters and an optional neighbourhood activity overlay.
- **Community Signals** — clusters of reports describing the same underlying
  problem, with linked report IDs, cluster radius and a consolidated action.
- **My Reports** — personal tracker with status filtering, search, table + card
  views.
- **Report Details** — full record, AI panel, lifecycle timeline, nearby related
  reports, operator status controls.
- **City Operations** — operator KPIs, prioritised queue, status/category
  filters, inline status updates, CSV export.
- **Global search** — across title, description, location, category and report ID.
- **Notifications** — received / under review / similar-nearby / resolved, with
  an unread badge and mark-as-read.



### Demo data
14 realistic reports across real Nagpur neighbourhoods (Dharampeth, Sitabuldi,
Manish Nagar, Sadar, Civil Lines, Wardha Road). Every demo row is stored with
`is_demo = 1` and surfaced in the UI as **SAMPLE DATA**, so it is never confused
with a citizen submission.   

## 2.System Architecture

```text

┌──────────────────────┐
│      Citizens        │
│  Report Civic Issue  │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│   PulseCity AI App   │
│  Report / Map / UI   │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│   Backend / API      │
│ Validation & Storage │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Open-Source AI Model │
│                      │
│ • Classification     │
│ • Severity           │
│ • Priority           │
│ • Summarization      │
│ • Related Issues     │
│ • Recommendations    │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│       Database       │
│ Reports + Analysis   │
└──────────┬───────────┘
           │
           ▼
┌────────────────────────────────┐
│       City Intelligence        │
│                                │
│ Dashboard │ Map │ Insights     │
│ Operations │ Reports            │
└────────────────────────────────┘



