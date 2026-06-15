# Reranking, Citation Injection, and Grounding Strategies

---

## Why Reranking Matters

The first-pass retrieval step (dense + sparse) is optimised for speed. It retrieves a candidate set of chunks that are likely to be relevant. But "likely relevant" is not the same as "most relevant for this specific query and task."

Reranking applies a more compute-intensive relevance model to the candidate set to produce a final ranked list. The quality improvement from reranking over first-pass retrieval alone is consistent and significant — typically 10–20% improvement in MRR and Precision@5 on standard benchmarks.

The tradeoff is latency and cost. Reranking is O(N) compute for N candidates, whereas vector search is sub-linear. For real-time queries, this matters. For batch or asynchronous workflows (brief generation, QBR drafting), the latency cost of reranking is acceptable.

---

## Reranking Approaches

### Cross-Encoder Reranking

**How it works:** A cross-encoder model takes the query and a candidate chunk as a single concatenated input and produces a relevance score. Unlike the bi-encoder embedding approach (where query and chunk are embedded separately), the cross-encoder allows the model to compute attention across both the query and the chunk simultaneously — enabling it to detect subtle relevance signals that bi-encoder similarity misses.

**Example:** Query: "Why is the Acme Corp health score declining?" Candidate chunk: "Acme Corp's export feature usage dropped 35% in the last 14 days per product analytics." A cross-encoder can score this as highly relevant by recognising that export feature usage is a health score component. A bi-encoder might underrank this chunk if the word "health score" is not in the chunk.

**Popular models:** 
- ms-marco-MiniLM-L-6-v2 (fast, English-only, good for general enterprise text)
- Cohere Rerank API (managed, strong quality, configurable context size)
- bge-reranker-large (open-source, strong performance on BEIR benchmarks)

**Latency:** ~50–200ms for 20 candidates using an efficient cross-encoder. Acceptable for most workflows.

---

### ColBERT-Style Late Interaction

**How it works:** ColBERT stores per-token embeddings for each chunk at index time (rather than a single document embedding). At retrieval time, it computes token-level maximum similarity between query tokens and document tokens — enabling more precise relevance detection at a lower latency than cross-encoders.

**Advantages over cross-encoder:** Faster at query time because the document embeddings are pre-computed. Scales better for large candidate sets.

**Disadvantages:** Larger index size (one vector per token per chunk, vs. one vector per chunk for standard bi-encoder). Requires a specialised vector store that supports late-interaction retrieval (e.g., RAGatouille, which wraps ColBERT).

**When to use:** High-volume query workloads where cross-encoder latency is unacceptable; tasks where exact-term matching in a semantic context is important (e.g., specific technical terms in product documentation).

---

### LLM-Based Reranking

**How it works:** Ask an LLM to rate the relevance of each candidate chunk to the query, or to rank a list of candidates. Output: relevance score per chunk, or an ordered ranking.

**Example prompt:**
```
Given this query: "[QUERY]"
Rate the relevance of the following document to this query on a scale of 1-10.
Document: "[CHUNK_TEXT]"
Return only the number.
```

**Advantages:** Can incorporate task-specific relevance criteria that a generic reranker cannot (e.g., "relevance for a pre-visit field consultant brief" is a more nuanced criterion than generic document relevance).

**Disadvantages:** Slow (one LLM inference per candidate × N candidates). Expensive. Not suitable for real-time queries. Best used in offline processing or for tasks where quality far outweighs latency.

**When to use:** Offline batch processing where quality is paramount; generating evaluation datasets; calibrating simpler rerankers.

---

## Citation Injection

Citation injection is the process of mapping claims in the generated output back to specific source documents or chunks, and embedding that mapping in the output in a way that is visible and verifiable.

### Why Citations Are Non-Negotiable in Enterprise AI

Enterprise users — CSMs, PMs, field consultants, finance teams — are professionally accountable for the information they act on. If an AI system tells a CSM that a customer is at churn risk, and the CSM does nothing and the customer churns, the CSM needs to be able to defend why they did or did not act. "The AI said so" is not a defence. "The AI said so based on a 35% usage decline in the last 14 days (source: product analytics, as of [date])" is a defensible basis for a professional judgment.

Citations also enable rapid quality checks. A CSM reviewing an account brief can spot-check two or three citations in 60 seconds. If those citations are accurate, their confidence in the rest of the brief increases significantly. Without citations, the entire brief must be manually verified or accepted on faith.

### Citation Enforcement in Prompts

The system prompt must make citation enforcement explicit and non-optional:

```
For every factual claim you make, you must cite the source using [SOURCE_ID].
SOURCE_ID corresponds to the source identifiers provided in the context.
Do NOT make any factual claim that is not supported by a provided source.
If you cannot find a source for a claim, either omit the claim or write:
"[UNVERIFIED: claim text — no source available in current context]"
```

**What "citation enforcement" means in practice:**
- The generation prompt requires citation markers for all factual claims
- A post-generation validation step (the Citation Agent) checks that every citation marker resolves to a real source chunk
- Claims with no corresponding source are flagged, removed, or labeled as unverified before delivery

### Citation Injection Patterns

**Inline citation (recommended for briefs and summaries):**
> Support ticket volume for Acme Corp increased 40% in the last 30 days [ZD-4521, ZD-4498]. The primary issue category is "Export timeout" [ZD-4521].

**Footnote citation (recommended for long documents like QBRs):**
> Product usage declined 22% in Q4 2025.¹
> ¹ Source: Product analytics dashboard, Acme Corp workspace, pulled 2026-01-05

**Source card (recommended for interactive interfaces):**
> The export timeout issue was first reported on [date]. [View Ticket →]

---

## Grounding Strategies

Grounding is the broader practice of ensuring that model outputs are connected to verifiable facts in the retrieval corpus, not generated from the model's training data. Citation enforcement is one grounding strategy. These are others:

### Retrieval-First Grounding

Every claim in the output must trace to a retrieved chunk. The model is instructed not to make claims that are not in the retrieved context, even if the model "knows" the answer from training. This is the most important grounding discipline for enterprise use cases where the authoritative source is your operational data, not general world knowledge.

**Implementation:** System prompt instruction ("Do not use knowledge from your training; only use the provided context"). Post-generation validation (Citation Agent).

### Confidence Gating

Assign a confidence score to the generated output based on the quality and completeness of the retrieved evidence. Outputs below a confidence threshold are:
- Not delivered (replaced with a "not enough information" message), OR
- Delivered with a visible "low confidence" flag, OR
- Routed to human review

**Confidence signals:**
- Number of citations in the output (more citations = more grounded)
- Reranker scores of cited chunks (higher scores = more relevant evidence)
- Presence of "I'm not sure" or "insufficient information" language in the output (model self-reported uncertainty)

### Negative Space Handling

The model should explicitly state when it cannot find information, rather than generating a plausible-sounding guess. The system prompt should include: "If the provided context does not contain sufficient information to answer the question, say explicitly: 'I do not have sufficient information to answer this. The available context covers [X] but not [Y].'"

Explicit "I don't know" responses are more useful to enterprise users than confident-sounding guesses. They tell the user to look elsewhere rather than acting on incorrect information.

### Source Authority Ranking

Not all sources are equally authoritative. In a context assembly step, you can rank sources by authority and include that authority rank in the chunk metadata. The generation prompt can instruct the model to prefer claims supported by higher-authority sources:

```
When multiple sources make conflicting claims, prefer the source with the highest authority.
Authority ranking (highest to lowest): official_database > recent_transcript > 
jira_comment > slack_message > third_party_document
```

---

## Grounding in Multi-Agent Systems

In multi-agent systems, grounding becomes a coordination problem. Each specialist agent retrieves its own context and generates its own output. The coordinator synthesises the outputs. At the synthesis stage, the coordinator must:

1. Preserve citation provenance from each specialist: "This claim came from the Account Intelligence agent, which cited [CRM-note-2026-04]"
2. Detect conflicts between specialist outputs and surface them rather than synthesising a false consensus
3. Apply grounding rules at the coordinator's synthesis prompt — the coordinator must cite which specialist's output supports each claim in the final synthesis

This is more complex than single-agent citation but essential for the audit trail to remain coherent across a multi-agent pipeline.

---

*See also: [Enterprise RAG Architecture](/rag-workflow-documentation/enterprise-rag-architecture.md) · [RAG Failure Modes](/rag-workflow-documentation/rag-failure-modes.md) · [RAG Evaluation Plan](/rag-workflow-documentation/rag-eval-plan.md)*
