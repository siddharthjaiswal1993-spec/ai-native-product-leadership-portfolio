# FlowCRM AI — Walkthrough Guide

**GitHub:** https://github.com/siddharthjaiswal1993-spec/flowcrm-ai

---

## Demo Goal

Show how an AI-native CRM adoption layer surfaces a sales rep's prioritized daily worklist, drafts CRM field updates from call notes, and auto-suggests next actions — reducing the manual CRM data entry that causes CRM adoption failure.

---

## What Problem This Demonstrates

CRM adoption failure is the most consistent problem in enterprise sales tooling. The root cause is simple: sales reps perceive CRM as a reporting tool for managers, not a tool that helps them sell. Every minute spent on CRM data entry is a minute not selling. FlowCRM is an AI-native adoption layer that sits on top of an existing CRM (Salesforce, HubSpot) and makes the daily CRM workflow so fast that rep adoption becomes the rational choice.

---

## Primary Persona

**Mid-market Account Executive at a B2B SaaS company.** Manages 40 active accounts, 12 active opportunities. Current CRM usage: updates Salesforce after every call, but only the minimum fields required. Spends 45 minutes per day on CRM data entry. Primary complaint: "Salesforce is for my manager, not for me."

---

## Loom Script (~3 minutes)

"Let me walk you through FlowCRM AI — a TanStack Start prototype for a CRM adoption layer.

The CRM problem I'm solving: CRM tools have a 50-70% adoption rate in most sales organizations because reps don't see the personal value. The update flow is manual, time-consuming, and designed for reporting rather than helping the rep do their job.

FlowCRM is an adoption layer — not a replacement CRM. It sits on top of Salesforce or HubSpot and does three things:

First, it surfaces a prioritized daily worklist. Not 40 accounts — the 5 that need attention today, ranked by deal risk, next action due date, and stage progression signals. The rep starts every day knowing exactly what to do first.

Second, when a call ends, FlowCRM drafts the CRM field updates — next steps, discovery notes, objection tracking, deal stage recommendation — from the call notes or transcript. The rep reviews, edits, and approves. What previously took 10 minutes takes 90 seconds.

Third, it suggests next-best actions based on deal stage and buyer engagement signals. Not generic 'send a follow-up' — specific recommendations like 'Buyer mentioned budget freeze in Q3. Schedule a call to revisit post-August 1st.'

I built a TanStack Start prototype with 6 routes, a complete data model, and realistic synthetic scenarios for Acme Logistics as the demo company."

---

## Click Path (Prototype)

1. Run locally: `cd flowcrm-ai && npm install && npm run dev` → http://localhost:3000
2. Show daily worklist — 5 prioritized accounts with reason codes
3. Click into an opportunity — show account detail + deal stage
4. Show CRM field update draft — AI-drafted from mock call notes
5. Show next-best action recommendation — specific, contextual suggestion
6. Navigate to analytics — show rep productivity metrics

---

## Key Screens

**Screen 1 — Daily Worklist (Home)**
Highlight: 5 accounts flagged for today out of 40. Each has a reason ("Next action overdue 3 days", "Champion went dark", "Renewal in 45 days"). This is the core value proposition: AI triage, rep action.

**Screen 2 — Opportunity Detail**
Highlight: Account health, deal stage, last activity, contact engagement, and open risks in one view. No tab-switching between account, contact, and opportunity objects in Salesforce.

**Screen 3 — AI Field Update Draft**
Highlight: After a call, the rep pastes or uploads notes. AI generates: next steps, updated close date assessment, discovery notes, and deal stage recommendation. The rep edits and confirms. One click to sync to Salesforce.

**Screen 4 — Next-Best Action**
Highlight: Specific action recommended based on deal context. Not a generic reminder — a contextual suggestion with reasoning. Rep can accept, modify, or dismiss with one click.

---

## If Asked "Is This a Real Product?"

"FlowCRM AI is a working prototype built with TanStack Start and TypeScript. It runs locally and uses synthetic mock data — there's no real Salesforce connection in the prototype. The concept is real; the integration layer would require a Salesforce OAuth integration to ship."

---

## Interview Talking Points

**For "Tell me about a prototype you built":**
"FlowCRM AI is the clearest example of me building code, not just specs. I built a TanStack Start prototype with 6 routes, a Zustand state store, and realistic synthetic data. The goal was to validate the user flow before investing in a full Salesforce integration. The prototype taught me that the daily worklist is the right entry point — reps need to open the product and immediately know what to do. Everything else is secondary to that first screen."

**For "How do you think about CRM adoption?":**
"The adoption failure in CRM isn't a features problem — it's a value exchange problem. Reps give data; they perceive the data goes to their manager's dashboard. FlowCRM inverts this: the system uses the data to reduce the rep's manual work, surface their own deal risks earlier, and make their next action obvious. Once the rep experiences personal value from CRM data quality, the adoption flywheel turns."
