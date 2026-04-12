# Health Coach App — Regulatory & Compliance

## Swiss nDSG vs. GDPR

Switzerland's Federal Act on Data Protection (nDSG, in force Sept 2023) aligns closely with GDPR but has one critical difference: **individual (not just company) liability up to CHF 250,000** for intentional violations.

Both laws classify blood biomarker data and wearable biometric data as **sensitive personal data** — the highest protection category.

---

## Compliance Checklist

| Requirement | Action needed | Priority |
|---|---|---|
| Lawful basis | Explicit opt-in consent per data type (wearables, biomarkers, etc.) | CRITICAL |
| Data residency | EU/CH servers only (AWS Frankfurt, Azure Switzerland) — no US storage without legal review | CRITICAL |
| Breach notification | Notify users + Swiss FDPIC within 72 hours if breach affects 20+ people | CRITICAL |
| Encryption | End-to-end encryption for biomarker data at rest and in transit | HIGH |
| Data Processing Agreements | Sign DPAs with Apple, Garmin (Terra), Oura, Claude/Anthropic | HIGH |
| User rights | Export, correction, deletion within 30 days on request | HIGH |
| Retention policy | Define max retention (e.g., delete after 24 months unless user opts for longer) | MEDIUM |
| Privacy policy | Must be available in German, French, Italian (Swiss official languages) | MEDIUM |
| Children | No targeting under-16 without parental consent | MEDIUM |

**Budget estimate:** Engage a Swiss data protection lawyer before launch. ~CHF 15,000-30,000 for audit + implementation.

---

## Health Coaching vs. Medical Advice (The Gray Zone)

**Health coaching (unregulated in most EU countries):**
- General lifestyle advice: nutrition, exercise, stress management, sleep hygiene
- Educational content: "how to interpret your biomarkers"
- Motivational support: habit formation, behavior change

**Medical advice (regulated, requires license):**
- Diagnosing conditions
- Prescribing treatments or medications
- Making clinical recommendations for specific diseases

**Switzerland specifically:**
- "Wellness coaching" is largely unregulated
- If the AI makes clinical recommendations, Swissmedic may classify the app as a medical device
- Avoid: diagnostic claims, treatment recommendations, medication suggestions

**Germany (largest TAM):**
- Health coaches are not regulated — but cannot use medical terminology or make clinical claims
- Cannot advertise as replacing physician care

---

## App Regulatory Strategy

1. **Explicit disclaimer in app:** "This app provides health insights and coaching, not medical diagnosis or treatment. Always consult your healthcare provider for medical decisions."

2. **AI guardrails (hard-coded, not just prompted):**
   - Never recommend medications
   - Never diagnose conditions
   - Always suggest "consider discussing with your doctor" for out-of-range biomarkers
   - Language: "may," "consider," "suggests" — not "you have," "you must," "indicates"

3. **Professional review at scale:** Consider having a licensed physician or nutritionist review AI coaching logic before scaling (adds credibility, reduces liability)

4. **Positioning:** "Preventive health coaching" — not "medical advice." Some Swiss insurers cover this under wellness budgets.

---

## Data Residency Architecture

```
User in Switzerland
      ↓
iOS App (data on-device via HealthKit)
      ↓
FastAPI Backend
  └── Hosted on: AWS eu-central-1 (Frankfurt) or Azure Switzerland North
      └── TimescaleDB: same region
      └── Redis: same region
      └── Claude API calls: data processed by Anthropic
          (sign Business Associate Agreement / DPA with Anthropic)
```

**Never store:**
- Health data in US data centers without explicit legal review
- PHI (personally identifiable health information) in unencrypted logs
- Lab results in plain text (always encrypted at rest)

---

## Third-Party DPAs Required

| Provider | Data shared | DPA needed |
|---|---|---|
| Apple (HealthKit) | User health metrics | Apple's own consent handles this; your terms must align |
| Terra SDK | Garmin/Oura/Strava OAuth tokens + synced metrics | Yes — Terra DPA |
| Anthropic (Claude) | Health context per coaching request | Yes — Anthropic DPA/BAA |
| AWS / Azure | All stored health data | Yes — standard DPA included in their enterprise terms |
| PDF parsing service | Lab report contents | Yes — if using third-party (Spike, DocuPipe, etc.) |

---

## Swissmedic Medical Device Risk

**When it applies:** If the app claims to diagnose, monitor, or treat a medical condition.

**How to stay clear:**
- No diagnostic claims ("this suggests you may have X condition")
- No therapeutic claims ("use this to treat X")
- Frame everything as coaching and education
- Add FDA/Swissmedic disclaimer to app store listing

**If the app grows into clinical territory** (e.g., partnering with physicians, making clinical recommendations), consult Swissmedic proactively before that feature ships.
