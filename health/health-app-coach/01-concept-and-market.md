# Health Coach App — Concept & Market Research

## The Idea

An app that aggregates data from wearables (Apple Watch, Garmin, Oura Ring), activity trackers (Strava), nutrition apps (Lifesum), and blood biomarker tests (Swiss labs like Ares), then delivers personalized daily AI coaching based on all of it combined.

**The gap:** No app today properly combines wearables + blood biomarkers + activity into one daily coaching experience. The European/Swiss market is entirely underserved.

---

## API Landscape

| Source | API exists? | Key restriction |
|---|---|---|
| Apple HealthKit | Yes, on-device only | Requires native iOS app — no backend access |
| Garmin | Yes, free | Requires business partner approval |
| Oura Ring | Yes | Gated behind Oura Membership (Gen3/Ring 4) |
| Strava | Yes, free | **Explicit AI prohibition since Nov 2024** — cannot use data in AI models |
| Lifesum | No public API | Requires direct partnership negotiation |
| Swiss labs (Ares, Synlab) | None documented | Requires B2B partnerships; use PDF parsing as interim |

**Critical constraints:**
- Apple Health forces a native iOS app (no backend-only approach)
- Strava data can be displayed but not fed into the AI coaching model

---

## Competitive Landscape

| App | Strengths | Weaknesses |
|---|---|---|
| **WHOOP** | OpenAI-powered coach, strain/sleep/lab integration | Requires proprietary wearable |
| **Levels** | CGM + biomarkers + nutrition, $288/yr | CGM-only focus, US-centric |
| **Superpower** | 100+ lab tests + wearables, $16/mo, $30M raised | Users report AI insights are generic |
| **InsideTracker** | 48 biomarkers + 1,500 DNA markers + wearable sync | No daily coaching feel |
| **Function Health** | 160+ biomarker tests | No AI coaching |
| **athletedata.health** | Strava/WHOOP/Oura aggregation | Fitness metrics only, no biomarkers |

---

## Market Gaps (The Opportunity)

1. **No true multi-source integration** — wearables + biomarkers + activity + nutrition in one place
2. **AI coaching is shallow everywhere** — even well-funded players get criticized for generic insights
3. **European/Swiss market completely underserved** — no local lab integrations, no regional adaptation
4. **Actionable daily guidance vs. dashboards** — most apps give data, not "here's what to do today"
5. **Data ownership transparency** — users increasingly suspicious of data practices

---

## Market Size

- Wearable medical devices: $45B (2024) → $151.8B (2029), CAGR 27.5%
- European biohacking market: €7.46B (2025), growing 17.39% CAGR through 2034
- Europe corporate wellness: €19.14B (2024) → €38.02B (2034), CAGR 7.1%
