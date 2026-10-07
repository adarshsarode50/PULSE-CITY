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

## 3.Open-Source AI

PulseCity AI will use an open-weight language model as the
core intelligence layer for analyzing citizen reports.

The model will help:
- classify civic issues
- estimate severity and priority
- summarize reports
- identify related issues
- generate recommended actions
- generate city-level insights

The model is a core component of the system rather than an
optional chatbot. 
---  
## 4.Data Flow

1. A citizen submits a civic issue.
2. The backend validates the report.
3. The report is sent to the open-source AI model.
4. The AI analyzes the issue and generates structured results.
5. The results are stored with the original report.
6. Related reports are identified and grouped.
7. Aggregated data is used to generate city-level insights.
8. Citizens and operators view the information through the dashboard,
   map, and reports interface.
   
 ## 5. Security

- All third-party API calls happen **server-side**; no keys reach the client.
- `Validator` enforces required fields, lengths, enums and numeric ranges;
  `sanitizeText()` strips tags and script payloads from free text.
- `assertOperator()` gates every `/api/operations/*` route server-side.
- Uploads are validated by MIME type and size before touching storage.
- `secureHeaders()` sets a Content-Security-Policy; responses carry
  `X-Content-Type-Options` and `Referrer-Policy`.
- A fixed-window rate limiter protects report submission, AI, upload and search.
- Errors return a stable JSON envelope with a request id; internals are logged,
  never returned.

---

## 6.Problem

Problems such as:

* Citizens don't have an effective way to report local problems.
* Municipal teams can receive many scattered complaints.
* Important patterns can be difficult to identify.
* Similar complaints may be duplicated.
* City decision-makers need structured insights.
---

## 7.How It Will Work

1. Citizen opens PulseCity AI.
2. Citizen reports a local issue.
3. The report is sent to the AI analysis layer.
4. The open-source AI model analyzes the report.
5. The system categorizes and prioritizes the issue.
6. Related reports are identified.
7. The information appears on the city map and dashboard.
8. City operators can review and act on prioritized issues.


---
## 8.Implementation Plan

### Phase 1 — Core Application
Build the report submission and city dashboard.

### Phase 2 — Open-Source AI
Integrate the selected open-source/open-weight model.

### Phase 3 — Civic Intelligence
Implement classification, severity, priority, related issues,
and recommended actions.

### Phase 4 — City Analytics
Build the map, trends, community signals, and AI insights.

### Phase 5 — Testing
Evaluate AI outputs using representative civic reports.

### Phase 6 — Final Demo
Deploy and demonstrate the complete working system.

---
## 9. Scalability

PulseCity AI is designed to support expansion from a single city
to multiple cities by separating civic reports, locations,
analytics, and city-specific configurations.

The architecture can later support larger datasets, additional
AI models, and integrations with municipal systems.

