# From Systems of Record to Systems of Intelligence

*A strategic memo on the three-wave evolution of enterprise software and where AI-native products must go next.*

---

## The Premise

Enterprise software has been through two waves and is entering a third. Most PMs building today are thinking about the second wave. The companies that win over the next decade will be building for the third.

Understanding which wave you are in — and whether the wave you are designing for matches the wave your customer is experiencing — is one of the most important strategic orientation tasks for any product leader.

---

## Wave 1: Systems of Record (1990s – 2010s)

The defining problem of enterprise software in the first wave was: information is scattered across people's heads, inboxes, and paper. The value proposition was simple — capture it, store it, search it.

CRMs were address books with structured fields. ERPs were spreadsheets that could talk to accounting. HRIS systems were paper files made digital.

The product promise was: "You will never lose the information again."

The measure of success was adoption (data entered) and retrieval (data found).

**What was not included:** any opinion about what the data meant, what should happen next, or whether anything was going wrong.

The user had to provide all the analytical layer. The software just stored and surfaced. The human synthesised, diagnosed, and decided.

---

## Wave 2: Systems of Workflow (2010s – early 2020s)

Data alone is not enough. You need it to move. The second wave of enterprise software added orchestration: approval chains, automated notifications, handoff routing, task management, conditional logic.

Salesforce became not just a contact database but an opportunity pipeline with stage gating. ServiceNow became not just a ticket tracker but a multi-step approval and routing engine. Jira became not just an issue log but a sprint workflow with blocking relationships.

The product promise shifted to: "You will never drop the ball."

Workflow automation became a competitive differentiator. Rules engines, if-then triggers, and conditional logic became core product surface area. The question was not just "what happened?" but "what should happen next, and is it happening?"

**What was still not included:** judgment. The workflow could move information through predefined stages, but it could not decide what mattered, what was at risk, or what the right action actually was. It executed the rules you wrote — it did not notice when the rules were wrong or the situation was novel.

The user still had to provide the analytical judgment. The software just routed.

---

## Wave 3: Systems of Intelligence (2023 onwards)

The third wave adds judgment.

Not judgment that must be pre-programmed into rules. Judgment that emerges from the data itself — from patterns across accounts, signals across interactions, anomalies across time series, context from documents that no rule engine was designed to read.

A system of intelligence:
- Detects risks before the human does
- Diagnoses root causes from multi-signal data
- Recommends specific actions with supporting reasoning
- Routes for human decision where uncertainty is too high
- Verifies that the action produced the intended outcome
- Learns from human overrides to improve future recommendations

This is not feature parity with systems of workflow. It is a qualitatively different kind of product. The system does not just move information — it reasons about it.

---

## Why the Transition Is Not Just Technical

The temptation is to treat systems of intelligence as a technology upgrade — add an AI API call to an existing workflow product and call it intelligent. This approach produces AI features, not AI-native products. It misses the fundamental design shift.

A system of intelligence requires:

**Different data architecture.** You need not just structured records but a live, queryable corpus of all signal types — notes, tickets, usage telemetry, external alerts, communication transcripts. The AI must be able to retrieve the relevant context for any reasoning task from this corpus on demand.

**Different design principles.** The unit is not a form field or a workflow step — it is a recommendation. Every recommendation must be explainable, auditable, and override-able. The product must show its reasoning, not just its conclusions.

**Different success metrics.** Adoption rate and records created are systems-of-record metrics. Workflow completion rate is a systems-of-workflow metric. Recommendation acceptance rate, override rate, time-to-action reduction, and business outcome lift are systems-of-intelligence metrics.

**Different PM skills.** Building a system of intelligence requires the PM to understand how the AI generates its outputs, what failure modes look like, how to evaluate quality, how to design human oversight into the loop, and how to instrument the product to detect when the AI is wrong.

---

## The Competitive Implication

Every mature vertical SaaS market will be disrupted by a competitor who ships a genuine system of intelligence before the incumbent adds AI features to their existing architecture.

The incumbent has advantages in data (more customer data = better signal) and distribution (existing customer relationships). But incumbents consistently underestimate how structurally different the third-wave design pattern is from the second-wave pattern they optimised their organisation to build.

The incumbent ships an "AI assistant" that answers questions about the data the system of record already holds. The challenger ships a system that reads every signal the customer generates, surfaces anomalies the human would never have noticed, routes them to the right person at the right time, and tracks whether the action produced the intended result.

Those are not the same product. One is a feature. One is a category.

---

## What This Means for Product Strategy

**If you are building a net-new AI-native product:** Design for Wave 3 from day one. Build the signal corpus, not the record store. Build the diagnosis engine, not the report. Build the recommendation surface, not the data view.

**If you are building AI into an existing Wave 2 product:** Be honest about which capabilities are genuinely Wave 3 (new reasoning the system did not previously provide) and which are Wave 2 with AI-generated text (natural language query over data that already existed). The former creates new value. The latter improves UX. Both are worthwhile — but do not mistake one for the other.

**If you are evaluating a competitor:** Ask not whether they have "AI" but whether their system detects things the human could not detect alone. If the answer is no, they are still in Wave 2.

---

## Where I Build

Every product in my portfolio is designed as a system of intelligence, not a system of record or a system of workflow.

The Franchise Operations AI identifies compliance risk signals across 150+ franchise locations before a field consultant's scheduled visit. ProductPilot OS synthesises product signals from five source types to surface opportunities that no PM would identify from reading the sources individually. SuccessOS detects early churn signals by correlating engagement patterns, support volume, and renewal timeline.

These systems are not answering questions about data the customer already knows about. They are finding the signal in the noise — the thing the human would have missed.

That is the value proposition of Wave 3. That is where enterprise B2B SaaS is going.

---

*This memo reflects personal views on enterprise software evolution. Related: [Workflows Are the Product, Models Are the Engine](workflows-are-the-product-models-are-the-engine.md) · [Enterprise AI Moats](enterprise-ai-moats.md) · [Portfolio Thesis](../about/portfolio-thesis.md)*
