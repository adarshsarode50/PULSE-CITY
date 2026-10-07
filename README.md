# PulseCity AI

> **Understand Your City. Improve Your Community.**

PulseCity AI is an AI-powered civic intelligence platform. Citizens report local
problems, the AI analyses each report the moment it is submitted, and the
platform turns those reports into city-level patterns, insights and actionable
work queues for community organisations and municipal authorities.

This is a **working MVP** prototype: report submission, persistent storage,
AI analysis, city analytics, insight generation, an interactive map, search,
status tracking and an operator console are all functional end-to-end.

---

---

## 1. What is built

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

### Data model (7 entities)
`users`, `reports`, `report_images`, `ai_analysis`, `insights`, `locations`,
`notifications` — plus `timeline` for lifecycle events and `app_meta` for the
report-ID sequence and seed marker.

### Demo data
14 realistic reports across real Nagpur neighbourhoods (Dharampeth, Sitabuldi,
Manish Nagar, Sadar, Civil Lines, Wardha Road). Every demo row is stored with
`is_demo = 1` and surfaced in the UI as **SAMPLE DATA**, so it is never confused
with a citizen submission.

---

## 2. Functional entry URIs

### Pages
| Path | Description |
| --- | --- |
| `/` | Dashboard — hero, live statistics, recent reports, top insights |
| `/report` | Report an Issue form — includes a **real interactive map picker** (tap/drag pin + "Use my location") |
| `/city-pulse` | City Pulse intelligence dashboard |
| `/insights` | AI Insights |
| `/map` | Smart Map (`?area=Dharampeth` focuses a neighbourhood, `?near=me` opens on the visitor's own area) |
| `/signals` | Community Signals |
| `/my-reports` | My Reports tracker |
| `/reports/:id` | Report details (e.g. `/reports/PC1001`) |
| `/operations` | City Operations console |

### API
| Method | Path | Description |
| --- | --- | --- |
| `POST` | `/api/reports` | Submit a report → validation, storage, AI analysis, duplicate detection, notifications |
| `GET` | `/api/reports` | List reports. Filters: `status`, `category`, `severity`, `area`, `search`, `mine=true`, `sort`, `limit`, `offset` |
| `GET` | `/api/reports/:id` | Detail + timeline + images + related reports |
| `PATCH` | `/api/reports/:id` | Update `status` / `severity` / `priority` / `category` |
| `POST` | `/api/reports/:id/action` | Generate the AI recommended action |
| `POST` | `/api/ai/analyze` | Re-analyse a stored report (`report_id`) or preview ad-hoc text |
| `GET` | `/api/ai/insights` | Read the last generated insights |
| `POST` | `/api/ai/insights` | Generate new insights from the live report set |
| `POST` | `/api/ai/related` | Find related reports for a report |
| `GET` | `/api/ai/status` | Active AI provider chain (no secrets) |
| `GET` | `/api/analytics/stats` | Headline metrics + all chart series |
| `GET` | `/api/analytics/areas` | Neighbourhood activity |
| `GET` | `/api/analytics/signals` | Community signals |
| `GET` | `/api/analytics/health` | City health score with its inputs |
| `GET` | `/api/search?q=` | Global search with facets |
| `GET` | `/api/operations/summary` | Operator KPIs |
| `GET` | `/api/operations/queue` | Prioritised queue |
| `PATCH` | `/api/operations/:id` | Operator status update |
| `GET` | `/api/map/reports` | Provider-neutral GeoJSON marker payload |
| `GET` | `/api/map/locate?lat=&lng=` | Reverse geocode a coordinate → neighbourhood, ward, address label |
| `GET` | `/api/map/nearby?lat=&lng=&radius=` | "My area": reports within a radius of the visitor, with counts + distances |
| `POST` | `/api/files/upload` | Image upload (validated, 5 MB max) |
| `GET` | `/api/files/:key` | Serve a stored image |
| `GET` | `/api/notifications` | Notification feed |
| `POST` | `/api/notifications/read` | Mark one or all as read |
| `GET` | `/api/platform/status` | Active adapters + AWS readiness |
| `GET` | `/api/health` | Health probe |

---

## 3. AI architecture

`AIService` is the single abstraction the app talks to:

```ts
analyzeReport(report, allReports)      -> AIAnalysis
generateCityInsights(reports, stats)   -> Insight[]
findRelatedReports(report, allReports) -> RelatedReport[]
generateRecommendedAction(report)      -> string
```

Providers are tried in order; the first success wins:

1. **`BedrockProvider`** — Amazon Bedrock. Uses Bedrock's OpenAI-compatible
   endpoint when `BEDROCK_API_KEY` is set, otherwise SigV4-signed `InvokeModel`
   with IAM credentials (Web Crypto, no AWS SDK in the bundle).
2. **`OpenAIProvider`** — any OpenAI-compatible chat-completions endpoint.
3. **`LocalProvider`** — the deterministic **PulseCity civic analyser**. Always
   available, so the product never breaks when a model is unreachable.

Select with `AI_PROVIDER = auto | bedrock | openai | local`.

### Why the analysis is genuinely dynamic
The local engine is not a lookup table. It derives every field from the actual
report text:

- **Category** — weighted phrase scoring across nine civic category profiles.
- **Severity** — safety markers (`accident`, `injured`, `school`, `children`,
  `live wire`, `open manhole`, …), escalation language (`weeks`, `still`,
  `repeatedly`), category baselines, and a boost from the number of similar
  nearby reports. An explicit user severity acts as a floor, never a ceiling.
- **Priority** — derived from severity plus cluster size.
- **Summary / impact / action** — composed from the report text and a
  category-specific civic playbook.
- **Keywords** — term frequency weighted by category relevance.
- **Similarity** — Jaccard token overlap + category agreement + haversine
  distance, which powers duplicate detection and clustering.

**Verified example.** Submitting *"Large pothole near school entrance — two
students on a scooter fell last week"* produced `Road & Traffic`, severity
raised **High → Critical**, priority **Urgent**, a summary referencing school
children and two-wheeler riders, and an action that consolidated the 4 similar
reports already logged in Dharampeth.

---  
 ## 4. Security

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

## 5. Tech stack

- **Hono** on **Cloudflare Workers / Pages** — server-rendered JSX, no client framework
- **Cloudflare D1** (SQLite) — schema and demo seed applied idempotently on boot
- **Vanilla ES2020** client modules — no build step, no bundler for the front end
- **Leaflet + OpenStreetMap** for maps, **Chart.js** for visualisations
- Custom dark civic-tech design system (`public/static/styles.css`)

---

## 6. Demo script (all verified working)

1. Open `/` — dashboard with live statistics from the seeded demo dataset.
2. Open `/city-pulse` — health score, trends, distributions, neighbourhood table.
3. Click **Report an Issue**.
4. Enter: title *"Large pothole near school entrance"*, description *"Large
   pothole creating a safety problem for students and commuters"*, category
   *Road & Traffic*, severity *High*, location *Dharampeth*.
5. Submit — the AI analysis modal runs, then shows category, severity, priority,
   summary, impact and recommended action.
6. Return to `/city-pulse` — totals, critical count and health score have changed.
7. Open `/insights` → **Generate New AI Insights** — the new report contributes
   to a Dharampeth road-damage cluster signal.
8. Open `/map` — the new marker is on the map (pulsing if critical).
9. Open `/my-reports` — the report appears with status **Submitted**.
10. Open `/operations` — change the status; the reporter is notified.

### Real-map flow (also verified)

1. On `/report`, scroll to **Pinpoint it on the map**.
2. Tap anywhere on the map — a pin drops, and the coordinates are written into
   the (read-only) Latitude/Longitude fields.
3. The address line resolves to the nearest real neighbourhood, the **Location**
   field is prefilled, and a *"N reports within 1.2 km"* readout appears with the
   three closest existing reports.
4. Press **Use my location** — the browser asks for permission, then the map
   centres on the visitor's *actual* position, shows a GPS accuracy circle, and
   fills in their real area.
5. Submit — the report is stored at the pinned coordinates and appears at that
   exact spot on `/map`.
6. On `/map`, press **My area** (or open `/map?near=me`) — a "you are here" dot
   plus a 1.5 km radius ring is drawn, and the *Reports near you* panel lists the
   reports around the visitor with distances, severity and open/critical counts.

---

## 7. Real map (geolocation + location picker)

The map is a genuine interactive map — Leaflet 1.9.4 with CARTO/OpenStreetMap
tiles — not a static image.

| Capability | Where | How it works |
| --- | --- | --- |
| Open the visitor's area | `/map` → **My area**, or `/map?near=me` | `navigator.geolocation.getCurrentPosition()` → `GET /api/map/nearby` |
| Pick a precise issue location | `/report` → map picker | Click/tap or drag the pin → `GET /api/map/locate` resolves the neighbourhood |
| Show the visitor's position | Both maps | Blue "you are here" dot + accuracy/radius circle |
| Nearby report counts | Both maps | Server-side haversine distance, sorted nearest-first |

**Provider-neutral by design.** All reverse geocoding and distance maths happens
server-side behind `MapService.locate()` (`src/services/map.ts`). The default
implementation resolves coordinates against the known neighbourhood grid, so the
picker works with **no third-party geocoding API, no key and no network round
trip**. Swapping in Amazon Location Service, Mapbox or Nominatim is a change to
that one method — the HTTP contract (`/api/map/locate`, `/api/map/nearby`) and
every page stay exactly the same.

**Privacy.** Coordinates are only used to compute a neighbourhood label and
distances, are sent only to this application's own `/api/map/*` routes, and are
never forwarded to a third party. Location permission is requested on the
visitor's explicit click, never on page load. The map still works fine if
permission is denied — the citizen can simply tap the map instead.

---

## 8. Not yet implemented

- Real citizen authentication (the demo uses a seeded citizen + operator session)
- Live municipal system integration (SCADA, complaint gateways)
- Push/email/SMS notification delivery (in-app notifications only)
- Automatic geocoding of free-text addresses (coordinates come from the
  neighbourhood grid or a map pin)
- Photo moderation and EXIF stripping
- Multi-city data partitioning (the city selector is presentational)
- OpenSearch/DynamoDB backfills and migrations for existing data

---

## 9. Recommended next steps

1. Connect **Amazon Bedrock** (`BEDROCK_MODEL_ID` + credentials) to move analysis
   from the local engine to a hosted model — no frontend changes required.
2. Provision the **DynamoDB** single table and the three GSIs, then set
   `DYNAMODB_TABLE` to activate the adapter.
3. Create the **S3** bucket and set `S3_BUCKET` so photos are stored as objects.
4. Add **Cognito** (or any OIDC provider) and replace `resolveUserId()` with real
   session claims; then set `ALLOW_ANON_OPERATOR=false`.
5. Deploy the API routes to **Lambda behind API Gateway** for the AWS demo.
6. Add a **DynamoDB stream → OpenSearch** indexer for full-text search at scale.

---

