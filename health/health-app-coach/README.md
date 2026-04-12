# Health Coach App

A daily AI health coach that aggregates wearables, activity, and blood biomarker data into one personalized coaching experience.

## Files

| File | Contents |
|---|---|
| [01-concept-and-market.md](01-concept-and-market.md) | Idea overview, API landscape, competitive analysis, market gaps |
| [02-technical-architecture.md](02-technical-architecture.md) | Full tech stack, database schema, data flow, PDF parsing approach |
| [03-go-to-market.md](03-go-to-market.md) | Target segments, pricing, acquisition channels, B2B angle, 3-phase roadmap |
| [04-product-design.md](04-product-design.md) | App screens, onboarding flow, user experience, MVP scope |
| [05-ai-coaching-layer.md](05-ai-coaching-layer.md) | AI context architecture, prompt design, guardrails, cost estimates |
| [06-regulatory-and-compliance.md](06-regulatory-and-compliance.md) | GDPR, nDSG, medical device risk, DPA requirements |

## One-Line Summary per File

**Concept & Market:** No app today combines wearables + biomarkers + activity into daily coaching. EU/Swiss market is wide open.

**Tech:** Swift iOS + FastAPI + TimescaleDB + Terra SDK (wearable aggregation) + Claude API + PDF parsing for labs. MVP in ~2 months.

**GTM:** Start with biohackers (high willingness to pay, existing wearables). €69/month. B2C first, B2B corporate wellness in month 9+.

**Product:** One coach, one daily brief, one chat. Morning brief + quick event log (one tap: alcohol, hard workout, travel, etc.) is the core UX.

**AI:** Context = 7-day metrics + biomarkers + event log + user goals. ~2,500 tokens/request. ~€0.50/user/month in AI costs.

**Regulatory:** Health data = sensitive personal data under nDSG/GDPR. Swiss servers required. Stay on "coaching" side of the medical advice line.

## Data Sources

| Source | Integration | Status |
|---|---|---|
| Apple HealthKit | Native iOS (on-device) | Required — no backend API |
| Oura Ring | Terra SDK (webhook) | Available with Oura membership |
| Garmin | Terra SDK (polling) | Requires business partner approval |
| Strava | Terra SDK (display only) | **AI prohibition** — cannot feed to coaching model |
| Swiss labs (Ares, Synlab) | PDF upload + Claude Vision | No public API; use PDF parsing for MVP |

## MVP Build Order

```
Month 1-2:  Swift iOS + HealthKit + Oura (Terra) + Morning brief + Coach chat
Month 3:    Garmin (Terra) + Lab PDF upload + Trend charts + Weekly summary
Month 4-5:  Strava (display) + Proactive check-ins + Profile refinement
Month 9+:   B2B corporate wellness, lab partnerships, Android/web
```
