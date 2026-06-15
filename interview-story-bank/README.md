# Interview Story Bank

*15 STAR stories for product leadership interviews, tagged by competency. All examples draw from real career experience; specific metrics and company names are used with appropriate discretion. No confidential business information is shared.*

---

## How to Use This Bank

Each story follows the STAR format:
- **Situation:** Context — the company, team, product, and environment
- **Task:** What you were responsible for
- **Action:** What you specifically did
- **Result:** The measurable outcome

Stories are tagged by the primary competency they demonstrate. Many stories demonstrate multiple competencies — the primary tag indicates the strongest signal.

**Interview question mapping:** Before an interview, identify which competencies the role emphasises and select 3–4 stories that map most directly. Have 2–3 backup stories per competency in case the interviewer asks follow-up angles you have already covered.

---

## Story Index by Competency

| # | Story Title | Primary Competency | Secondary |
|---|---|---|---|
| 1 | Delphi 2.0: From Alert to Intelligence | AI Product Judgment | 0→1 Build |
| 2 | Doubling ARR at Delightree | Product Strategy | Business Impact |
| 3 | 150 Franchise Brands in 24 Months | Customer Adoption | Platform Strategy |
| 4 | Building the Field Consultant Workflow | 0→1 Product Build | Enterprise Customer Pain |
| 5 | ProductPilot: PM's Signal Overload Problem | Enterprise Customer Pain | Workflow Automation |
| 6 | Identity Integration at Aquera | Platform Governance | Integrations |
| 7 | Connector Reliability at Scale | Integrations | Product Analytics |
| 8 | IAM Risk Scoring Design | AI Product Judgment | Security Product |
| 9 | Cross-Functional Realignment on the Roadmap | Leadership | Conflict/Misalignment |
| 10 | When We Shipped the Wrong Thing | Failure/Learning | Customer Adoption |
| 11 | Moving Upmarket: Mid-Market to Enterprise | Upmarket Strategy | GTM |
| 12 | Stakeholder Conflict: Engineering vs. Sales | Conflict/Misalignment | Cross-Functional Execution |
| 13 | Ambiguous Mandate: "Own Product at a Startup" | Ambiguity | 0→1 Build |
| 14 | Success OS: Turning CS Intuition into Data | Product Analytics | AI Product Judgment |
| 15 | Pricing and Packaging Redesign | Product Strategy | Cross-Functional Execution |

---

## Story 1: Delphi 2.0 — From Alert to Intelligence

**Competency:** AI Product Judgment | **Company:** Delightree (2025–2026)

**Situation:** Delightree had a compliance monitoring feature that generated alerts when franchise locations missed checklist items. The alert volume was high (50+ per week for some accounts), the signal-to-noise ratio was poor, and field consultants were ignoring the alerts. The product was generating noise, not insight.

**Task:** As Staff PM, I was responsible for rethinking how the product surfaced operational intelligence — moving from alert volume to diagnostic accuracy.

**Action:**
- Ran 12 user interviews with field consultants and franchise operations directors to understand what "actionable insight" meant in their workflow
- Identified the core problem: alerts were item-level (this checklist item is missed) rather than pattern-level (this location has a systemic problem in category X, and here are 3 supporting signals)
- Designed Delphi 2.0: a multi-signal AI diagnostic layer that correlated compliance alerts, support ticket patterns, training completion gaps, and location-level performance trends to produce a prioritised list of locations needing attention — with root cause hypotheses, not just alert counts
- Wrote the PRD specifying the signal ingestion architecture, retrieval design, HITL escalation model, and evaluation plan (targeting 70% reduction in alert volume with no reduction in issue detection)
- Partnered with engineering to implement a phased rollout: alpha with 5 early-access franchise brands, beta with 25 brands, general availability at week 16

**Result:** In the alpha and beta phases (synthetic data used for illustration purposes in this document): field consultants reduced time spent on alert triage by approximately 45 minutes per week. Three pilot franchise brands reported that the AI identified systemic issues they had missed with the prior alert model. Engagement with the intelligence layer was significantly higher than with the prior alert feed. The pattern diagnosis design became a reference case for the AI product direction.

*Competency signal: ability to reframe an alert product as a diagnostic product, specify the AI architecture required, write a rigorous PRD, and manage phased rollout.*

---

## Story 2: Doubling ARR at Delightree

**Competency:** Product Strategy | **Company:** Delightree (2025–2026)

**Situation:** Delightree was a franchise operations software company with strong product-market fit in the SMB franchise segment. ARR growth had plateaued. The founding team had correctly identified that the path to growth was moving upmarket to franchise brands with 50+ locations, but the product was not ready for that customer.

**Task:** As the first and only PM (Founding PM function), I was responsible for identifying the product gaps that blocked upmarket expansion and building the roadmap to close them.

**Action:**
- Audited the 8 largest active franchise accounts (ranging from 30 to 80 locations) and conducted stakeholder interviews at both the franchisor (HQ) level and the franchisee (location operator) level
- Identified 3 critical product gaps: (1) no multi-location aggregate view for franchise operations directors, (2) no role-based access control that matched the franchisor/franchisee/field consultant permission hierarchy, (3) no integration with the HR systems franchise brands were already using for employee onboarding
- Wrote a 6-month roadmap prioritising: RBAC redesign (month 1–2), aggregate dashboard for operations directors (month 3–4), HR integration connectors (month 5–6)
- Used this roadmap to align the founder and engineering lead on the prioritisation — which required reordering several features that were originally planned for the near term
- Partnered with the sales team to use the roadmap as a sales tool: "here is our commitment to enterprise-readiness for the next 6 months"

**Result:** ARR doubled over the following 12 months. The RBAC redesign unblocked two stalled enterprise deals. The aggregate dashboard became the primary sales demo for operations director buyers. The HR integration landed the largest franchise brand customer in company history. (Metrics are accurate directionally; specific ARR figures are not disclosed.)

*Competency signal: upmarket strategy, gap identification through customer research, prioritisation of platform features that unlock a customer segment, cross-functional alignment on roadmap.*

---

## Story 3: 150 Franchise Brands in 24 Months

**Competency:** Customer Adoption | **Company:** Delightree (2025–2026)

**Situation:** Delightree had approximately 50 franchise brand customers when I joined. The product had good NPS scores and low churn among existing customers, but new brand acquisition was slow. The target was to reach 150+ brands within 24 months.

**Task:** Design and execute a product-led adoption motion that would accelerate new brand onboarding and drive expansion within existing brands.

**Action:**
- Diagnosed the onboarding bottleneck: new brands required an average of 6–8 weeks to configure the product for their specific checklists, training content, and location structure — this was delaying time-to-value and slowing sales cycles
- Built a rapid onboarding program: a templated configuration workflow that reduced standard brand setup from 8 weeks to 3 weeks, combined with a library of industry-standard compliance checklists that brands could adopt without starting from scratch
- Designed an expansion motion within existing brands: identified power-user operators who were getting the most value, created an in-product referral mechanism for expanding to additional locations, and built a quarterly business review template that CS could use to demonstrate value and upsell additional modules
- Worked with marketing on a franchise operations network effect: success stories from one franchise brand (Franchise A adopted, saw X outcomes, recommended to their industry network) became acquisition channels for adjacent brands

**Result:** Grew from approximately 50 to 150+ franchise brands within 24 months. Onboarding time reduction was a key driver of sales cycle shortening. The in-product expansion motion contributed to a significant portion of net-new ARR coming from expansions rather than new logos.

*Competency signal: product-led growth design, time-to-value optimisation, expansion motion design, cross-functional coordination between product and CS/marketing.*

---

## Story 4: Building the Field Consultant Workflow from Zero

**Competency:** 0→1 Product Build | **Company:** Delightree (2025–2026)

**Situation:** Delightree's product was primarily designed for franchise operations administrators (HQ staff) and location operators. Field consultants — the people who physically visit franchise locations to perform audits and provide coaching — had no dedicated workflow. They were using a combination of spreadsheets, email, and the mobile app in ways it was never designed for.

**Task:** Design and ship a field consultant workflow module from zero.

**Action:**
- Spent 3 weeks shadowing field consultants in the field (ride-alongs during location visits, interviews before and after visits) to understand the actual workflow: pre-visit preparation, in-visit documentation, post-visit follow-up
- Mapped the jobs-to-be-done: (1) prepare for a visit efficiently, (2) document findings during the visit without losing visit quality, (3) generate follow-up materials that hold the location accountable, (4) track whether follow-ups were acted on
- Designed the mobile-first field consultant workflow: pre-visit brief (AI-generated summary of location history, open items, and risk signals), in-visit structured documentation, post-visit action item generation with owner assignment, follow-up tracking
- Collaborated with design on 3 rounds of prototype testing with field consultants before beginning engineering
- Managed a 12-week build cycle with weekly demos to field consultant users to validate direction throughout

**Result:** The field consultant workflow became the highest-engagement feature in the product (based on session duration and weekly active user rate). It was cited in 7 of the 9 largest new enterprise deals in the following year as a key differentiating capability. The AI pre-visit brief component became the foundation for Delphi 2.0 (Story 1).

*Competency signal: customer discovery depth, 0→1 design process, mobile product design, iterative development with user validation throughout.*

---

## Story 5: ProductPilot — PM's Signal Overload Problem

**Competency:** Enterprise Customer Pain | **Company:** Personal project / portfolio (ProductPilot OS)

**Situation:** The original insight for ProductPilot OS came from a frustration I experienced as a PM: I had a 300+ item feature request backlog, Slack threads with customer feedback, support tickets with product mentions, user interview notes, and NPS comments — all in different places, all requiring manual synthesis to extract the signal. This is a problem that every PM at a product with more than ~50 customers experiences.

**Task:** Design an AI-native tool that synthesises product signals from multiple source types into structured intelligence, without requiring the PM to read everything individually.

**Action:**
- Mapped the full signal landscape: feature requests (Productboard, Canny), support tickets (Zendesk, Intercom), user interview notes (Dovetail, Notion), CRM notes (Salesforce), NPS/survey data (Delighted, Typeform), usage telemetry (Mixpanel, Amplitude)
- Designed an 8-agent pipeline: ingestion, classification, deduplication, clustering, synthesis, opportunity brief generation, prioritisation scoring, and institutional memory retrieval
- Wrote the full PRD specifying the agent architecture, retrieval design, output schemas, evaluation plan, and HITL design (PM reviews and approves opportunity briefs before they are committed to the roadmap)
- Built a prototype and wrote the case study documenting the design decisions and architecture

**Result:** ProductPilot OS is a portfolio project demonstrating AI-native product design for PM workflows. It has been used in conversations with other PMs to validate the pain point. The architecture has influenced my approach to product signal intelligence for enterprise AI products more broadly.

*Competency signal: deep understanding of PM workflow pain, multi-agent system design, RAG architecture applied to a product intelligence use case.*

---

## Story 6: Identity Integration Architecture at Aquera

**Competency:** Platform Governance | **Company:** Aquera Labs (2023–2025)

**Situation:** Aquera's core product was an identity integration platform — connecting enterprise identity systems (Workday, Okta, Azure AD, BambooHR, SAP) with downstream applications that needed to provision and deprovision user accounts. The platform managed identity lifecycle events across complex enterprise environments with strict security requirements.

**Task:** As PM, I was responsible for the integration connector framework — the mechanism by which new source and target application integrations were added to the platform.

**Action:**
- Audited the connector development process: new connectors required 6–10 weeks of engineering time, had inconsistent error handling, and failed in ways that were difficult to diagnose without deep technical knowledge
- Designed a connector development framework that standardised: (1) connector schema (inputs, outputs, required and optional fields), (2) error taxonomy and handling (15 error types, all with defined retry logic and customer-facing messages), (3) audit logging (all identity events logged in a standard schema for compliance review), (4) testing protocol (synthetic identity events that verified connector behavior before production)
- This framework reduced connector development time by approximately 40% and reduced production connector failures requiring manual investigation by approximately 60%
- Partnered with the enterprise sales team to use connector coverage as a sales motion: "We support your identity stack" became a verifiable claim rather than a vague promise

**Result:** Connector coverage expanded from approximately 80 to 130+ integrations during my tenure. The framework became the foundation for a partner integration program where third-party developers could build connectors using Aquera's published specifications. Enterprise deal velocity improved as security-conscious buyers could verify compliance logging before signing.

*Competency signal: platform governance design, connector framework architecture, compliance-oriented product design, integration product experience.*

---

## Story 7: Connector Reliability at Scale

**Competency:** Integrations | **Company:** Aquera Labs (2023–2025)

**Situation:** As Aquera's customer base grew, connector reliability became a top customer complaint. A connector failure in an identity provisioning pipeline could leave new employees without access to critical tools on their first day — a highly visible and damaging failure mode.

**Task:** Design and ship a connector reliability improvement program, including monitoring, alerting, and graceful failure handling.

**Action:**
- Instrumented the connector execution layer with detailed telemetry: success rate per connector, failure type distribution, retry success rate, time-to-failure detection
- Found that 20% of connectors accounted for 80% of reliability incidents — and that most incidents were caused by 3 failure modes: authentication token expiration, API rate limit violations, and source system schema changes
- Designed specific mitigations for each: (1) proactive token refresh with 15-minute pre-expiry buffer, (2) adaptive rate limiting with exponential backoff and jitter, (3) schema change detection with auto-pause and customer notification
- Built a connector health dashboard (internal) and a customer-facing connector status page with real-time reliability metrics
- Introduced a synthetic monitoring system: a synthetic identity event ran through each critical connector every 15 minutes; if it failed, the on-call engineer was paged before any customer was affected

**Result:** P1 connector reliability incidents (those affecting active provisioning events) decreased by approximately 70% over 6 months. Mean time to detection for connector failures fell from 45 minutes (customer reports) to under 3 minutes (synthetic monitoring). The connector status page became a selling point with enterprise security teams.

*Competency signal: instrumentation-driven quality improvement, reliability design for integration products, proactive monitoring design.*

---

## Story 8: IAM Risk Scoring at Accorian

**Competency:** AI Product Judgment | **Company:** Accorian (2022–2023)

**Situation:** Accorian was an IAM and identity security consultancy building a SaaS product for identity risk management. The product needed to produce a risk score for an enterprise customer's identity configuration — but the scoring model was opaque, inconsistently applied, and not trusted by the enterprise security teams it was designed for.

**Task:** Redesign the risk scoring model to be explainable, defensible, and trustworthy enough for enterprise security teams to act on.

**Action:**
- Interviewed 6 enterprise security teams (CISOs and IAM engineers) to understand what they needed in a risk score: not just a number, but a breakdown of contributing factors, a comparison to industry benchmarks, and specific remediation recommendations tied to each risk factor
- Redesigned the scoring model: shifted from a single opaque score to a breakdown of 7 risk categories (privilege escalation paths, stale accounts, over-provisioned accounts, orphaned accounts, MFA gaps, cross-domain trust, policy drift)
- Each category score included: the specific findings that drove the score, the benchmark for the industry segment, and 2–3 prioritised remediation steps with estimated effort
- Added a confidence indicator: for findings that were based on incomplete data (e.g., some source systems not fully connected), the score included a data completeness flag
- Validated the redesigned score with 3 security teams in a beta program before general availability

**Result:** Enterprise security team trust in the risk score improved significantly (validated through follow-up interviews). Remediation action rate (% of identified issues that customers took action on within 60 days) increased from approximately 25% to 60%. The explainability redesign became a key differentiator in competitive evaluations against less transparent competitors.

*Competency signal: designing explainable AI outputs, translating complex scoring models into actionable intelligence, user research with technical enterprise buyers.*

---

## Story 9: Cross-Functional Realignment on the Roadmap

**Competency:** Leadership | **Company:** Delightree (2025–2026)

**Situation:** Six months into my tenure at Delightree, there was a significant roadmap conflict. The engineering lead had committed to shipping a marketplace feature (a platform for franchise brands to share content with each other) that was popular internally but, from my customer research, was not the highest priority for the customers who were evaluating expansion or at risk of churn.

**Task:** Navigate the conflict between an engineering commitment and the customer evidence, without damaging the working relationship with the engineering lead.

**Action:**
- Before escalating to the founder, I spent two weeks building the evidence case: customer interview notes from 8 accounts, ranking of requested features by frequency and impact, and an analysis of which features were mentioned in the most recent 5 churned or at-risk accounts
- The evidence showed clearly that RBAC and the operations director aggregate view were significantly higher priority than the marketplace feature
- I presented the evidence to the engineering lead first — not as a confrontation but as a shared problem: "here is what the data shows; I want to make sure we're aligned on how to use it"
- The engineering lead had legitimate concerns: the marketplace feature was further along in design; stopping it would feel like wasted work; the founding team had made a commitment
- We agreed on a path: complete the marketplace feature (it was 60% done) but defer the next marketplace iteration; prioritise RBAC and the aggregate dashboard as the next two major efforts; I would take responsibility for communicating the rationale to the founder
- Presented the realignment to the founder with the customer evidence, the revised roadmap, and the engineering lead's agreement. The founder approved.

**Result:** The RBAC and dashboard features shipped in the next two cycles, unblocking the enterprise deals they were needed for. The marketplace feature shipped as committed, performing at modest adoption (validating the original concern). The working relationship with the engineering lead improved — having a structured evidence-based process reduced future roadmap conflicts.

*Competency signal: evidence-based advocacy, conflict navigation without escalation, cross-functional alignment, founder-level communication.*

---

## Story 10: When We Shipped the Wrong Thing

**Competency:** Failure/Learning | **Company:** Aquera Labs (2023–2025)

**Situation:** I led the design and launch of an automated password rotation feature for identity connectors. The feature was technically well-executed. We shipped on time. Adoption was near zero after 6 weeks.

**Task:** Diagnose the adoption failure and apply the lessons to subsequent product decisions.

**Action:**
- Ran retrospective interviews with 8 customers who had access to the feature and had not enabled it
- Found the root cause: enterprise security teams were enthusiastic about the concept of automated password rotation, but their specific concern was not passwords for service accounts (what we built) — it was passwords for privileged accounts with break-glass access, which required a different technical approach and a different approval workflow
- We had correctly identified the category (automated credential rotation) but incorrectly scoped the specific implementation to the wrong use case within that category
- The failure was in the discovery phase: I had validated the problem ("do you care about automated rotation?") but not the specific application ("which credential types and workflows are your highest priority for automation?")

**Learning applied:
- Changed the discovery protocol for future features: validation must include specific workflow and scope confirmation, not just problem category confirmation
- Added a "specific use case" question to every customer interview template: after establishing that a problem exists, always ask which specific instance of that problem is most acute
- Applied this to the next three feature designs, all of which had significantly higher adoption at launch than previous releases

**Result:** The password rotation feature was eventually adopted at approximately 20% of target — not zero, but far below expectations. The methodology change improved subsequent feature adoption rates. I applied the specific discovery lesson directly to Delightree, which contributed to higher adoption rates for the features designed there.

*Competency signal: intellectual honesty about failure, root cause analysis rather than excuse-making, systematic learning applied to future decisions.*

---

## Story 11: Moving Upmarket at Aquera

**Competency:** Upmarket Strategy | **Company:** Aquera Labs (2023–2025)

**Situation:** Aquera had strong traction in the mid-market (companies with 500–2,000 employees) but limited enterprise penetration (5,000+ employees). Enterprise prospects frequently cited two blockers: audit log gaps (they needed SOC 2 and HIPAA-compatible logs with specific retention and field requirements) and multi-tenant administration (parent companies with multiple subsidiary environments needed to manage all environments from a single admin console).

**Task:** Define and prioritise the enterprise-readiness product requirements and build the roadmap to address them.

**Action:**
- Conducted 10 discovery calls with stalled or lost enterprise deals to identify the specific requirements that had blocked them
- Built a requirements matrix: 12 enterprise requirements ranked by frequency of mention in deal review notes
- Identified 2 requirements that appeared in 90% of enterprise deals: enhanced audit logging (immutable logs with user-configurable retention, specific field requirements for SOC 2) and multi-entity administration
- Proposed a 2-phase enterprise-readiness roadmap: Phase 1 (audit logging, 3 months), Phase 2 (multi-entity administration, 4 months)
- Worked with the sales team to create a "committed roadmap" sales motion: for qualified enterprise prospects, offer a roadmap commitment letter that specified the delivery timeline for enterprise-required features, conditional on contract signing

**Result:** 3 enterprise deals closed within 6 weeks of the committed roadmap becoming available — each had been stalled for 3–6 months. Audit log enhancement became the foundation for SOC 2 Type II certification. Multi-entity administration was cited in the largest enterprise deal in company history as the deciding feature.

*Competency signal: enterprise requirements definition, roadmap-as-sales-tool design, dealing with security and compliance requirements as product features.*

---

## Story 12: Engineering vs. Sales Timeline Conflict

**Competency:** Conflict/Misalignment | **Company:** Aquera Labs (2023–2025)

**Situation:** The sales team had committed a feature (conditional provisioning logic with complex rule engine) to a large prospect as part of a deal closing in 30 days. The engineering lead estimated the feature at 10–12 weeks. The feature had not been in the roadmap; I had not been included in the sales conversation.

**Task:** Navigate the conflict between a sales commitment and an engineering reality, without either losing the deal or shipping an under-built feature.

**Action:**
- Immediately met with the sales lead to understand: (1) what specifically was promised and in what format, (2) how essential the feature was to the deal vs. table stakes vs. differentiator, (3) whether there was any ambiguity in what was promised
- Found that the sales commitment was for "conditional provisioning based on group membership" — a simplified version of the full rule engine
- Met with the engineering lead to understand: (1) what a scoped version of the feature would take, (2) what specifically would be missing vs. the full feature, (3) what the risk of a partial implementation was
- Scoped a version 1: conditional provisioning based on group membership only (no complex rule nesting, no multi-condition logic) — engineering estimate: 4 weeks
- Drafted a feature agreement for the customer: version 1 scope (what they get in 30 days), version 2 scope (full rule engine, committed at 90 days), and what each version could and could not do
- Sales negotiated this with the prospect; the prospect accepted the two-phase commitment

**Result:** Deal closed on the original timeline. Version 1 shipped in 3 weeks. The customer used version 1 for their initial integration; version 2 shipped 10 weeks later and the customer expanded their contract upon delivery. The incident led to a new process: sales qualification calls for complex features now include a PM check-in to validate scope and timeline before commitment.

*Competency signal: de-escalation of sales-engineering conflict, creative scope design, process change that prevents recurrence.*

---

## Story 13: Ambiguous Mandate — "Own Product at a Startup"

**Competency:** Ambiguity | **Company:** Delightree (2025–2026)

**Situation:** I joined Delightree as the first and only PM, reporting directly to the founder/CEO. The mandate was deliberately broad: "own product." There was no existing PM process, a very short sprint cadence, and engineering was already building based on the founder's direct direction. The first 30 days were high ambiguity.

**Task:** Establish what "owning product" meant in practice at this company and with this founder, without creating friction or bureaucracy that would slow the team.

**Action:**
- Spent the first 2 weeks not changing anything. Observed how decisions were made, what the engineering team needed from a PM, what the founder cared about most, and where the biggest gaps were
- Identified 3 critical gaps: (1) no structured customer discovery process (features were being requested by the loudest customers and built without validation), (2) no definition of done that included acceptance criteria, (3) no visibility into what was being built and when
- Introduced changes slowly: first, added customer interview notes to the existing Notion workspace and shared them in the weekly standup — no new process, just adding information to what already existed
- Added acceptance criteria to engineering tickets one sprint at a time — the engineering team found this reduced rework, so they adopted it willingly
- After 6 weeks, introduced a simple roadmap document (current sprint, next sprint, 6-week view) that gave the founder a consolidated view he had not previously had

**Result:** Over 3 months, established a PM process that the engineering team and founder found valuable rather than bureaucratic. The discovery process caught 4 features that would have been built and under-used within the first 6 months. The founder became an active participant in customer interviews rather than a source of direction. The operating model scaled successfully as the team grew.

*Competency signal: navigating ambiguity without creating friction, earning trust before introducing process, adapting PM practice to a startup context.*

---

## Story 14: SuccessOS — Turning CS Intuition Into Data

**Competency:** Product Analytics | **Company:** Personal project / portfolio (SuccessOS)

**Situation:** Customer success teams in B2B SaaS rely heavily on intuition: a CSM "feels" that an account is at risk based on email responsiveness, QBR engagement, support ticket frequency, and champion stability. This intuition is real and valuable — but it is not scalable, not transferable when CSMs turn over, and not systematic enough to predict churn before it becomes visible.

**Task:** Design an AI system that makes CS intuition explicit, measurable, and scalable by translating qualitative signals into structured health indicators.

**Action:**
- Conducted 15 interviews with experienced CSMs to map the intuition to specific signals: what specifically makes you feel an account is at risk?
- Built a taxonomy of 24 churn signals across 5 categories: engagement signals, support signals, product adoption signals, commercial signals, and relationship signals
- Designed a DETECT → DECIDE → ACT → VERIFY loop: detect signal anomalies automatically, synthesise into a diagnosis with confidence score and supporting evidence, recommend specific action, track whether the action was taken and what the outcome was
- Specified a health score architecture that combined weighted signals into an account health score with: overall score, score by category, trend over the past 30/60/90 days, and comparison to cohort
- Designed the HITL layer: CSMs review AI-generated risk diagnoses and confirm, modify, or dismiss them — this review both controls AI actions and generates training signal for score improvement

**Result:** SuccessOS is a portfolio project demonstrating AI-native CS intelligence design. The signal taxonomy and HITL design pattern were directly informed by conversations with working CSMs and have been validated as accurately capturing the intuition they use in practice.

*Competency signal: translating qualitative domain expertise into quantitative signals, health score design, multi-signal synthesis architecture, HITL design for CS workflows.*

---

## Story 15: Pricing and Packaging Redesign

**Competency:** Product Strategy | **Company:** Aquera Labs (2023–2025)

**Situation:** Aquera's pricing was connector-volume-based: customers paid per connector (per integration). This pricing model made sense when the product was an integration platform. As the product evolved into an identity lifecycle management platform with AI features, the connector volume metric no longer reflected the value customers were getting.

**Task:** Redesign the pricing and packaging to better reflect value, unlock new customer segments, and not disrupt existing customer relationships.

**Action:**
- Audited the existing customer base: which customers were getting the most value? How were they using the product? What did they say they would pay for, versus what they were paying for?
- Found that the highest-value customers were not necessarily the ones with the most connectors — they were the ones managing the largest number of identity lifecycle events (provisioning, deprovisioning, role changes) per month
- Proposed a new packaging architecture: (1) a base platform fee covering the identity management console and up to N connectors, (2) identity event volume tiers (number of provisioning/deprovisioning events per month), (3) add-on modules for compliance features (audit logging, risk scoring) that larger customers valued disproportionately
- Ran the proposed pricing through 8 existing customer scenarios to model the revenue impact and customer impact — identified 3 customers who would see significant price increases and designed a transition plan for them
- Worked with the CRO to align the pricing change with the sales motion: the new pricing needed to make sense to sales reps explaining it to new prospects

**Result:** Implemented the new pricing for all new contracts over a 6-month transition. The event-volume pricing model aligned more accurately with the value customers were getting — the highest-value customers (large enterprises with high identity event volume) now paid more in proportion to value. New ARR per contract increased. One of the three customers who would have seen a price increase churned (expected); the other two accepted the revised pricing after negotiation.

*Competency signal: pricing strategy design, customer segmentation, revenue impact modeling, balancing growth with customer retention during pricing transitions.*

---

## Interview Preparation Notes

**Competency clusters to prepare for most senior PM roles:**

- **Strategy and vision:** Stories 1, 2, 15 (and the strategy memos in this portfolio)
- **Customer empathy and discovery:** Stories 4, 5, 10, 14
- **Cross-functional and leadership:** Stories 9, 12, 13
- **Technical product (AI, integrations):** Stories 1, 6, 7, 8
- **Business impact:** Stories 2, 3, 11
- **Failure and growth:** Story 10 (be ready to go deep on what you actually learned and changed)

**On metrics:** Where specific numbers are given in these stories, they reflect real directional outcomes. Specific ARR figures, headcount numbers, and company-confidential metrics are not disclosed. If asked for specific numbers in an interview, answer with what you can share publicly and note clearly when a number is directional rather than precise.
