# SuccessOS AI: Customer Success Intelligence

**One-line summary:** A CS intelligence platform that synthesises churn signals, expansion opportunities, meeting notes, and QBR content from live customer data — with CSM approval required for every customer-facing action.

*All customer data, health scores, and signal examples in this case study are synthetic. SuccessOS AI is a portfolio prototype, not a shipping product.*

---

## Context and Background

Customer success is one of the highest-leverage functions in B2B SaaS, and one of the most information-overloaded. A CSM managing 30–60 enterprise accounts is expected to monitor health signals across all of them, prepare for EBRs, follow up on support escalations, identify upsell opportunities, and maintain relationship continuity — simultaneously.

The information a CSM needs to do this well exists. Usage data is in the product analytics platform. Support escalations are in Zendesk. Deal history and expansion opportunities are in Salesforce. Meeting notes are in Gong. QBR materials are in slide decks scattered across Google Drive. But synthesising that information for 40 accounts, continuously, is not humanly possible at the quality level enterprise customers expect.

The result is that health monitoring is reactive. Expansion signals get missed. QBRs are under-prepared. Churn that was visible in the data weeks before it happened goes undetected because no one had time to look.

---

## Problem Statement

CSMs in enterprise B2B SaaS are unable to maintain consistent, data-driven engagement across a large account portfolio because health signals are distributed across disconnected systems, synthesis is entirely manual, and there is no systematic mechanism for detecting expansion signals or churn precursors before they become visible at the relationship layer.

The consequence is that customer success costs too much per account to deliver at high quality, churn is detected late, and expansion opportunities surface on the customer's timeline rather than the CSM's.

---

## Users and Personas

**The CSM (Primary):** Manages 30–60 enterprise accounts. Deeply skilled at customer relationships and business conversations. Not a data analyst. Frustrated by the gap between what they know the data can tell them and how long it takes to find it. Wants briefings, not dashboards — answers, not queries.

**The CS Manager (Secondary / Approver):** Manages a team of 6–12 CSMs. Wants visibility into which accounts are at risk, which are primed for expansion, and whether the team is distributing attention correctly. Uses platform-level reporting to manage by exception.

**The VP of Customer Success (Executive Sponsor):** Cares about net revenue retention. Wants a system that can prove it detected churn risk before it manifested, and that can demonstrate the contribution of CS to expansion revenue. Primary buyer.

---

## Existing Workflow Pain

Current state:
1. CSM logs into product analytics, exports a CSV, and manually identifies accounts with declining usage (30 min/week)
2. Opens Salesforce and manually reviews open opportunities and customer notes per account (20 min/account prep)
3. Reviews open support tickets in Zendesk for their portfolio (15 min/day)
4. Searches Gong for call recordings from the last 30 days (ad hoc, when they remember)
5. Prepares QBR slide deck from scratch for each account: pulls metrics from product analytics, copies them to slides, adds narrative manually (3–4 hours per QBR)
6. Sends QBR to account owner for review via email (no tracking)

Health monitoring: reactive. QBR prep: manually intensive. Expansion identification: opportunistic.

---

## Product Thesis

A continuous health synthesis engine that monitors all signals across a CSM's portfolio in real time, surfaces the three most important developments per account per week in a digestible briefing, and drafts the QBR, follow-up email, and expansion brief — with CSM review and approval before any customer-facing output — will allow a CSM to manage a portfolio twice as large at higher quality, and will shift churn detection from reactive to predictive.

---

## Solution Overview

**Portfolio Health Digest (Weekly):** Every Monday morning, the CSM receives a digest covering their full account portfolio: health score trends, any account that crossed a risk threshold in the last week, the top three accounts needing attention, and any expansion signals detected. The digest is generated from a synthesis of product usage data, support ticket volume and sentiment, Gong call analysis, and CRM opportunity status.

**Account Intelligence Card:** For any account, on demand, the CSM can pull a card showing: 90-day health score trend, top three risk signals, top two expansion signals, meeting history and last discussed topics, open support issues, and a recommended next action. The card is generated on demand from the RAG pipeline and is current to the last ingestion cycle.

**QBR Automation:** The CSM enters an account name and QBR date. The system generates a slide-ready QBR structure: goals vs. actuals, health trend, top achievements, open risks, recommended next quarter goals, and a suggested success narrative. The CSM reviews each section, edits, and approves before the deck is shared with the customer.

**Follow-Up Drafting:** After a meeting, the CSM uploads or connects the Gong transcript. The system extracts commitments, open items, and discussion themes, drafts a follow-up email, and queues it for CSM approval and send.

**Expansion Signal Routing:** When the system detects a combination of signals that pattern-matches to historical expansion (high usage in a specific feature area, active hiring in departments that would expand the use case, positive sentiment in the last two calls), it generates an expansion brief and routes it to the CSM for review. The CSM can dismiss, tag for follow-up, or route to the AE.

---

## AI-Native Layer

- **Health scoring:** Multi-signal composite model. Usage trend (product analytics), support sentiment (NLP on tickets), engagement rate (meeting frequency vs. baseline), commercial signal (contract renewal proximity, open opportunities). Each signal weighted by customer tier and product category.
- **Churn signal detection:** Trained on historical churn patterns — which signal combinations in the 90 days prior to churn are predictive? The model surfaces accounts matching those patterns with a risk flag and a plain-language explanation of why.
- **Meeting intelligence:** Gong transcript processing with a domain-specific extraction prompt: identifies commitments, open items, customer sentiment indicators, and competitive mentions.
- **QBR generation:** Structured RAG: account-scoped retrieval from all indexed sources, fed to a QBR-specific generation prompt with section-level output and citation mapping.
- **Expansion signal detection:** Pattern matching against a predefined expansion signal taxonomy (feature depth signals, team growth signals, multi-product adjacency signals) with a confidence threshold before routing.

---

## Workflow and Agent Architecture

```
[Continuous Ingestion]
Product Analytics → Usage Index (daily)
Zendesk/Intercom  → Support Index (real-time)
Gong/Chorus       → Meeting Index (post-meeting)
Salesforce        → CRM Index (daily)

        ↓

[Health Engine]
Per-account: composite health score (daily calculation)
Risk detection: threshold crossing alerts
Expansion detection: pattern match against expansion taxonomy

        ↓ (weekly schedule)

[Portfolio Digest Agent]
1. Pull all accounts in CSM portfolio
2. Rank by health change magnitude (last 7 days)
3. Surface top 3 attention items + any threshold crossings
4. Draft digest with links to account intelligence cards

        ↓ (on demand)

[Account Intelligence Agent]
1. Retrieve all indexed signals for account (scoped RAG)
2. Synthesise into account card: health, risks, expansion, meetings, open items
3. Recommend next action

        ↓ (post-meeting trigger)

[Meeting Intelligence Agent]
1. Ingest transcript (Gong webhook or file upload)
2. Extract: commitments, open items, sentiment, competitive mentions
3. Draft follow-up email
4. Queue for CSM approval

        ↓ (expansion signal trigger)

[Expansion Brief Agent]
1. Detect expansion signal combination
2. Generate expansion brief: why now, what to propose, evidence base
3. Route to CSM for review

        ↓ (all customer-facing actions)

[CSM Approval Queue]
CSM reviews all drafted emails, QBR sections, expansion briefs
Approve → Send / Publish / Route to AE
Edit → Revised draft → Approve
Reject → Dismiss (logged)
```

---

## Key Product Decisions

**1. No customer-facing action executes without CSM approval.**
This was the non-negotiable design constraint from the start. The CSM owns the customer relationship. An AI-drafted follow-up email that is factually accurate but tonally wrong can damage a relationship that took two years to build. The approval layer is not optional and cannot be toggled off by individual users. CS managers can set policies for what requires approval vs. what can be suggested without an approval step, but no customer-facing content sends without a human in the loop.

**2. Health score is a synthesis, not a formula.**
Early designs used a weighted formula (usage × weight + support × weight + engagement × weight). This produced defensible scores but poor explainability — CSMs could not tell a customer "your health score dropped because..." in a way that was honest. The redesign made health score a synthesis: the model generates a plain-language health summary from the signal mix, and the score is derived from the summary's sentiment and urgency classification. This improved CSM trust and customer-facing usefulness.

**3. Expansion signals route to CSM, not AE, first.**
The first prototype routed expansion signals directly to the AE. This created conflict — the AE would reach out to the customer before the CSM was aware, which looked disorganised. The routing was changed: expansion signals go to the CSM first, with an option to loop in the AE. This aligned with how high-performance CS teams already operated.

**4. QBR generation is section-by-section, not full-deck.**
Generating a full QBR deck at once looked impressive in demos but produced poor review behaviour — CSMs would approve the whole thing too quickly or get overwhelmed and abandon the review. Breaking the generation into discrete sections (goals, health, achievements, risks, next quarter) with separate review steps for each produced higher-quality final output and higher CSM confidence in what was being shared.

---

## Metrics and Success Criteria

**Primary:**
- Net Revenue Retention rate (NRR) for accounts managed with SuccessOS vs. control group (12-month measurement)
- Churn prediction accuracy: % of churned accounts that had a risk flag ≥30 days before churn notice (target: >70%)
- Time to QBR prep: hours per QBR baseline vs. post-launch (target: 80% reduction)

**Secondary:**
- Portfolio digest open rate (CSM engagement with weekly briefing)
- Expansion brief acceptance rate (CSM routes to AE vs. dismisses)
- Follow-up email override rate (how often CSMs substantially edit the drafted email — signals for generation quality)

**Leading indicators:**
- Health score update latency (<24 hours from ingestion trigger)
- Transcript processing latency (follow-up draft available <30 minutes after call end)
- Signal ingestion coverage (% of accounts with all four signal sources indexed)

---

## Risks and Guardrails

**False positive churn signals:** If the system flags too many accounts as at-risk and they do not churn, CSMs will stop acting on flags — the classic precision-recall tradeoff. Target precision >70% at threshold. Calibrate threshold against historical churn data before go-live.

**Missed signals from thin accounts:** Accounts with low product usage and infrequent meetings have sparse signals. Health synthesis for these accounts is less reliable. Flag as "insufficient signal coverage" in the account card rather than generating a low-confidence health score.

**CSM approval fatigue:** If the approval queue becomes too long, CSMs will start bulk-approving without reading, which defeats the purpose of HITL. Monitor average time per approval (target: >45 seconds, indicating real review). Queue management: show only the highest-priority approvals above the fold. Batch low-urgency items (e.g., draft follow-ups) into a weekly batch review.

**PII in meeting transcripts:** Customer meeting recordings contain sensitive business information. Access control is account-scoped — CSMs can only access transcripts for their own accounts. Transcripts are not cross-referenced across customer accounts.

---

## GTM and Adoption Considerations

Target: Enterprise B2B SaaS companies with 20–200 CSMs, $10M–$200M ARR, existing Gong or Chorus deployment and a CRM. The QBR automation is the highest-impact entry point because the pain is universal, the time savings are quantifiable, and the output (the QBR deck) is visible to multiple stakeholders.

Pilot design: Pair 6 CSMs (3 on SuccessOS, 3 control), run for two quarters, measure QBR prep time and NRR delta. The pilot needs to be long enough to capture at least one renewal cycle per cohort.

Pricing model (illustrative): Per-CSM-seat per-month, tiered by accounts managed. Enterprise tier includes on-premise data processing (for companies with strict data residency requirements on customer call recordings).

---

## What I Would Build First (MVP Wedge)

QBR automation only. Three signal sources: product analytics, CRM, and Gong transcripts. Section-by-section review. No health score, no digest, no expansion detection.

QBR prep is the pain point that every CSM can immediately articulate, the time savings are measurable from day one, and the output is visible to CSM management and customers — making adoption evidence easy to collect. Health monitoring and expansion detection are phases two and three.

---

## What I Would Measure (Instrumentation Plan)

Week 1: QBR draft generation rate, CSM time from request to approved deck

Week 2: Section-level override rate (which QBR sections get edited most?)

Month 1: CSM self-reported satisfaction with QBR quality, number of QBRs completed with SuccessOS vs. prior period

Month 2: Customer-facing QBR satisfaction (optional post-QBR survey to customers)

Month 3: NRR delta for SuccessOS accounts vs. control, if pilot cohort is large enough for a signal

Ongoing: Approval queue engagement time (proxy for review quality), signal ingestion latency

---

## Lessons and Tradeoffs

The most important design insight in building CS intelligence tooling is that CSMs do not trust tools that make them look uninformed in front of customers. Any AI output that is factually wrong, tonally off, or that reveals information the CSM was not aware of — in a customer interaction — destroys trust in the tool faster than any positive experience builds it.

This means the trust bar for customer-facing output is higher than for internal output. The approval layer is the product feature that makes customer-facing AI viable in CS, not just a compliance checkbox.

The tradeoff between comprehensiveness and actionability is also real. A weekly digest covering all 40 accounts with 10 signals per account produces paralysis, not action. The design principle was: surface the three most important things per week, with deep drill-down available on demand. Defaulting to a narrow, high-signal summary was consistently more valued by CSMs than a comprehensive but overwhelming feed.

---

## Related Resources

- [Customer Success Agent Blueprint](/agent-workflow-blueprints/customer-success-agent.md)
- [Customer Success Intelligence Prompts](/prompt-library/customer-success-intelligence-prompts.md)
- [HITL Governance Framework](/hitl-governance/human-in-the-loop-design.md)
- [SuccessOS AI GitHub Project](https://github.com/siddharthjaiswal1993-spec/success-os-ai)
