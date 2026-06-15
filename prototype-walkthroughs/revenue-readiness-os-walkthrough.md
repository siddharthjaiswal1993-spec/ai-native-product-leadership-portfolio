# Revenue Readiness OS — Walkthrough Guide

**GitHub:** https://github.com/siddharthjaiswal1993-spec/revenue-readiness-os

---

## Demo Goal

Show how an AI-native sales readiness platform gives reps a personalized skill score, lets them practice with an AI roleplay buyer, gives managers a prioritized coaching queue, and connects readiness to pipeline outcomes.

---

## What Problem This Demonstrates

Sales enablement has a measurement problem: completion rates are tracked, revenue impact is not. A rep who completed the onboarding certification may still lose deals because of poor objection handling. Revenue Readiness OS is a concept for a continuous readiness intelligence system where AI measures actual skills, not just training completion.

---

## Primary Persona

**Frontline Sales Manager at a B2B SaaS company.** Manages 8 AEs. Spends 6 hours per week in 1:1s, but coaching is intuition-based — whoever asks gets coached. High-performers get praise; struggling reps get overlooked until they miss quota. Has no structured data on why specific deals are lost.

---

## Loom Script (~3 minutes)

"Let me show you Revenue Readiness OS — an AI-native sales readiness platform prototype.

The enablement problem I'm solving: most sales training measures completion, not capability. A rep who finished the module does not necessarily know how to handle a CFO objection about implementation risk. The skill gap is invisible until a deal is lost.

Revenue Readiness OS introduces continuous skill measurement through AI-graded roleplay simulations and behavioral signal analysis. The readiness score per rep is a composite across 10 skill dimensions — discovery quality, objection handling, value articulation, technical fluency, and more.

The rep experience: they open the platform and see their personal readiness score, which skills are strong, and which have gaps. They can launch an AI roleplay simulation — the system plays a buyer persona relevant to their current deals — and get scored on their performance after each session.

The manager experience: instead of intuition-based coaching, they see a prioritized coaching queue. The 2 reps who need coaching today, ranked by the expected revenue impact of the coaching intervention. Each recommendation includes the specific skill gap, the deal at risk, and a suggested coaching conversation starter.

I built a 10-page React prototype: command center, rep readiness, AI roleplay, coaching queue, enablement programs, content intelligence, skill gap analytics, deal readiness, analytics and ROI, and an AI assistant."

---

## Click Path (Prototype)

1. Navigate to `revenue-readiness-os/prototype` and run `npm install && npm run dev`
2. Show Command Center — team readiness overview and AI executive summary
3. Show Rep Readiness — individual skill score breakdown
4. Show AI Roleplay — scenario card and simulated buyer conversation interface
5. Show Coaching Queue — prioritized manager action list with reasoning
6. Show Analytics & ROI — ramp time correlation to readiness score

---

## Key Screens

**Screen 1 — Command Center**
Highlight: Team readiness heatmap showing all 8 reps' composite scores by skill dimension. At-risk reps highlighted in amber/red. AI-generated executive summary of team readiness status. This replaces the manager's gut feel with structured data.

**Screen 2 — AI Roleplay**
Highlight: Scenario card shows buyer persona, deal stage, and key objections expected. Rep enters the simulated conversation. After completion, scores appear by skill dimension with specific feedback. Practice without risking a real deal.

**Screen 3 — Coaching Queue**
Highlight: 2-3 coaching actions prioritized for today. Each shows: rep name, specific skill gap, the deal at risk, expected revenue impact, and suggested coaching approach. Not "coach everyone equally" — coaching where the ROI is highest.

**Screen 4 — Analytics & ROI**
Highlight: Readiness score correlation to quota attainment. Ramp time trend for the past 4 cohorts. Content adoption vs. skill improvement comparison. The metric that matters is pipeline impact, not completion rates.

---

## If Asked "Is This a Real Product?"

"Revenue Readiness OS is a React prototype with synthetic rep and deal data. The 10-page UI demonstrates the full platform experience, but it runs on mock data with no real LLM integration in the prototype. Not affiliated with any enablement platform."

---

## Interview Talking Points

**For "Tell me about a metrics-first product design":**
"In Revenue Readiness OS, I started the design with the outcome metrics before designing a single feature. The north star is readiness-to-quota correlation — if higher readiness scores don't correlate with higher quota attainment, the product isn't working. I then worked backwards: what input metrics predict readiness? Roleplay scores, call analysis signals, assessment results. This forced me to eliminate content completion tracking as a readiness proxy — it measures activity, not capability."
