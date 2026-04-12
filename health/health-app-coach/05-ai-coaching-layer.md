# Health Coach App — AI Coaching Layer

## The Core Failure Mode to Avoid

Every competitor (Superpower, InsideTracker, etc.) treats coaching as **insight generation** — "your HRV was low last night." That's just reading data back to the user.

**The right model is context-aware reasoning** across all signals simultaneously.

---

## Context Architecture

Per coaching request, the AI receives ~2,500 tokens total:

```
User context:
├── Last 7 days: HRV trend, sleep score, resting HR (daily summaries, not raw)
├── Last 24h: workout intensity, steps, calories
├── Last event log: "had drinks", "hard session", "traveling", "stress"
├── Blood biomarkers (last test): cortisol, testosterone, ferritin, etc.
├── User goals: "improve VO2max" / "more energy" / "better sleep"
└── Last 3 coaching interactions (continuity)
```

**What NOT to include:**
- Entire health history (context bloat kills accuracy)
- Raw minute-level sensor data
- Old coaching interactions (>1 week stale)
- Duplicate signals (steps + step count)

---

## Key Design Principles

### 1. The Event Log is the Most Important Feature
If the AI knows you had a party last night, it gives completely different advice. Without that context, it's guessing. **Make event logging frictionless: one tap + optional voice note.**

Event types:
- Alcohol (even one drink delays HRV rebound 48h)
- Bad sleep (travel, baby, noise)
- Hard workout (context for next-day recovery)
- Illness or feeling off
- High stress (work, personal)
- Ate poorly (processed food, late meals)
- Travel (timezone, sleep disruption)

### 2. Rolling Memory (4-Week Trends)
Don't just look at today. Track trends.
- "You've had 3 poor recovery nights this week" > "your HRV was 42ms"
- "Your HRV baseline has improved 18% over the last month" shows progress
- "This is the 4th time this month alcohol correlated with a 2-day HRV dip" is personalized

### 3. Goal-Anchored Advice
Every coaching message ties back to the user's declared goal. Generic apps forget this.
- User goal: "improve performance" → frame recovery advice in terms of training adaptation
- User goal: "more energy" → frame the same advice in terms of daily energy levels

### 4. Always 3 Actionable Bullets
The morning brief ends with exactly 3 things to do today. Not observations. Not warnings. Actions.
- ✓ "Easy run or mobility work today"
- ✓ "Prioritize protein + hydration"  
- ✓ "Aim for 9pm wind-down"
- ✗ "Your HRV suggests you should be careful"

### 5. Predictive Follow-Up
The coach makes predictions it can verify:
- "Check back Saturday morning — you should see HRV start climbing"
- "If you avoid alcohol tonight, your deep sleep should improve tomorrow"
This creates a feedback loop that builds trust.

---

## Morning Brief Prompt Structure

```
System:
You are a personal health coach with access to the user's wearable data,
blood biomarkers, and daily logs. Your role is to give specific, actionable
daily guidance — not generic wellness advice.

Rules:
- Always reference specific data points (not generic)
- End with exactly 3 actionable items for today
- Tie advice to the user's stated goal: {goal}
- If a biomarker is abnormal, suggest discussing with a doctor — never diagnose
- Use "may," "suggests," "consider" — never "you have," "you must"
- Be direct and concise. No filler. No disclaimers in the main message.

User profile:
Age: {age}, Goal: {goal}

Last 7 days:
{daily_metrics_table}

Latest blood results ({test_date}):
{biomarker_summary}

Events logged recently:
{event_log}

Last coaching interaction ({n} days ago):
{last_coaching_snippet}

Generate this morning's health brief.
```

---

## Guardrails (Medical vs. Coaching Line)

| Risky (avoid) | Safe (use) |
|---|---|
| "Your cortisol indicates adrenal fatigue" | "Your cortisol is elevated — consider discussing with your doctor" |
| "Take ashwagandha to treat your stress" | "Adaptogens like ashwagandha may support stress response — research suggests..." |
| "Your glucose spikes indicate prediabetes" | "Your glucose patterns show sensitivity to certain foods" |
| "You should see a cardiologist" | "Consider discussing this pattern with your healthcare provider" |
| "This is causing your fatigue" | "This may be contributing to your energy levels" |

**Hard rules:**
- Never recommend medications (prescription or OTC)
- Never diagnose conditions
- Never replace professional medical advice
- Always suggest doctor consultation for out-of-range biomarkers
- Add disclaimer in app footer (not in every message — that kills UX)

---

## Model Choice

**Use Claude (Anthropic API)** as the primary reasoning engine.
- 200K token context window — can afford richer context
- Excellent at synthesizing multi-variable inputs
- Strong instruction following for guardrails
- Prompt caching keeps costs low for repeated user profile context

**Cost estimate per user per month:**
- 1 morning brief/day: ~2,500 tokens in + ~500 tokens out = ~3,000 tokens
- ~10 coach chat exchanges/week: ~1,500 tokens average each
- Total: ~30,000 × 4 = ~120,000 tokens/month/user
- At Claude Sonnet pricing (~$3/M input, $15/M output): ~€0.40-0.60/user/month

At €69/month pricing, AI cost is <1% of revenue per user.

---

## Weekly Summary Format

Sent every Sunday morning:

```
This week's summary:

Sleep: Average 7.1h (↑ from 6.6h last week) ✓
HRV: Average 44ms (↓ from 52ms baseline) — stress or training load high
Workouts: 4 sessions, 1 rest day

What we learned:
→ Alcohol on Tuesday correlated with your two lowest HRV days
→ Your best sleep came after the Thursday evening walk
→ Recovery was fastest after your mobility sessions vs. hard runs

Next week focus:
→ Protect sleep on weekdays (10:30pm target)
→ Try 10 min evening walk — your data suggests it helps
→ Schedule a rest day mid-week given accumulated training load
```
