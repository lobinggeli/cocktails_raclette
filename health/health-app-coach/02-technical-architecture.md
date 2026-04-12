# Health Coach App — Technical Architecture

## Recommended MVP Stack

```
Swift (iOS)  →  FastAPI (Python)  →  TimescaleDB
     +               +                    +
HealthKit       Terra SDK           Redis cache
(on-device)  (Garmin/Oura/Strava)
     +              +
Lifesum data   Claude API (coaching)
(via HealthKit      +
 or direct API) PDF parser (lab reports)
```

---

## Nutrition Data — Lifesum Integration

Vital does not build its own food logging UI. Instead, it imports nutrition data from **Lifesum**, where users already track their meals. Same model as Garmin or Oura — connect once, data flows automatically.

### Integration Approach

**Phase 1 — HealthKit bridge (MVP, $0, ships in days)**

Lifesum already syncs to Apple Health. If the user has enabled this in Lifesum settings, Vital reads their nutrition data from HealthKit automatically — no Lifesum API or partnership needed.

```
User logs meal in Lifesum
        ↓
Lifesum syncs to Apple Health (user setting)
        ↓
Vital reads from HealthKit on-device
        ↓
Nutrition data enters coaching context
```

Limitation: HealthKit stores daily totals only (calories, protein, carbs, fat) — no individual meal breakdown or timing. Sufficient for coaching context in MVP.

**Phase 2 — Direct Lifesum partnership**

Negotiate B2B API access with Lifesum. This unlocks:
- Individual meal entries with timestamps (breakfast / lunch / dinner)
- Food item names (not just totals)
- Meal timing (critical: late dinner → poor sleep correlation)
- Two-way data if needed

Lifesum has no public API — requires direct business partnership. They are actively partnering (Consupedia deal April 2025, FIIT wellness deal). Approach them with user base as leverage.

---

## Meal Tracking API Landscape

| Source | Access | Cost | Data quality |
|---|---|---|---|
| **HealthKit (Lifesum bridge)** | Free, on-device | $0 | Daily totals only — good for MVP |
| **Lifesum direct API** | B2B partnership | Negotiated | Full meal log with timing — ideal |
| **MyFitnessPal** | Partner only, closed | N/A | Not accepting new partners — avoid |

---

## Nutrition Data Schema (from Lifesum via HealthKit or API)

```sql
-- Daily nutrition totals (from HealthKit bridge)
nutrition_daily(
  user_id,
  date,
  calories,
  protein_g,
  carbs_g,
  fat_g,
  fiber_g,
  source           -- 'healthkit' | 'lifesum_api'
)

-- Individual meals (from Lifesum direct API only)
meals(
  user_id,
  logged_at,       -- timestamp — critical for late-dinner correlations
  meal_type,       -- breakfast | lunch | dinner | snack
  calories,
  protein_g, carbs_g, fat_g,
  food_names       -- array of food item names logged
)
```

## How Lifesum Data Feeds the Coaching AI

Add to daily coaching context:

```
Yesterday's nutrition (via Lifesum):
  Total: 2,100 kcal | Protein: 142g | Carbs: 198g | Fat: 68g
  Dinner logged at 9:47pm — high carb
```

**Coaching examples unlocked:**

- "You had a high-carb dinner at 9:47pm → likely explains the lighter deep sleep (1h08 vs your 1h44 avg)"
- "You're 3 days into a high-protein week and your HRV is trending up — the recovery correlation is showing"
- "You've logged 1,400 kcal today and have a hard run tomorrow — consider a larger dinner tonight"
- "Your cortisol is elevated AND you've been under-eating (avg 1,600 kcal last 5 days). Chronic under-fuelling elevates cortisol — these two are likely connected."

---

## Stack Decisions

### Mobile — Swift native (not React Native)
React Native works but lags 3-12 months on new HealthKit APIs. For an app where HealthKit is core, go native from day one. SwiftUI provides direct, zero-latency integration.

**If cross-platform is needed later:** React Native via `react-native-healthkit` is viable, but expect feature lag.

### Wearable Integration — Terra SDK
[Terra](https://tryterra.co) is a unified API layer that wraps Garmin, Oura, Strava, and 50+ others into one normalized webhook. Instead of building each integration separately, Terra handles:
- Different OAuth flows per provider
- Polling vs. webhook differences
- Data format normalization

**Saves ~2-3 months of integration work.** Worth the cost for an MVP.

**Native webhook support per platform:**
- **Oura:** Full webhook support (sleep, readiness, activity, HRV, VO2max, etc.)
- **Strava:** Webhook support but AI prohibition — display only, don't feed to coaching model
- **Garmin:** No native webhooks — Terra adds a webhook layer on top of polling

### Database — TimescaleDB
TimescaleDB (PostgreSQL extension) chosen over InfluxDB because:
- Full SQL — can join health metrics with biomarkers, coaching history, user profiles
- 3-71x faster than InfluxDB for complex cross-metric queries
- One database for time-series + relational data (no dual-DB complexity)
- PostGIS support if location data from Strava is needed later

**Core schema:**
```sql
-- Time-series metrics (hypertable, partitioned by time + user_id)
metrics(user_id, timestamp, hrv, sleep_score, resting_hr, steps, calories, ...)

-- Aggregated daily summaries (for fast trend queries)
daily_summaries(user_id, date, avg_hrv, sleep_hours, total_steps, readiness_score, ...)

-- Blood biomarkers
biomarkers(user_id, test_date, lab_name, test_name, value, unit,
           reference_min, reference_max, status, raw_pdf_url, extraction_confidence)

-- Coaching interactions
coaching_log(user_id, timestamp, context_snapshot, coach_message, user_reply)

-- Event log
events(user_id, timestamp, type, note)
-- types: alcohol, bad_sleep, hard_workout, travel, illness, stress, ate_poorly
```

**7-day metrics query for AI context:**
```sql
SELECT
  DATE_TRUNC('day', timestamp) as date,
  AVG(hrv) as avg_hrv,
  AVG(sleep_score) as avg_sleep,
  SUM(steps) as daily_steps,
  AVG(resting_hr) as avg_rhr
FROM metrics
WHERE user_id = $1 AND timestamp > NOW() - INTERVAL '7 days'
GROUP BY date
ORDER BY date;
```

### Backend — FastAPI (Python)
- Async-first, handles concurrent polling for thousands of users
- Native integration with Claude SDK, LangChain, LlamaIndex
- Built-in OpenAPI docs; Pydantic for health data validation

### Message Queue — Celery + Redis
Used for:
- Polling Garmin every 4-6 hours (when not using Terra)
- Processing uploaded lab PDFs
- Generating coaching messages asynchronously

---

## Lab Report Integration (PDF Parsing)

No public APIs exist for Swiss labs (Ares, Synlab, Unilabs). The approach:

### Phase 1 — Claude Vision (MVP)
1. User uploads PDF or takes photo of lab report
2. Claude Vision extracts: test name, value, unit, reference range, date
3. ~90% accuracy on clean PDFs, ~$0.10/report
4. Low-confidence extractions flagged for user confirmation
5. Structured data stored in `biomarkers` table

**This works on day 1. Zero lab partnership required.**

### Phase 2 — Dedicated PDF parser
Services like Spike Lab Reports API provide normalized LOINC-coded output. More reliable at scale. ~$0.50-2.00/report.

### Phase 3 — Direct lab partnership
Approach Ares or Synlab with a value prop: you bring them users who need regular testing, they give you structured data export. B2B deal, not a public API.

---

## AI Coaching Context Window

Target: **2,000-3,500 tokens per coaching request** (not the full context window).

| Data | Tokens | Notes |
|---|---|---|
| Last 7 days metrics (summarized) | 500-1,000 | Averages by day, not raw datapoints |
| Latest biomarkers + trends | 200-300 | Last 3 lab results |
| User goals + profile | 100-200 | Stable, rarely changes |
| Last 3 coaching interactions | 300-500 | Continuity, what worked |
| Recent event log | 100-200 | Alcohol, hard workout, travel, etc. |
| System prompt + disclaimers | 300-500 | Fixed overhead |
| User's current message | 50-200 | Always include |

**Architecture:**
```
User Request
    ↓
[Retrieval Layer]
  - TimescaleDB: fetch 7-day metric summaries
  - Biomarker cache: latest labs + trend direction
  - Coaching log: last 3 interactions
  - Event log: last 7 days of logged events
  - User profile: goals, age, conditions
    ↓
[Context Assembly]
  - Filter by relevance score
  - Stay within token budget
    ↓
[Claude API Call]
  - System prompt (role, tone, disclaimers)
  - Assembled context
  - User request
    ↓
[Stream response back to app]
[Log interaction for future context]
```

---

## Full Data Flow

```
┌─────────────────────────────────────────┐
│          iOS App (Swift)                │
│  - HealthKit (on-device sync)           │
│  - Manual event logging                 │
│  - Lab PDF upload                       │
│  - Coach chat interface                 │
└──────────────┬──────────────────────────┘
               │
               ▼
    ┌──────────────────────┐
    │   FastAPI Backend    │
    └──────────┬───────────┘
               │
    ┌──────────┴──────────┬──────────┬──────────┐
    │                     │          │          │
    ▼                     ▼          ▼          ▼
┌────────┐          ┌─────────┐ ┌────────┐ ┌────────┐
│ Terra  │          │ Polling │ │Webhook │ │Claude  │
│ SDK    │          │(Garmin) │ │(Oura/  │ │Vision  │
│(unified│          │         │ │Strava) │ │PDF OCR │
│webhook)│          └────┬────┘ └───┬────┘ └────┬───┘
└────┬───┘               │          │           │
     └───────────────────┴──────────┴───────────┘
                          │
                 ┌────────▼──────────┐
                 │  Celery Queue     │
                 │  (Redis broker)   │
                 └────────┬──────────┘
                          │
             ┌────────────┴────────────┐
             │                         │
             ▼                         ▼
     ┌──────────────┐         ┌──────────────┐
     │ TimescaleDB  │         │ Redis Cache  │
     │ (metrics +   │         │ (sessions +  │
     │ biomarkers)  │         │ rate limit)  │
     └──────┬───────┘         └──────────────┘
            │
            ▼
     ┌──────────────────────┐
     │  Context Assembly    │
     │  + Claude API        │
     └──────────┬───────────┘
                │
                ▼
     ┌──────────────────────┐
     │  Streaming Response  │
     │  → iOS App           │
     └──────────────────────┘
```

---

## Infrastructure

| Component | Choice | Why |
|---|---|---|
| Cloud | AWS Europe (Frankfurt/Zurich) or Azure Switzerland | GDPR/nDSG data residency |
| Containers | Docker + ECS or Kubernetes | Horizontal scaling for polling jobs |
| Monitoring | OpenTelemetry + Prometheus | Health data queries need observability |
| Logging | Structured JSON (Datadog or Loki) | Audit trails required for compliance |

---

## Key Risks

| Risk | Mitigation |
|---|---|
| Garmin no native webhooks | Use Terra SDK (adds webhook layer) |
| PDF parsing accuracy <85% | Show confidence scores, ask user to confirm |
| HealthKit data stale on sync | Cache with timestamps, show "data age" |
| Multi-wearable data inconsistency | Normalize to percentiles vs. raw values |
| HIPAA/nDSG compliance | Encrypt at rest + transit; audit logging; BAA with Claude/LLM provider |
