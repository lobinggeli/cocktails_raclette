# Health Coach App — Product Design

## Core Philosophy

One screen, one coach, one daily habit. Not a dashboard. Not another data app.

The user opens it, the coach talks to them, they respond. Everything else is secondary.

---

## App Structure (3 tabs)

```
┌───────────────┬───────────────┬───────────────┐
│     TODAY     │     COACH     │      DATA      │
│   (Home)      │   (Chat)      │   (Trends)    │
└───────────────┴───────────────┴───────────────┘
```

---

## Onboarding Flow (~5 minutes)

```
Screen 1: Welcome
"Hi, I'm your health coach.
I'll combine your wearable data, blood results,
and daily life to guide you — day by day."

[ Get started ]

─────────────────────────────────────────────────

Screen 2: What's your main goal?
○ More energy day-to-day
○ Better sleep and recovery
○ Improve athletic performance
○ Understand my blood results
○ General preventive health

(Can select multiple, ranked by priority)

─────────────────────────────────────────────────

Screen 3: Connect your devices
[ Connect Apple Health ]   ← auto-requests HealthKit permission
[ Connect Oura Ring ]      ← Terra OAuth
[ Connect Garmin ]         ← Terra OAuth
[ Connect Strava ]         ← Terra OAuth (display only)
[ Skip for now ]

─────────────────────────────────────────────────

Screen 4: Quick profile
Age:       [ 34        ]
Sex:       [ Male      ]
Wake time: [ 7:00 AM   ]

Any health conditions? (optional, improves coaching)
[ None / Enter manually ]

─────────────────────────────────────────────────

Screen 5: Your first briefing loads...
(Coach processes last 30 days of HealthKit data in background)

"Based on what I can see so far, here's
where we'll start..."
```

---

## Tab 1: TODAY (Home)

Morning view — generated fresh each day from last 24h of data.

```
┌─────────────────────────────────────────┐
│  Friday, April 11                        │
│                                          │
│  ┌─────────────────────────────────────┐│
│  │  Good morning, Loic.                ││
│  │                                      ││
│  │  You slept 6h42 with HRV of 38ms —  ││
│  │  below your baseline of 52ms.        ││
│  │  Recovery is at 61%.                 ││
│  │                                      ││
│  │  Yesterday's hard session likely     ││
│  │  explains this. Today is a good      ││
│  │  day to keep intensity moderate.     ││
│  │                                      ││
│  │  → Easy run or mobility work         ││
│  │  → Prioritize protein + hydration    ││
│  │  → Aim for 9pm wind-down             ││
│  └─────────────────────────────────────┘│
│                                          │
│  [ Ask a follow-up ]   [ Log something ] │
│                                          │
│  ─────────── Today's snapshot ────────  │
│  Sleep    HRV      Steps    Readiness    │
│  6h42     38ms     0        61%          │
│  ↓ low    ↓ low    —        ↓            │
│                                          │
│  ─────────── Up next ─────────────────  │
│  No workout logged for today             │
│  [ Log planned workout ]                 │
└─────────────────────────────────────────┘
```

**Rules for the daily brief:**
- Always 3 actionable bullet points (never just observations)
- References yesterday's data and logged events
- Ties back to the user's stated goal
- Takes 2-3 seconds to generate

---

## Tab 2: COACH (Chat)

The user can talk to the coach anytime. Event log lives here.

```
┌─────────────────────────────────────────┐
│  Coach                                   │
│                                          │
│  ┌───────────────────────────────────┐  │
│  │ Your HRV has been trending down    │  │
│  │ for 4 days. Are you under more     │  │
│  │ stress than usual, or was there    │  │
│  │ anything that could explain it?   │  │
│  └───────────────────────────────────┘  │
│                          10:32 AM       │
│                                          │
│          ┌─────────────────────────┐    │
│          │ Had a couple drinks Tue │    │
│          │ and Wed, and work has   │    │
│          │ been intense            │    │
│          └─────────────────────────┘    │
│                          10:33 AM       │
│                                          │
│  ┌───────────────────────────────────┐  │
│  │ That explains the pattern well.   │  │
│  │ Alcohol suppresses deep sleep,    │  │
│  │ which lowers HRV recovery.        │  │
│  │                                   │  │
│  │ With the work stress on top, your │  │
│  │ nervous system is running high.   │  │
│  │                                   │  │
│  │ Two things that will help fast:   │  │
│  │ → No alcohol tonight (even 1      │  │
│  │   drink delays HRV rebound 48h)  │  │
│  │ → 10 min nasal breathing before  │  │
│  │   sleep                           │  │
│  │                                   │  │
│  │ Check back Saturday morning —     │  │
│  │ you should see HRV start climbing │  │
│  └───────────────────────────────────┘  │
│                          10:33 AM       │
│                                          │
│  ┌──────────────────────────────────┐   │
│  │ Quick log                        │   │
│  │  🍺 Alcohol    😴 Bad sleep       │   │
│  │  💪 Hard workout  🤒 Feeling off  │   │
│  │  ✈️ Travel     🍕 Ate poorly      │   │
│  └──────────────────────────────────┘   │
│                                          │
│  [ Type a message... ]            [ ↑ ] │
└─────────────────────────────────────────┘
```

**Quick log chips are the key UX differentiator.** One tap to give the coach context. The AI references these in future messages. This is what makes coaching feel human.

---

## Tab 3: DATA (Trends)

Simple and non-overwhelming. The coach references these; users explore when curious.

```
┌─────────────────────────────────────────┐
│  Your data                               │
│                                          │
│  [ 7D ]  [ 30D ]  [ 90D ]               │
│                                          │
│  HRV (ms)                               │
│  ████████████████████░░░░  38           │
│  Baseline: 52ms  ↓ -27% this week       │
│                                          │
│  Sleep (hrs)                            │
│  6.1  7.2  6.8  7.4  7.0  6.8  6.7     │
│  Mon  Tue  Wed  Thu  Fri  Sat  Sun      │
│                                          │
│  Resting HR                             │
│  52 bpm  +3 vs baseline                 │
│                                          │
│  ─────────── Blood results ───────────  │
│  Last test: March 18, 2026              │
│  Testosterone  18.2 nmol/L  ✓ Normal    │
│  Cortisol      28.4 µg/dL   ↑ High      │
│  Ferritin      67 µg/L      ✓ Normal    │
│                                          │
│  [ Upload new results ]                  │
│                                          │
│  ─────────── Connected sources ───────  │
│  ✓ Apple Health    ✓ Oura Ring          │
│  ✓ Garmin          ○ Strava             │
│  ○ Lab results     [ + Add source ]     │
└─────────────────────────────────────────┘
```

---

## Lab Upload Flow

```
[ Upload new results ]
        ↓
"Upload your lab report PDF
 or take a photo of the page"

[ Choose PDF ]  [ Take photo ]
        ↓
Extracting results... (5-10 seconds)
        ↓
"I found 12 results. Please confirm:"

Testosterone    18.2  nmol/L    ✓
Cortisol        28.4  µg/dL     ✓
Ferritin        67    µg/L      ✓
Vitamin D       ——    (unclear) [ Enter manually ]
        ↓
[ Confirm & analyze ]
        ↓
Coach message appears in chat:
"Your cortisol is elevated at 28.4 µg/dL
(reference: 6.2–19.4). Combined with the
HRV trend we've been seeing, this suggests
your stress load has been high.

Here's what this means for your goals..."
```

---

## What the First 7 Days Feel Like

**Day 1:** Onboards, connects Apple Health + Oura. Gets first morning brief. Surprised it already knows their sleep patterns from the last 30 days.

**Day 2:** Opens at 8am. Coach notes great sleep (HRV up). Suggests a harder training day. User logs "planned: 10km run."

**Day 3:** HRV down after the run. Coach explains recovery timeline. User asks "how long until I'm ready for another hard session?" — Coach says Saturday, explains why.

**Day 4:** User taps "had drinks last night" from quick log. Coach acknowledges it in the morning brief, adjusts recommendations.

**Day 5:** User uploads their Ares blood test PDF. Coach cross-references cortisol with HRV trend. Gives synthesized picture no single app has ever shown them.

**Day 7:** Coach sends weekly summary: "Here's what we learned about your body this week." User feels like they finally understand what's going on.

---

## MVP Scope

### V1 (Month 1-2)
- iOS app, Swift
- Onboarding + goal setting
- Apple HealthKit sync (sleep, HRV, steps, resting HR)
- Oura via Terra
- Morning brief (Claude API)
- Coach chat tab
- Quick event log (alcohol, hard workout, bad sleep, travel, etc.)

### V2 (Month 3)
- Garmin via Terra
- Lab PDF upload + Claude Vision extraction
- Trend charts (Data tab)
- Weekly summary message

### V3 (Month 4-5)
- Strava (display only — not fed to AI due to their terms)
- Proactive mid-day check-in messages
- Profile refinement from ongoing interactions
- Android / web consideration
