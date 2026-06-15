# ProductPilot OS: Product Signal Intelligence

**One-line summary:** An eight-agent system that connects Slack, Jira, CRM, support tickets, and customer interview transcripts into a cited PRD engine — so that product managers spend their time deciding, not synthesising.

*All example data, signal content, and output examples in this case study are synthetic. ProductPilot OS is a portfolio prototype, not a shipping product.*

---

## Context and Background

Product managers in B2B SaaS spend a disproportionate amount of their time on information gathering. A PM preparing a PRD might spend two to three hours pulling data: checking what the top five customers have requested in the last 90 days, reviewing the latest round of support tickets for a feature area, catching up on the last three customer calls, and checking what engineering flagged as technical debt adjacent to the proposed work. Then they spend another hour or two synthesising that into a document.

The information exists. It is already in the company's tools. The problem is that it is distributed across four to eight systems, none of which are connected, and the PM has to manually pull and synthesise it on every cycle.

ProductPilot OS is a product intelligence platform that automates this synthesis, grounding every output in specific, cited sources — so that PMs can focus on the judgment layer (prioritisation, tradeoffs, stakeholder alignment) rather than the information gathering layer.

---

## Problem Statement

Product managers in growing B2B SaaS companies spend 40–60% of their PRD preparation time on information gathering and manual synthesis from disconnected tools. This results in PRDs that are under-researched, PRDs that miss the most important customer signals, and a backlog prioritisation process that is more driven by whoever spoke to the PM last week than by a systematic view of customer need.

The secondary problem is institutional memory. When a PM leaves, the context about why decisions were made leaves with them. The signals that informed prioritisation are not persisted in a retrievable form.

---

## Users and Personas

**The Product Manager (Primary):** Mid-to-senior PM in a B2B SaaS company with 50–500 employees. Technically literate but not an engineer. Has accounts in Jira, Slack, HubSpot or Salesforce, Zendesk or Intercom, and possibly Gong or Chorus. Wants well-researched PRDs without the research burden. Key pain: "I know the answers are in our systems somewhere, I just don't have time to find them all."

**The Product Leader (Approver / Power User):** Director or VP of Product. Wants visibility into the signal basis for prioritisation decisions. Cares about consistency across the PM team. Uses the platform to review the evidence base behind proposals before approving roadmap items.

**The Customer Success Manager (Indirect Contributor):** Closes the loop between customer signals and product team. Wants customer requests and pain points to be visible in the product process without having to write separate Jira tickets or repeat feedback in every product review.

---

## Existing Workflow Pain

Current state for a PM preparing a PRD:
1. Search Slack for relevant threads in 3–4 channels (20 min)
2. Pull open Jira tickets in the relevant epics (15 min)
3. Open CRM and manually read through account notes for top customers (30 min)
4. Open Zendesk and search for tickets in the feature area (15 min)
5. Find and re-read Gong transcripts from the last 3 customer calls (45 min)
6. Open a blank doc and start manually synthesising (60 min)
7. Write the PRD (60–90 min)

Total pre-writing time: ~3 hours. Total writing time: ~90 minutes. A significant proportion of the PM's time is spent on tasks that do not require PM judgment.

---

## Product Thesis

If you can ingest all the relevant signals into a shared knowledge graph, chunk and index them by product area and customer segment, and run an eight-agent synthesis pipeline on every PRD request, you can deliver a first draft PRD grounded in specific cited sources in under five minutes — and reduce the PM's pre-writing information work from three hours to ten minutes of review and judgment.

The product is not a PRD generator. It is a cited synthesis engine. The PM still makes every decision. The product ensures those decisions are made with complete information.

---

## Solution Overview

**Signal ingestion:** Connectors pull from Slack, Jira, HubSpot/Salesforce, Zendesk/Intercom, and Gong/Chorus transcripts on a configurable schedule. Content is parsed, chunked by semantic unit, embedded, and indexed in a vector store with metadata: source system, date, customer segment, product area tag.

**PRD brief request:** The PM enters a natural language request ("Build me a brief for the SSO integration feature") plus optional context (target customer segment, priority tier). The system routes to the eight-agent pipeline.

**Eight-agent pipeline:**
1. **Signal Retrieval Agent** — Pulls relevant chunks from all indexed sources for the given product area
2. **Customer Request Aggregation Agent** — Groups and ranks customer requests by frequency, customer tier, and recency
3. **Technical Constraint Agent** — Surfaces related Jira tickets, engineering comments, and known technical debt from the indexed corpus
4. **Competitive Signal Agent** — Surfaces any competitive mentions in transcripts, support tickets, or CRM notes
5. **Synthesis Agent** — Drafts a structured opportunity brief: problem statement, evidence summary, user impact, technical considerations
6. **Citation Agent** — Maps every claim in the brief to a specific source chunk with a link
7. **PRD Draft Agent** — Expands the opportunity brief into a structured PRD template (problem, users, JTBD, functional requirements, non-goals, success metrics)
8. **Review Formatting Agent** — Formats the output for the PM's preferred destination (Notion, Confluence, or Jira)

**PM review:** The PM receives the drafted PRD brief with inline citations. They can reject, edit, or promote each section. Overrides are logged and fed back into the system's evaluation dataset.

---

## AI-Native Layer

- **RAG pipeline:** Hybrid retrieval (dense semantic + sparse BM25 keyword) with metadata filtering by product area, date range, and customer tier. Reranking by relevance score before context assembly.
- **Knowledge graph overlay:** Product area taxonomy maintained by the PM team, used to tag all ingested chunks. Enables retrieval by product area without relying solely on semantic similarity.
- **Citation enforcement:** The synthesis prompt architecture requires each claim to cite a specific source chunk. The Citation Agent validates that all claims in the output have a corresponding source, and flags uncited claims for PM review.
- **Override learning:** Every PM edit to a generated PRD section is captured as a preference pair. These pairs are used to calibrate the rubrics used in the LLM-as-judge eval and, over time, to fine-tune the generation prompts for each team's PRD style.

---

## Workflow and Agent Architecture

```
[Ingestion Layer]
Slack  ─┐
Jira   ─┤
CRM    ─┼─→ Parser → Chunker → Embedder → Vector Store
Support─┤                                    + Metadata Index
Gong   ─┘

        ↓ (PM enters PRD request)

[Routing Layer]
PRD Request → Product Area Classifier → Context Window Assembler

        ↓

[Eight-Agent Pipeline]
1. Signal Retrieval Agent    → Top-N relevant chunks per source
2. Customer Aggregation Agent → Ranked request list with frequency
3. Technical Constraint Agent → Related Jira + tech debt signals
4. Competitive Signal Agent   → Competitive mentions in corpus
5. Synthesis Agent            → Opportunity brief draft
6. Citation Agent             → Validates + maps all citations
7. PRD Draft Agent            → Full PRD structure from brief
8. Review Formatting Agent    → Formats for Notion/Confluence/Jira

        ↓

[PM Review Interface]
Inline citation viewer
Section-level approve / edit / reject
Override logging → Eval dataset

        ↓ (approved)

[Destination]
Published to Notion / Confluence / Jira Epic
```

---

## Key Product Decisions

**1. Citations are non-negotiable, not optional.**
Early prototypes let PMs toggle citations on or off. Usage data showed that PMs who turned citations off were more likely to question and discard the output, not less. The reason: without citations, it is impossible to verify whether the synthesis is hallucinated or real. With citations, PMs could quickly spot-check and build confidence. Citations became a mandatory output element.

**2. Eight agents vs. one large prompt.**
The initial architecture was a single large synthesis prompt with all context passed at once. This produced outputs that were difficult to debug when they were wrong — a claim about competitive signals might be hallucinated because the model did not retrieve any real competitive mentions, but the failure was invisible. Breaking into specialised agents with defined input/output schemas made failures visible and correctable at each step.

**3. Product area taxonomy maintained by humans, not inferred by AI.**
The temptation was to let the AI auto-tag all incoming signals with product areas. Testing showed that auto-tagging was inconsistent across ambiguous signals (a support ticket about performance could belong to three different product areas). The taxonomy is human-maintained by the PM team, and auto-tagging suggestions are presented for human confirmation before indexing.

**4. Override learning requires explicit PM consent per session.**
PMs were initially uncomfortable knowing that their edits were being used to train future outputs. This was addressed by making override logging opt-in per session with a clear explanation: "Your edits improve future drafts for your team. Disable in settings." Opt-in rate was high (>85%) once the value proposition was clear.

---

## Metrics and Success Criteria

**Primary:**
- PM time-to-first-PRD-draft (target: <10 minutes review from request to approved draft vs. ~3 hours baseline)
- PM-reported citation accuracy (monthly sample evaluation: target >90% of citations correctly linked)
- PRD stakeholder satisfaction score (engineering, design, CSM rating of PRD completeness)

**Secondary:**
- Signal ingestion coverage (what % of company's product signals are indexed)
- Citation volume per PRD (as a proxy for evidence depth)
- Override rate per agent (which agents generate the most corrections — signals for improvement)

**Leading indicators:**
- Pipeline run time (<5 minutes end-to-end target)
- Retrieval recall@5 for known-signal queries (tested monthly against golden dataset)
- Agent-level error rate

---

## Risks and Guardrails

**Hallucinated customer requests:** The most dangerous failure mode — the system asserts that "Customer X requested SSO in Q3" when no such request exists. Mitigated by: citation enforcement, retrieval-first architecture (claims require a retrieved source), monthly human eval on a sample of customer request claims.

**Signal recency gaps:** If ingestion lags, the brief may miss recent high-priority signals. Mitigated by: data freshness timestamp displayed in the brief header, freshness alert if any source is >48 hours stale.

**PM over-trust:** If PMs publish AI-generated PRDs without meaningful review, quality will decline over time as the system's errors compound. Mitigated by: mandatory section-level review before publication, no auto-publish capability.

**PII in transcripts:** Customer interview transcripts may contain sensitive information. Mitigated by: PII redaction layer applied at ingestion, customer name substitution in stored chunks, access control by team.

---

## GTM and Adoption Considerations

ProductPilot OS targets B2B SaaS product teams with 3–15 PMs who are already using Jira, Slack, and either a CRM or support platform. The integration surface area is the primary adoption driver and the primary deployment risk.

Land and expand: start with one PM team, one product area, three signal sources. Expand integrations and PM coverage after the core signal-to-brief pipeline is validated.

Pricing model (illustrative): Per-PM per-month seat, with a usage component based on pipeline runs above a threshold. Enterprise tier with on-premise deployment and data residency controls.

---

## What I Would Build First (MVP Wedge)

Slack + Jira + one support tool (Zendesk or Intercom). Three agents only: Signal Retrieval, Customer Aggregation, and PRD Draft. Citations from retrieved chunks but no separate Citation Agent validation. PM review before any output is published. Single destination: Notion.

This proves the core value prop — AI-generated, cited PRD first drafts — with the minimum integration surface area. Remaining agents and integrations are later releases.

---

## What I Would Measure (Instrumentation Plan)

Day 1: Pipeline run time, PM first-session activation (did they review a generated brief?)

Week 2: Section-level override rate (which sections get edited most?)

Month 1: PM self-reported time savings, PRD completion rate (did PMs who used the brief publish a PRD?)

Month 2: Citation accuracy human eval (random sample of 20 briefs, manual verification of 5 citations per brief)

Month 3: Downstream stakeholder satisfaction — does engineering find PRDs generated with ProductPilot more complete than those generated without?

---

## Lessons and Tradeoffs

The central lesson is that the quality of a synthesis system is bounded by the quality of its retrieval, not the quality of its generation. You can have a very capable frontier model generating outputs, but if the retrieved context is missing the two most relevant customer requests, the output will be confidently wrong about customer priorities.

Invest in retrieval quality first. Ingestion coverage, chunking strategy, metadata tagging, and retrieval recall are the real product work. Generation quality is table stakes once retrieval is right.

The tradeoff between eight agents and a single large prompt is real. Eight agents are more expensive to run (more tokens, more latency) but dramatically more debuggable. For a product that enterprise customers are going to trust with their roadmap decisions, debuggability is more valuable than speed.

---

## Related Resources

- [Product Signal Intelligence PRD](/ai-prds/product-signal-intelligence-prd.md)
- [Product Intelligence Agent Blueprint](/agent-workflow-blueprints/product-intelligence-agent.md)
- [RAG System Overview](/rag-workflow-documentation/rag-system-overview.md)
- [ProductPilot OS GitHub Project](https://github.com/siddharthjaiswal1993-spec/productpilot-os)
