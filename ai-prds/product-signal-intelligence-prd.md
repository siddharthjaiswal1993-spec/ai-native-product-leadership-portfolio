# PRD: Product Signal Intelligence (ProductPilot OS)

**Version:** 1.0  
**Status:** Draft for Review  
**Author:** Siddharth Jaiswal  
**Date:** 2026-06  
*Note: All data, personas, and scenario examples are synthetic. This is a portfolio artifact demonstrating AI product requirements methodology.*

---

## Problem

Product managers in B2B SaaS spend 40–60% of their PRD preparation time gathering and synthesising information from disconnected tools — Slack, Jira, CRM, support platforms, and customer interview recordings. This results in PRDs that are under-researched or miss the highest-priority customer signals. It also means that the quality of a PRD is heavily influenced by whoever the PM spoke to most recently, rather than by a systematic view of all available signals.

The secondary problem is institutional memory decay. When a PM leaves, the reasoning behind prioritisation decisions leaves with them. Signals that informed key choices are not persisted in a retrievable, citable form.

---

## Target Users

**Primary:**
- Product managers at B2B SaaS companies with 3–15 PMs, $5M–$150M ARR. Mid-to-senior experience level. Technically literate, not engineers. Already using Jira, Slack, and at least one CRM or support platform.

**Secondary:**
- Product leaders (Directors, VPs) who want visibility into the evidence base behind prioritisation decisions and consistency in PRD quality across the team.
- Customer success managers who want customer requests and pain points to flow into the product process without manual re-entry.

---

## Jobs to Be Done

1. **PM — PRD brief:** "Give me everything the company knows about this feature area — customer requests, support tickets, engineering constraints, competitive mentions — in five minutes, cited."
2. **PM — Decision support:** "Before I decide whether to prioritise this feature, show me the evidence that this is actually the most-requested thing from our target segment."
3. **Product Leader — Quality assurance:** "Show me the evidence base behind this PRD so I can assess whether the prioritisation is well-grounded."
4. **CSM — Signal input:** "I have customer feedback that should inform the product roadmap. Help me get it into a form the PM team can act on."
5. **PM — Institutional memory:** "What was the reasoning behind the decision not to build this feature last year?"

---

## Scope

This PRD covers:
- Multi-source signal ingestion: Slack, Jira, CRM (Salesforce or HubSpot), Support (Zendesk or Intercom), Call recordings (Gong or Chorus)
- Semantic chunking, embedding, and indexed storage with product area taxonomy
- Eight-agent PRD brief generation pipeline
- PM review interface with section-level approve / edit / reject and inline citations
- Override logging and feedback loop to eval dataset
- Institutional memory retrieval: natural language Q&A over historical signals

---

## Non-Goals

This PRD does not cover:
- Automated roadmap prioritisation or scoring (PM makes prioritisation decisions; system provides evidence)
- Generating the final PRD (system generates a draft that requires PM review and editing)
- Real-time customer communication or CRM updates
- Financial modeling or revenue impact estimation
- Competitor product analysis beyond mentions in ingested internal content
- Integration with design tools (Figma, etc.) — future scope
- Customer-facing use of any generated content (all outputs are internal)

---

## User Stories

1. **As a PM,** I want to submit a natural language PRD brief request (e.g., "Summarise the case for building SSO support") and receive a structured brief with cited evidence in under 5 minutes, so that I can start PRD writing with complete context rather than spending 3 hours gathering it.

2. **As a PM,** I want every factual claim in the generated brief to include a link to the source (specific Slack message, Jira ticket, CRM note, or transcript segment), so that I can verify claims before including them in my PRD and present them to stakeholders with confidence.

3. **As a PM,** I want to review the brief section by section (problem statement, evidence, customer requests, technical constraints, competitive mentions), approve or edit each section, and have my edits logged, so that I maintain full authorship of the final PRD.

4. **As a PM,** I want to ask natural language questions about historical decisions (e.g., "Why did we deprioritise SSO in Q2 last year?") and get a cited answer from indexed signals, so that I do not repeat decisions that have already been made and can build on prior reasoning.

5. **As a product leader,** I want to see which sections of a PRD brief were edited or overridden by the PM, so that I can assess the quality of AI-generated content and identify areas for improvement.

6. **As a CSM,** I want to submit customer feedback through a structured form that tags it with account tier, product area, and feature request, so that it is immediately indexed and visible in the next PM brief for that product area.

7. **As a PM,** I want the brief to rank customer requests by a combination of frequency, customer tier, and recency, so that I can see which requests represent the highest-weighted customer need, not just the loudest voice.

8. **As a product leader,** I want to see a team-level view of brief generation usage, override rates by section, and avg time from brief generation to PRD completion, so that I can assess adoption and identify where the tool is or is not adding value.

---

## Functional Requirements

### Signal Ingestion
- Connectors for: Slack (selected channels by configuration), Jira (selected projects), Salesforce or HubSpot (accounts, contacts, notes, opportunities), Zendesk or Intercom (tickets), Gong or Chorus (call transcripts via webhook)
- Ingestion frequency: Slack (real-time via event webhook), Jira (daily), CRM (daily), Support (real-time), Transcripts (post-call webhook)
- Parsing: Extract text, metadata (source, date, author, customer/account reference), and signal type (customer request, bug report, competitive mention, feature idea, etc.)
- Chunking: Semantic chunking with 512-token target chunk size, 50-token overlap. Split on topic boundaries where detectable (for long documents/transcripts).
- Embedding: Embed each chunk using a sentence embedding model. Store in vector database with metadata index.
- Product area taxonomy: Maintained by PM team in a configuration interface. Used to tag all ingested chunks. Auto-tagging suggestions presented to PM team admin for confirmation.

### PRD Brief Generation Pipeline
- PM enters: PRD topic (natural language), target customer segment (optional), priority tier (optional)
- System routes to eight-agent pipeline:
  - Agent 1 (Signal Retrieval): Top-20 chunks per signal source for the topic, retrieved by hybrid search
  - Agent 2 (Customer Aggregation): Cluster and rank customer requests by frequency × tier weight × recency decay
  - Agent 3 (Technical Constraint): Surface related Jira issues, engineering comments, known technical debt
  - Agent 4 (Competitive Signal): Extract competitive mentions from transcripts, CRM notes, support tickets
  - Agent 5 (Synthesis): Draft problem statement, evidence summary, user impact, technical considerations
  - Agent 6 (Citation): Validate that every claim in the synthesis maps to a retrieved chunk. Flag uncited claims.
  - Agent 7 (PRD Draft): Expand synthesis into PRD sections: problem, users, JTBD, functional requirements, non-goals, success metrics
  - Agent 8 (Formatting): Format for PM's configured destination (Notion, Confluence, or Jira)
- Pipeline runtime target: <5 minutes end-to-end

### PM Review Interface
- Section-by-section display with inline citation links
- Per-section controls: Approve (green), Edit (open text), Reject (red with reason)
- Citation viewer: clicking a citation opens the source document/message at the relevant passage
- Override logging: all edits captured as {section_id, original_text, edited_text, timestamp}
- No section can be published without PM approval action (even if the PM approves without editing)
- "Publish to [Destination]" button only enabled after all sections have been reviewed

### Customer Request Ranking
- Ranking formula (configurable): frequency × tier_weight × recency_score
  - tier_weight: Enterprise = 3, Mid-market = 2, SMB = 1 (configurable by PM admin)
  - recency_score: 1.0 for last 30 days, 0.7 for last 90 days, 0.4 for last 180 days, 0.2 for older
- Ranked list displayed with: request text, customer count, representative customer examples with tier, source citations
- PM can override ranking manually with a reason logged

### Institutional Memory Q&A
- PM types a natural language question ("Why did we deprioritise [feature] in Q2?")
- System retrieves from full indexed corpus (not scoped to a single topic)
- Response includes: synthesised answer + citations (Slack thread, Jira comment, meeting note, etc.)
- No action is taken based on memory queries — this is retrieval only

---

## AI Capabilities

| Capability | Method | Model Class |
|---|---|---|
| Signal ingestion parsing | Structured extraction from unstructured text | Small LLM or regex pipeline |
| Semantic chunking | Sentence-window chunking with topic boundary detection | Embedding + classification |
| Signal retrieval | Hybrid BM25 + dense retrieval with metadata filtering | Embedding model + BM25 |
| Customer request clustering | Semantic clustering + frequency aggregation | Embedding model + clustering |
| Brief synthesis | Multi-source synthesis with citation enforcement | Frontier LLM (large) |
| Citation validation | Claim-to-source mapping validation | Frontier LLM (medium) |
| PRD section drafting | Structured generation from synthesised brief | Frontier LLM (large) |
| Memory Q&A | RAG over full corpus | Frontier LLM (medium) |

---

## Model Choice Considerations

**Brief synthesis and PRD drafting:** Highest quality requirement. Use a frontier model. The synthesis step determines the quality of everything downstream. Cost is justified — a brief generation pipeline that runs 10× per week per PM team at $0.50–$1.00 per run is a very different economics conversation than a per-token endpoint.

**Citation validation (Agent 6):** Can use a smaller model with a tightly defined task: "Given this claim and this list of source chunks, does any chunk support this claim? Return yes/no + source ID." This is a classification task, not a generation task, and can run on a smaller, faster, cheaper model.

**Signal retrieval:** Not an LLM task. Embedding model + BM25 index + reranker. The retrieval architecture matters more than the generation model for the quality of the brief.

**Semantic chunking:** Consider sentence-window chunking (each chunk is a sentence window with surrounding context retained) over fixed-token chunking for conversational content like Slack threads and transcripts. Fixed-token chunking is acceptable for structured Jira tickets and CRM notes.

---

## Prompting, RAG, and Tool-Use Design

### Signal Retrieval Architecture
- **Indexing:** Dense embeddings (all-mpnet-base-v2 or equivalent) + BM25 inverted index in parallel
- **Query expansion:** For each PM brief request, generate 3 query variants (rephrasing the PM's natural language request) to improve retrieval recall
- **Retrieval:** For each query variant, retrieve top-20 from dense + top-20 from BM25, deduplicate, merge, run cross-encoder reranker on merged set, take top-20
- **Metadata filtering:** Always filter by product_area tag before retrieval (reduces noise, speeds retrieval)
- **Context assembly:** Order retrieved chunks by: (1) recency, (2) tier weight, (3) reranker score

### Synthesis Prompt Design
```
System: You are a product intelligence assistant helping a product manager prepare a PRD brief. 
Your task is to synthesise the provided evidence into a structured opportunity brief. 

Rules:
1. Every factual claim you make must be grounded in a provided source document. 
2. Cite each claim with the format [SOURCE_ID].
3. If you cannot find evidence for a section, write "INSUFFICIENT EVIDENCE: [describe what is missing]"
4. Do not generate claims about customer needs, competitive position, or technical constraints 
   that are not supported by the provided context.
5. The output must be in the required JSON schema.
```

### Citation Enforcement (Agent 6)
- After synthesis, run citation validation: for each claim with a citation, verify the source chunk contains information that supports the claim
- Validation prompt: "Does the following source text support this claim? Return: supported / not_supported / partially_supported + brief explanation"
- Claims returned as "not_supported" are flagged and the synthesis agent is re-prompted to either (a) remove the claim or (b) find an alternative source

---

## Eval Plan

### Offline Evals

**Retrieval evaluation (weekly, automated, golden dataset of 50 signal-query pairs):**
- Recall@10: % of relevant chunks in the top-10 retrieved results — target >80%
- Precision@5: % of the top-5 retrieved results that are relevant — target >70%
- MRR (Mean Reciprocal Rank) for known-relevant chunks — target >0.7

**Synthesis quality evaluation (monthly, human-reviewed sample of 20 briefs):**
- Citation accuracy: % of cited claims that correctly reference the source — target >95%
- Claim accuracy: % of factual claims verified against the source — target >92%
- Hallucination rate: % of claims with no supporting source — target <3%
- Section completeness: % of briefs with all required sections non-empty — target >95%

**PRD draft quality evaluation (monthly, human PM review of 10 drafts):**
- Functional requirements completeness: % that include at least 5 specific, testable requirements — target >90%
- Success metric specificity: % that include at least 2 quantified success metrics — target >85%
- PM override rate: % of PRD sections substantially edited by PM — tracked as quality signal

### Online Metrics

| Metric | Target |
|---|---|
| Pipeline runtime | <5 minutes p95 |
| PM activation rate | >70% of invited PMs run ≥1 brief in first 14 days |
| Citation accuracy (sampled, human review) | >95% |
| PM override rate per section | Tracked; target declining trend over 6 months |
| Time-to-PRD-first-draft | 80% reduction vs. PM-reported baseline |
| Retrieval recall@10 (automated, weekly) | >80% |

---

## Human-in-the-Loop Design

| Action | Autonomy Level | Rationale |
|---|---|---|
| Signal ingestion and indexing | Automatic | No human action needed; source documents are real |
| Product area auto-tagging | Suggestion requires PM admin confirmation | Taxonomy accuracy is critical; human confirmation ensures tag quality |
| Brief generation | Automatic; PM review required before publish | PM must validate before any brief influences a roadmap decision |
| PRD section approval | Required per section | PM authorship and accuracy responsibility |
| Institutional memory Q&A | Automatic response; no action taken | Read-only; no downstream consequences |
| Override logging | Automatic | Transparent data collection for eval improvement |

---

## Guardrails

- **No unreviewed content published:** The system cannot push any brief or PRD section to Notion, Confluence, or Jira without PM approval on every section. Hard system constraint.
- **Citation enforcement:** Synthesis is rejected and re-queued if any claim fails citation validation. The PM is never shown an uncited factual claim without a warning flag.
- **Customer tier data access:** Customer-specific data (account name, deal size, contact details) is only visible to team members with CRM access permission. The brief displays customer signals with tier label (e.g., "Enterprise customer in financial services") for users without CRM access; full account names visible to users with CRM permission.
- **Competitive mention handling:** Competitive mentions in the brief are marked as "[INTERNAL — DO NOT DISTRIBUTE]" and the brief publish destination cannot be a publicly accessible link.
- **Transcript PII:** Contact names in transcript chunks are redacted at ingestion (replaced with [CONTACT]) unless the PM has explicit permission to access contact-attributed quotes.

---

## Privacy and Security

- All signal data is stored within the customer's own cloud environment (VPC-isolated deployment option for enterprise tier)
- No ingested content is used for model training by the platform vendor without explicit written consent
- CRM data ingestion requires the PM admin to authenticate with the CRM using their own credentials; the platform stores an API token with read-only scope
- Transcript data: customers must confirm they have appropriate notice and consent from call participants before enabling transcript ingestion; a consent confirmation checkbox is required during setup
- Data retention: ingested chunks retained for 24 months by default; configurable by PM admin
- Access logs: all retrieval queries and generated briefs are logged for 90 days for security review

---

## Rollout Plan

**Phase 1 — Private Beta (Weeks 1–6):**
- 3 PM teams, 2 signal sources each (Slack + Jira)
- Brief generation only (Agents 1, 2, 5, 6 only — no PRD draft, no competitive signal agent)
- Manual citation review by product team
- Weekly PM feedback sessions

**Phase 2 — Expanded Beta (Weeks 7–14):**
- 10 PM teams, full 5-source ingestion
- All 8 agents active
- PM review interface with section-level approve/edit
- Override logging live

**Phase 3 — GA (Weeks 15–24):**
- Self-serve onboarding for PM teams
- Full integration catalog (Slack, Jira, Salesforce, HubSpot, Zendesk, Intercom, Gong, Chorus)
- Institutional memory Q&A available
- Product leader dashboard live

**Success gate for Phase 2:** Pipeline runtime <5 minutes p95, citation accuracy >90%, at least 2 PM teams reporting >50% reduction in brief preparation time.

---

## Success Metrics

**6-month targets (post GA):**
- PM brief preparation time: 80% reduction (self-reported, monthly survey)
- PRD citation coverage: >80% of PRD factual claims traceable to an indexed source
- Product leader satisfaction with PRD quality: 4.0+/5.0 (monthly survey)
- Monthly active PM teams: 80% of onboarded teams
- Zero incidents of published PRDs containing hallucinated customer data (measured via PM escalation log)

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Retrieval misses high-priority signals | High | High | Retrieval recall monitoring; PM feedback on missed signals |
| Hallucinated customer requests in brief | Medium | High | Citation enforcement; monthly human eval |
| PM over-trust reducing review quality | Medium | Medium | Monitor override rate; UI friction on bulk-approve |
| CRM integration auth complexity | High | Medium | Simple read-only OAuth; detailed setup guide |
| Taxonomy maintenance burden | Medium | Medium | Lightweight PM admin UI; auto-tagging suggestions reduce burden |

---

## Open Questions

1. Should the customer request ranking formula be configurable per PM team or fixed at the platform level? (Risk: misconfiguration; Benefit: flexibility for different team prioritisation models)
2. How should the system handle a brief request for a feature area with insufficient indexed signals — return a partial brief with a coverage warning, or reject and ask the PM to broaden the query?
3. Should the PM review interface include a side-by-side comparison with a previous brief for the same product area, to show how signals have changed over time?
4. How do we handle a PM who imports their own research (a competitor analysis document, a user research report) and wants it included in the brief? (Ad hoc document ingestion — not in scope for V1?)
5. What is the right model for the product area taxonomy — should it auto-expand as new terms are detected in ingested content, or should it be strictly PM-admin maintained?

---

*See also: [ProductPilot OS Case Study](/case-studies/productpilot-product-intelligence.md) · [Product Intelligence Agent Blueprint](/agent-workflow-blueprints/product-intelligence-agent.md) · [RAG Evaluation Framework](/ai-evaluation-frameworks/rag-evaluation.md)*
