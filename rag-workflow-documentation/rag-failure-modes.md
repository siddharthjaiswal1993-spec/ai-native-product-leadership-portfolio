# RAG Failure Modes: The 8 Most Common Failures, How to Detect Them, and How to Fix Them

---

## Why Failure Mode Analysis Is a PM Responsibility

Most PMs treat RAG failures as engineering bugs. They are not — they are product quality issues that manifest differently depending on the domain, the corpus, and the user's task. Understanding the failure modes is the first step to designing the mitigations into the product architecture, not retrofitting them after a user escalation.

Each failure mode below includes: what it looks like to the user, what causes it technically, how to detect it in production, and what to do about it.

---

## Failure Mode 1: Missing Chunk (Retrieval Miss)

**What it looks like:** The AI system says "I don't have enough information about this" when the information clearly exists in the company's data. Or the system generates a response that is missing a key relevant fact.

**Technical cause:** The relevant chunk is in the corpus but was not retrieved in the top-K results. This can happen because:
- The query phrasing is too different from the chunk phrasing (semantic gap)
- The chunk is buried below K in the ranking
- The metadata filter excluded the relevant chunk (too-narrow filter)
- The chunk was indexed with a poor embedding (e.g., truncated during embedding due to context length mismatch)

**Detection:** Monitor recall@K on a golden query set (queries with known relevant chunks). If recall@K drops below threshold, investigate. Also: capture "I don't know" responses and manually review whether the information existed in the corpus.

**Mitigation:**
- Query expansion: generate multiple rephrasings of the query to improve recall
- Increase K (retrieve more candidates before reranking)
- Audit metadata filters for over-narrowing
- Use HyDE to improve retrieval for abstract queries
- Review embedding model context length vs. average chunk size

---

## Failure Mode 2: Wrong Chunk (Retrieval Noise)

**What it looks like:** The AI system answers the question but with information from the wrong account, the wrong time period, or an adjacent topic. Example: asked about Acme Corp's support tickets, the system returns information about Ajax Corp's support tickets.

**Technical cause:**
- Entity resolution failure: the account name is not uniquely identifying in the corpus (multiple accounts with similar names)
- Date filtering not applied correctly
- Semantic confusion between similar concepts (two products with similar names, two customers in the same industry)
- Insufficient metadata scoping in the retrieval query

**Detection:** Monitor precision@5 on the golden eval set. For account-specific queries, test entity specificity (does the retrieval correctly scope to the specified account?).

**Mitigation:**
- Strict metadata filtering by entity_id (not just entity name) at the retrieval layer
- Entity resolution: ensure all account names are resolved to canonical IDs at ingestion time
- Add entity disambiguation logic at query processing: if the query contains an entity name, resolve to entity_id before retrieval

---

## Failure Mode 3: Hallucination Despite Retrieval

**What it looks like:** The AI system cites sources but makes claims that those sources do not actually support. Or the system retrieves relevant context but ignores it and generates from training data instead.

**Technical cause:**
- The system prompt does not sufficiently constrain the model to use only the retrieved context
- The model's training priors are strong enough to override the retrieved context (common for facts that are broadly known and contradicted by specific internal data)
- The context window is too large and the relevant chunks are in positions with low attention weight
- The model is generating plausible-sounding continuations of the context rather than factual claims

**Detection:** Monthly citation accuracy evaluation: for a sample of outputs, verify that each cited claim is actually supported by the cited source. Target: >95% accuracy.

**Mitigation:**
- Explicit system prompt instruction: "Only use the provided context. Do not use your training knowledge. If the context contradicts your training knowledge, trust the context."
- Retrieval-first architecture: ensure the most relevant chunks are positioned at the beginning and end of the context window
- Citation enforcement: require inline citations for all factual claims; validate via Citation Agent
- Reduce hallucination-prone claim types: avoid asking the model to fill in information not present in the context

---

## Failure Mode 4: Citation Fabrication

**What it looks like:** The AI system cites a specific source (e.g., "[CRM-note-2026-04-15]") that either does not exist, or that exists but does not contain the information the model claimed it contains.

**Technical cause:**
- The model generates a citation that looks plausible based on the citation format, without verifying that the cited source supports the claim
- This is a variant of hallucination that is particularly dangerous because it superficially looks like a grounded output

**Detection:** Citation validation step (Citation Agent): for every output citation, verify that the referenced chunk ID exists in the retrieval results AND that the chunk text contains information supporting the cited claim. Log citation validation failures.

**Mitigation:**
- Only allow citation IDs from the retrieved context in the generation prompt — the model is told "you may only cite SOURCE_IDs from the following list: [list]"
- Post-generation citation validation as a mandatory step before output delivery
- Log citation validation failures for human review
- Reject outputs with failed citations (re-generate or return an "insufficient evidence" response)

---

## Failure Mode 5: Stale Index

**What it looks like:** The AI system provides information that was accurate three weeks ago but is no longer current. Example: the system says a customer's renewal date is "Q2 2026" when it was rescheduled to Q1 2026 two weeks ago.

**Technical cause:**
- The ingestion pipeline has not run, has run partially, or the specific source has not been updated
- The source system's API changed and the connector is silently failing
- The ingestion schedule is too infrequent for the use case

**Detection:** Monitor data freshness per source type: track the timestamp of the most recently indexed chunk per source and alert when any source exceeds its freshness SLA (e.g., CRM data should be no more than 24 hours old).

**Mitigation:**
- Display data freshness timestamps in the user-facing output: "Data as of [timestamp]"
- Freshness alerting: surface a warning when any key source exceeds its freshness SLA
- Ingestion monitoring: alert on connector failures (no new chunks indexed from a source in X hours)
- For high-staleness-risk data (renewal dates, contract status), retrieve directly from the source system API at query time, not from the index

---

## Failure Mode 6: Context Window Overflow

**What it looks like:** The AI system appears to ignore relevant information that was retrieved, or produces outputs that are inconsistent with what the user knows is in the corpus. Or: the system errors with a context length exceeded message.

**Technical cause:**
- Too many chunks retrieved, or chunks are too large, exceeding the context window
- When the context exceeds the window, the oldest context is truncated — potentially the most important context
- Models attending preferentially to the beginning and end of the context window ("lost in the middle" problem) may underweight chunks positioned in the middle of a long context

**Detection:** Monitor token count of assembled context per query. Alert when context exceeds 80% of the model's context window. Monitor override rate for outputs where context was truncated.

**Mitigation:**
- Token budget management: cap the number of chunks assembled in the context to stay within a defined token budget
- Reranking: use reranking to ensure only the highest-relevance chunks are included in the context window
- Hierarchical summarisation: for very large retrieved sets, summarise groups of chunks before assembly
- Chunk size reduction: if chunks are consistently large, reduce the target chunk size
- Model selection: use a model with a larger context window for tasks that require many retrieved chunks

---

## Failure Mode 7: Semantic Collision

**What it looks like:** The AI system retrieves chunks about a different entity than the one queried — because two entities have similar descriptions, similar vocabulary, or similar context. Example: retrieval for "the reporting feature" in a product with two different reporting features surfaces chunks about the wrong one.

**Technical cause:**
- Two concepts or entities have similar semantic representations in the embedding space
- No metadata filter distinguishes between them
- The query is underspecified (does not include enough discriminating information)

**Detection:** Manual review of retrieval results for high-frequency ambiguous queries. Precision@5 analysis on ambiguous entity queries specifically.

**Mitigation:**
- Metadata tagging: tag chunks with specific entity identifiers (product_feature_id, account_id) at ingestion time; use these as filters at retrieval time
- Disambiguation at query processing: if the query contains an ambiguous term, prompt the user to disambiguate before retrieval
- Fine-tuned embedding model: for highly domain-specific disambiguation needs, a fine-tuned embedding model may be more precise

---

## Failure Mode 8: Corpus Coverage Gap

**What it looks like:** The AI system consistently fails to answer a category of queries accurately, not because retrieval is failing but because the relevant information was never ingested.

**Technical cause:**
- A data source that contains the relevant information was not connected to the ingestion pipeline
- The relevant information is stored in a format that the parser does not handle (e.g., embedded in a spreadsheet, in a PDF table, in a video recording)
- The relevant information was created after the ingestion pipeline was set up and the new source was not added

**Detection:** Track "I don't have information about this" responses by category. If a pattern emerges ("the system can never answer questions about contract dates"), investigate whether the relevant source is connected and ingesting correctly.

**Mitigation:**
- Corpus coverage audit: regularly review which data sources are connected, which are not, and whether there are important coverage gaps
- Gap escalation: when a "no information" response is surfaced to a user, provide a clear path to report the gap
- Source expansion roadmap: treat corpus coverage as a product roadmap item, not just an engineering task
- Signal-to-source tracing: for each type of query the system is expected to answer, document which source should contain the relevant data and verify it is connected

---

## Failure Mode Summary Table

| Failure Mode | User Experience | Primary Detection | Primary Mitigation |
|---|---|---|---|
| 1. Missing Chunk | "I don't know" when it should know | Recall@K eval | Query expansion, increase K |
| 2. Wrong Chunk | Wrong account / wrong entity | Precision@5 eval, entity query tests | Entity metadata filters |
| 3. Hallucination Despite Retrieval | Confident wrong answer | Citation accuracy eval | Prompt constraints, Citation Agent |
| 4. Citation Fabrication | Fake source cited | Citation ID validation | Citation ID whitelist, post-gen validation |
| 5. Stale Index | Outdated information | Freshness monitoring per source | Freshness alerts, direct-source fallback |
| 6. Context Overflow | Relevant context ignored | Token count monitoring | Token budget, reranking, chunk size |
| 7. Semantic Collision | Wrong entity retrieved | Precision@5 on ambiguous queries | Metadata tagging, disambiguation prompt |
| 8. Coverage Gap | "No information" on a whole category | "No information" response tracking | Corpus coverage audit, source expansion |

---

*See also: [RAG Evaluation Plan](/rag-workflow-documentation/rag-eval-plan.md) · [Chunking, Embeddings, and Retrieval](/rag-workflow-documentation/chunking-embeddings-retrieval.md) · [Reranking and Citation Grounding](/rag-workflow-documentation/reranking-citation-grounding.md)*
