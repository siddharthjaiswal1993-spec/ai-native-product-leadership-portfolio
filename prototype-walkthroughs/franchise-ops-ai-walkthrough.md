# Franchise Ops AI — Walkthrough Guide

**GitHub:** https://github.com/siddharthjaiswal1993-spec/franchise-operations-ai-gtm-strategy
**Prototype:** https://nexus-foresight-hub.lovable.app

---

## Demo Goal

Show how an AI-native franchise operations platform detects location-level risk, explains root causes, recommends next-best actions for field consultants, and verifies that interventions actually worked.

---

## What Problem This Demonstrates

Franchise software has solved workflow digitization — task lists, compliance checklists, training modules. The next generation needs to solve operational intelligence: which locations are at risk, why, what should happen next, and did the intervention work? This project is a complete strategy package plus a Lovable prototype showing the DETECT→DECIDE→ACT→VERIFY operating loop.

---

## Primary Persona

**Regional Field Consultant at a QSR franchise system.** Responsible for 12 locations across two states. Pre-visit prep takes 3+ hours of manual data gathering. High-risk locations are often discovered during the visit, not before. Has no structured way to track whether previous recommendations were implemented.

---

## Loom Script (~3 minutes)

"Let me walk you through my franchise operations AI project — a complete GTM strategy, product vision, and working prototype.

The problem I'm solving: franchise operations software has digitized workflows but not intelligence. A field consultant visiting 12 locations spends hours gathering data before each visit. High-risk signals are scattered across POS, compliance, training, and support systems with no unified view. After the visit, there's no structured way to know if the recommendations were implemented or whether they worked.

My thesis: the winning franchise platform won't just manage tasks — it will detect early operational risk, explain root causes, recommend next-best actions, and verify outcomes.

The core operating loop I designed is DETECT → DECIDE → ACT → VERIFY. This isn't just a product framework — it's the thing that creates compounding value. Every VERIFY cycle feeds back into the detection model, improving it over time.

The wedge product I identified is the AI Field Consultant — a pre-visit intelligence brief that gives the field consultant a prioritized view of risk signals before they walk in the door. This is the highest ROI entry point: measurable, human-centric, and tied to an existing workflow.

Let me show you the prototype. This is a Lovable-built UI with synthetic franchise data.

On the dashboard, you see the Location Health Score — an explainable, predictive, actionable composite score per location. Not a vanity metric. Every score is decomposed into its contributing signals: food safety compliance, training completion, sales trend, and customer satisfaction indicators.

When a field consultant clicks into a location, they see a pre-visit brief: the three highest-priority issues ranked by risk and expected impact, the last three field consultant visit outcomes, and an AI-generated recommended agenda for the visit.

After the visit, the VERIFY screen shows whether the recommended actions were completed, what the follow-up status is, and whether the location health score moved. This closes the feedback loop.

I published a research paper on this strategy through Gamma — it's linked in the repo."

---

## Click Path (Prototype)

1. Open [nexus-foresight-hub.lovable.app](https://nexus-foresight-hub.lovable.app)
2. Show dashboard — Location Health Score grid across 12 synthetic locations
3. Click into a flagged location — show the risk decomposition panel
4. Navigate to "Pre-Visit Brief" — show the AI-generated consultant agenda
5. Show "VERIFY" or post-visit panel — demonstrate outcome tracking
6. Switch to repo: show `/strategy/` folder for the DETECT→DECIDE→ACT→VERIFY framework doc

---

## Key Screens

**Screen 1 — Dashboard: Location Health Grid**
Highlight: 12 locations in a grid view, each with a composite health score and trend indicator. Red/amber/green scoring with explicit root cause tags (e.g., "Food Safety ↓", "Training Gap"). Not a generic dashboard — operationally actionable at a glance.

**Screen 2 — Location Detail: Risk Decomposition**
Highlight: Each health score decomposes into 5-6 signal categories. Each signal cites the source (POS data, compliance audit, training system). Explainability is a core design principle — field consultants won't trust a black-box score.

**Screen 3 — AI Field Consultant Brief**
Highlight: Pre-visit brief with 3 prioritized issues, recommended talking points, and prior visit context. This is the core wedge product — measurable ROI (prep time reduction), tied to an existing workflow (field visits), and immediately usable without changing franchise operations.

**Screen 4 — VERIFY Panel**
Highlight: Post-visit tracking showing whether recommended actions were completed and whether health score improved. This is the most important screen in the product — the one most likely to be cut in an MVP, and the one that creates compounding value over time.

---

## If Asked "Is This a Real Product?"

"No — this is a strategic portfolio project and prototype built on Lovable with synthetic franchise data. I built it as an independent analysis of where franchise operations software is heading. The strategy is my own original analysis; the prototype demonstrates what the user experience could look like. It's not affiliated with any franchise software company."

---

## Interview Talking Points

**For "Tell me about your GTM thinking":**
"In this project I designed the full GTM strategy for an AI-native franchise operations platform. The core GTM insight was that the wedge product matters more than the full vision at launch. The AI Field Consultant is the wedge: it's tied to an existing workflow, it has measurable ROI in prep-time reduction, and it generates the data exhaust needed to power everything else. You don't lead with the Location Health Score — you earn it by delivering value in the field visit workflow first."

**For "Tell me about a hard product decision":**
"The VERIFY step was the hardest decision. Most franchise software stops at ACT — you track whether a recommendation was made, but not whether it was implemented or whether it worked. I made VERIFY a core loop element, not an optional analytics tab. This creates compounding value because every verified outcome improves the detection model. It also makes it easy to prove ROI to the franchisor. But it requires cooperation from franchisees and field consultants to log outcomes — that's a product adoption challenge I addressed in the GTM section."
