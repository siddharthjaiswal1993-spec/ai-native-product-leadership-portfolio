# Online AI Product Metrics

**Metrics for deployed AI products: acceptance rate, override rate, task completion, latency, cost, escalation rate, and business outcome lift**

---

## Why Online Metrics Are Different from Offline Evals

Offline evals tell you whether the system works on a controlled sample. Online metrics tell you whether it is working in the real world, with real users, on the real distribution of inputs.

The gap between offline and online performance is one of the most important signals in AI product management:
- High offline performance + low online acceptance rate: the system works technically but users do not trust it, or the use case hypothesis was wrong
- Low offline performance + high online acceptance rate: users may be accepting without reading (review quality issue)
- Both high: the system is working as intended

Online metrics must be designed before launch. Instrumentation added post-launch is always missing the baseline data that makes change detection meaningful.

---

## Core Online Metrics for AI Products

### 1. Acceptance Rate

**Definition:** The fraction of AI-generated outputs that are used or approved by users without substantial modification.

**Measurement:** 
- For approval-gated outputs (HITL): % of items in the approval queue that receive a "approve" action (vs. edit or dismiss)
- For non-gated outputs (suggestions, drafts): % of outputs where the user proceeds to use the output without modifications

**Targets:**
- At launch: baseline measurement (what is it on day 1?)
- Month 3: target depends on use case. For pre-visit intelligence briefs: >80% accepted. For first-draft PRDs: >60% accepted without major edits.
- A declining acceptance rate is a red flag — investigate root cause (model quality drift, changing user needs, prompt regression, distribution shift)

**Warning:** High acceptance rate alone is not sufficient evidence of quality. Users may accept without reading. Cross-reference with override rate and error escalation rate.

---

### 2. Override Rate

**Definition:** The fraction of AI-generated outputs that are substantially edited or modified before use.

**Measurement:** Track edit events on AI-generated content. "Substantial edit" defined per product (e.g., >20% of text changed, or a specific field was modified).

**Why it matters more than acceptance rate:** Override rate tells you WHERE quality is failing. If users consistently override the "recommended action" section of the risk brief but rarely override the "evidence summary" section, that tells you which part of the generation pipeline needs work.

**Track at the section/component level**, not just at the full output level.

**Healthy patterns:**
- Override rate should be highest at launch and decline over time as the system improves and learns user preferences
- Override rate that is flat or rising over time indicates a quality issue that is not self-correcting

---

### 3. Task Completion Rate

**Definition:** The fraction of tasks initiated by users (or agents on behalf of users) that are completed successfully end-to-end.

**Measurement:** Define the "start" and "end" events for each task type. Track the funnel from start to successful completion. Track dropout points.

**Task completion rate by task type** is more informative than overall task completion. If QBR generation has 85% completion but follow-up email generation has 45% completion, that is a specific signal about the follow-up email workflow.

**Completion failure modes to track:**
- Task started but never submitted to the AI pipeline (user abandoned before triggering the AI)
- AI pipeline started but did not return output (system failure)
- Output returned but user dismissed without approving (quality failure)
- Output approved but delivery to downstream system failed (integration failure)

---

### 4. Latency

**Definition:** Time from user request (or agent trigger) to output available for review.

**Metrics:**
- P50 (median): the typical user experience
- P95: the worst experience for 95% of users (service level agreement target)
- P99: tail latency — affects users in slow regions, large contexts, or peak load

**Latency targets by workflow type:**

| Workflow Type | P50 Target | P95 Target | Why |
|---|---|---|---|
| Interactive Q&A (in-visit mode) | <1s | <2s | Real-time conversation; delay is disruptive |
| Background brief (pre-visit, pre-QBR) | <30s | <60s | Async; user can check email while waiting |
| Large synthesis (full QBR deck) | <3min | <5min | Substantial computation; expectation set by UX |
| Batch processing (nightly portfolio digest) | N/A (batch) | <10min total | Not real-time; runs while user is offline |

**Monitor latency trends.** Latency that was acceptable at launch may become unacceptable as corpus size grows and retrieval takes longer. Alert when P95 increases >25% from the prior week's average.

---

### 5. Cost Per AI Action

**Definition:** The fully loaded cost of producing one AI-assisted output or completing one AI-assisted action.

**Components:**
- Embedding cost (ingestion, at per-token rates)
- Retrieval cost (vector store query fees)
- Generation cost (input + output tokens × model price)
- Reranking cost (cross-encoder inference)
- Infrastructure overhead (serving, storage)

**Why PMs must own this metric:** Cost per action directly affects margin. If the cost of generating a pre-visit brief is $1.20 and the monthly pricing is $50/seat with 8 visits per month, the AI action cost consumes a significant fraction of revenue per seat. Knowing the break-even is a product requirement, not an afterthought.

**Cost optimisation levers (see [Cost and Latency Models](/cost-latency-models/)):**
- Model routing (smaller models for simple tasks)
- Prompt caching
- Retrieval result caching
- Context window management

---

### 6. HITL Escalation Rate

**Definition:** The fraction of AI-generated outputs that are escalated to a human reviewer (because of low confidence, rule trigger, or mandatory review requirement).

**Why it matters:** 
- Escalation rate that is too high: AI is adding overhead without reducing human work
- Escalation rate that is too low: May indicate approval fatigue — humans approving without real review

**Track by escalation type:**
- Mandatory HITL (always requires review): expected rate is 100% for certain output types
- Confidence-gated HITL (escalated because below confidence threshold): track the rate and calibrate the threshold
- Rule-triggered HITL (escalated because a specific content rule fired): track which rules are triggering and whether they are calibrated correctly

---

### 7. Business Outcome Lift

**Definition:** Measurable improvement in business outcomes attributable to AI-assisted workflows, compared to the baseline (manual process or control group).

**Examples:**
- CS intelligence: NRR improvement for accounts managed with AI-assisted weekly digest vs. control
- Pre-visit brief: Field consultant visit quality score improvement vs. control
- PRD generation: Engineering-reported PRD completeness score improvement vs. control

**Measurement methodology:**
- Pre/post comparison: measure the outcome metric before and after launch
- A/B test: ideally, run a controlled experiment with an AI-assisted group and a control group
- Matched cohort: compare AI-assisted users to a matched set of non-users

**Caution:** Business outcome attribution is hard. Many factors affect NRR beyond the CS intelligence tool. Be conservative in causal claims — "we observed a 3-point NRR improvement for the AI-assisted cohort during this period" is accurate; "the AI tool caused a 3-point NRR improvement" is likely an overstatement.

---

## Metrics Dashboard Structure

A well-designed online metrics dashboard for an AI product has three levels:

**Level 1 — Health Summary (daily check):**
- Acceptance rate (last 7 days)
- P95 latency (last 24 hours)
- Error rate (last 24 hours)
- Any active alerts

**Level 2 — Quality Trends (weekly review):**
- Override rate by component
- Task completion rate by task type
- HITL escalation rate by trigger type
- Cost per action (weekly average)

**Level 3 — Outcome Impact (monthly/quarterly review):**
- Business outcome metrics for AI-assisted cohort
- Offline eval results (from scheduled monthly eval run)
- Comparison to prior period and launch baseline

---

*See also: [Human Feedback Loop](human-feedback-loop.md) · [Agent Task Success Evals](agent-task-success-evals.md) · [AI Unit Economics](/cost-latency-models/ai-unit-economics.md)*
