# SuccessOS AI — Walkthrough Guide

**GitHub:** https://github.com/siddharthjaiswal1993-spec/Success_OS_AI

---

## Demo Goal

Show how an AI-native CS intelligence platform detects churn risk, surfaces expansion signals, processes meeting notes into action items, and generates QBR briefs — with CSM approval required before any customer-facing output.

---

## What Problem This Demonstrates

Customer Success teams are drowning in signals — health scores, NPS, product usage, support tickets, meeting notes, contract renewal dates — spread across Gainsight, Salesforce, Zendesk, and their own notes. The result: reactive CS. SuccessOS is a concept for a proactive, AI-assisted CS platform that gives CSMs a prioritized daily view of what matters and why.

---

## Primary Persona

**Customer Success Manager at a mid-market SaaS company.** Owns 30 accounts. Spends 6+ hours per week manually reviewing health dashboards, preparing for QBRs, and writing follow-up emails. Highest anxiety: missing a churn signal in a large account until it's too late to intervene.

---

## Loom Script (~3 minutes)

"Let me walk you through SuccessOS AI — a customer success intelligence platform concept.

The CS problem I'm solving isn't tooling — most CS teams have too many tools. The problem is signal synthesis. A CSM managing 30 accounts is monitoring health scores, product usage, support ticket volume, NPS surveys, and renewal dates across three or four systems. There's no unified 'what do I do today and why' view.

SuccessOS is an AI-native intelligence layer that synthesizes these signals into a prioritized CSM daily brief, surfaces churn risk and expansion opportunity, processes meeting transcripts into structured follow-up items, and generates draft QBR briefs for CSM review.

The core design principle I built around: CSM approval before any customer-facing output. The AI drafts; the CSM decides. This is non-negotiable in CS because customer relationships are high-trust and the cost of a wrong automated message is very high.

Let me show you what I built. The prototype covers the key surfaces: the daily work queue, account health detail, meeting intelligence, and QBR generation.

The daily work queue is the primary entry point. Every morning, the CSM sees a prioritized list of accounts that need attention today — not a static list of all 30, but a dynamically ranked queue based on composite risk signals. Each item has a reason ('Support ticket volume up 40% in 14 days' or 'No product login in 21 days') and a recommended action.

When the CSM clicks into an account, they see the health breakdown, the signal timeline, and the AI-generated recommended next action. The CSM can approve, edit, or dismiss it.

The meeting intelligence feature processes call transcripts into structured follow-up items, decisions, and risk signals — and routes them to the appropriate account record.

The QBR generator takes account data and produces a structured QBR brief with usage trends, health narrative, open risks, and recommended next commitments. The CSM reviews and edits before it goes to the customer."

---

## Click Path (Prototype)

1. Open the SuccessOS AI prototype (link in repo README)
2. Show daily work queue — prioritized account list with reason codes
3. Click into a flagged account — show health breakdown + signal timeline
4. Navigate to meeting intelligence — show transcript → action item extraction
5. Show QBR generator — show draft brief with CSM approval step

---

## Key Screens

**Screen 1 — Daily Work Queue**
Highlight: 5-7 accounts flagged today out of 30 total, each with a specific reason and risk level. This is the value prop in one screen: AI does the triage; CSM does the work. Not a 30-item list of everything.

**Screen 2 — Account Health Detail**
Highlight: Composite health score decomposed into signal categories (product adoption, support, NPS, engagement). Each signal cites source and date. Trend line shows direction of travel. Recommended next action is AI-generated but requires CSM approval.

**Screen 3 — Meeting Intelligence**
Highlight: Uploaded transcript → structured output: 3 follow-up items, 2 decisions, 1 risk flag. Each mapped to a person and due date. One-click to add to CRM. This is the highest-frequency use case — every meeting becomes a structured record.

**Screen 4 — QBR Brief Generator**
Highlight: AI-generated QBR draft with usage trends, health narrative, and success metrics. Clear "CSM Review" gate before any version is shared with the customer. The draft is a starting point, not a final output.

---

## If Asked "Is This a Real Product?"

"No — SuccessOS AI is a portfolio concept using synthetic account data and mock scenarios. The UI prototype demonstrates how I'd design the user experience. It's not connected to any real CS platform and doesn't reflect the roadmap of any company."

---

## Interview Talking Points

**For "How do you approach HITL design?":**
"In SuccessOS, the CSM approval gate is the product. I designed it so that no AI-generated content ever reaches a customer without CSM review and explicit approval. This isn't just a risk mitigation decision — it's the adoption strategy. CSMs will use an AI tool that augments their judgment; they won't use one that sends emails to their accounts without their knowledge. The HITL gate builds trust, and trust is what enables CSMs to gradually delegate more to the system over time."
