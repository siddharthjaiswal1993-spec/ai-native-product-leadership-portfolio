# Portfolio Thesis: From Systems of Record to Systems of Intelligence

---

## The Three Waves of Enterprise Software

Enterprise software has evolved through three distinct waves, each built on the infrastructure of the previous one.

**Wave One: Systems of Record (1990s–2010s).** The defining challenge was capturing and organising information. Salesforce captured customer data. SAP captured financial data. Jira captured engineering work. Workday captured people data. The value proposition was simple: instead of spreadsheets and filing cabinets, a shared database with structured access and basic reporting. Competitive differentiation came from completeness — which system had the most fields, the most integrations, the best reporting.

The limitation of systems of record is that they require humans to do all the interpretation. The data sits there. Someone has to pull a report, read it, form a judgment, and act. The system records; the human decides.

**Wave Two: Systems of Workflow (2010s–2020s).** The defining challenge was automating the movement of work between people and systems. Zapier, Workato, and MuleSoft automated data movement. Salesforce added workflow rules and approval processes. ServiceNow built approval chains for IT and HR. The defining concept was the workflow: a sequence of steps, with conditional logic, triggered by events, routed to the right person or system.

Systems of workflow reduced the friction of acting on information. But they still required humans to define the rules, and the rules were brittle. If the data did not match the expected pattern, the workflow stalled. Exceptions went to inboxes. Edge cases were handled manually. The system routed; the human still judged.

**Wave Three: Systems of Intelligence (2024–2030s).** The defining challenge is not capturing information or routing work — it is reasoning about information and taking verified action. The AI-native enterprise platform does not just store a customer's health score; it detects that the score is declining, diagnoses the most likely root cause from a synthesis of support tickets, product usage, and NPS, recommends a specific intervention, drafts the outreach, routes it for CSM approval, executes it upon approval, and then verifies whether the intervention produced the intended outcome.

This is not automation. This is reasoning with accountability.

---

## Why Now

Three things converged in 2023–2025 that made this wave viable at enterprise scale.

**First, foundation model capability crossed the threshold for complex reasoning.** The gap between a frontier model's ability to synthesise multi-source information and a human expert's ability to do the same narrowed enough that AI-assisted judgment became commercially viable in high-value enterprise workflows. Not perfect — but good enough to be genuinely useful, especially when paired with structured human review.

**Second, retrieval-augmented generation made enterprise knowledge accessible.** The combination of embedding models, vector databases, and RAG pipelines means that an AI system can now be grounded in a company's actual operational data: documentation, tickets, customer records, transcripts, policies. The model no longer has to hallucinate context it does not have.

**Third, the cost of deploying AI workflows dropped by an order of magnitude.** The economics of running 10,000 AI-assisted decisions per day became viable for mid-market SaaS companies, not just hyperscalers. This opened the door for AI-native workflow design to move from a competitive differentiator to a table stake.

---

## What the Next-Generation Enterprise Platform Looks Like

The AI-native enterprise platform has five characteristics that distinguish it from a system of workflow with an AI feature bolted on.

**1. The AI layer is not an add-on — it is the operating layer.** The platform does not have an AI assistant that helps users draft things. The AI is embedded in every decision point: it synthesises the data before the user sees it, flags the anomalies before they become incidents, proposes the action before the user has to formulate it.

**2. Human oversight is designed, not retrofitted.** The platform has a well-defined autonomy model: certain actions execute automatically; certain actions execute after system notification; certain actions require explicit human approval; certain actions are always human-initiated. The boundaries are visible, auditable, and configurable by role and context.

**3. The system compounds.** Every human approval or override teaches the system something. Every verified outcome adds a data point to the calibration model. The more the organisation uses the platform, the better the platform gets at predicting what each user or team will approve. This is the data flywheel that creates compounding value and switching cost.

**4. The audit trail is the product.** In regulated enterprise contexts, it is not enough to take the right action — you must be able to prove that the right action was taken, by whom, on what basis, and with what result. The AI-native platform treats the audit log not as a compliance requirement but as a first-class product feature.

**5. The evaluation system is internal.** The organisation does not just monitor model performance through external benchmarks; it maintains its own golden datasets, its own LLM-as-judge rubrics calibrated to its domain, and its own online metrics that connect AI behaviour to business outcomes.

---

## What Product Leadership Means in This Context

Leading product on an AI-native enterprise platform requires a different skill set from traditional B2B SaaS PM. Not a replacement — an extension.

The traditional PM skills still matter: customer empathy, prioritisation, roadmap discipline, cross-functional execution, GTM collaboration. But they are necessary, not sufficient.

The additional skills required are: evaluation methodology (how to measure whether an AI feature is actually working), failure mode analysis (how to anticipate and mitigate the ways an AI system can fail at enterprise scale), trust architecture (how to design the human-in-the-loop layer so it creates confidence rather than overhead), RAG pipeline design (how to ground the model in the right data at the right time), and AI unit economics (how to model the cost, margin, and break-even of deploying AI at scale).

This portfolio is a demonstration of those skills in practice. Every artifact — the agent blueprints, the eval frameworks, the HITL governance docs, the RAG architecture — is designed to show what AI-native product thinking looks like when it is rigorous, not superficial.

The enterprise AI market is in its first innings. The companies that win will be led by PMs who understand the full stack: the workflow, the model, the retrieval system, the trust layer, the evaluation system, and the unit economics. That is the work this portfolio represents.

---

*See also: [From Systems of Record to Systems of Intelligence](/strategy-memos/from-systems-of-record-to-systems-of-intelligence.md) and [How AI Agents Change SaaS PM](/strategy-memos/how-ai-agents-change-saas-pm.md)*
