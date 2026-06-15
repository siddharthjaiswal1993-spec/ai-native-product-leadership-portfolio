# Enterprise RAG Architecture

**A full architecture document for production RAG systems in enterprise B2B SaaS**

---

## Architecture Overview

A production enterprise RAG system consists of two primary pipelines:

1. **The Ingestion Pipeline** — Transforms raw data from operational systems into a searchable, indexed corpus
2. **The Retrieval-Generation Pipeline** — Takes a query, retrieves relevant context, and generates a grounded response

These two pipelines run independently. Ingestion is a background process (continuous or scheduled). Retrieval-Generation is a foreground process (on-demand, user-triggered, or agent-triggered).

---

## Ingestion Pipeline

### Stage 1: Data Source Connectors

**Purpose:** Pull raw content from operational systems into the ingestion pipeline.

**Connector types:**
- **Webhook-based (real-time):** Support tickets, CRM events, Gong call completions
- **API polling (scheduled):** LMS completion data, product analytics exports, CRM bulk sync
- **File drop (batch):** Uploaded documents, PDFs, CSV exports
- **Database snapshot (daily):** Structured tables from internal databases

**Key engineering decisions:**
- Each connector should output a standardised raw content object: {source_id, source_type, content_text, metadata, timestamp}
- Connectors should be stateless and idempotent — running the same connector twice should not create duplicate records
- Rate limiting and backoff should be built into each connector, not managed centrally

**PM decision: which sources to connect.** Not all data sources should be connected. Criteria for inclusion: (a) the data is used in decisions that the AI system is expected to support, (b) the data is authoritative (not a copy or derivative of another connected source), (c) access can be controlled at the appropriate granularity.

---

### Stage 2: Parsing and Cleaning

**Purpose:** Extract clean text and structured metadata from raw content in a variety of formats.

**What happens:**
- HTML parsing: strip nav, headers, footers, ads; extract main body text
- PDF parsing: extract text with page markers; handle tables as structured text
- Transcript formatting: convert speaker-labeled transcript to clean dialogue format; extract speaker names
- Structured data (JSON/CSV): convert to human-readable text or extract as key-value pairs
- PII detection and redaction: identify and redact/tokenise PII fields before indexing (email addresses, phone numbers, names in configurable categories)

**Quality checks:**
- Minimum length threshold: chunks below N characters are discarded (likely parsing artifacts)
- Language detection: non-target-language content flagged or handled appropriately
- Duplicate detection: near-duplicate content across sources identified and deduplicated

---

### Stage 3: Chunking

**Purpose:** Divide documents into segments of a size appropriate for retrieval — small enough to be specific, large enough to be coherent.

**Chunking strategies and tradeoffs:**

| Strategy | Description | Best for |
|---|---|---|
| Fixed-size chunking | Split every N tokens with M token overlap | Structured, uniform content (databases, forms) |
| Sentence-window chunking | Use individual sentences as retrieval units; include surrounding window as context | Short, factual content (tickets, CRM notes) |
| Semantic chunking | Split at topic boundaries detected by embedding similarity | Long-form documents with distinct sections |
| Parent-child chunking | Store large parent chunks for context; index small child chunks for retrieval | Documents with hierarchical structure |
| Recursive character splitting | Split at paragraph, then sentence, then word — stop when target size is reached | Mixed-format documents |

**Chunk size considerations:**
- Too small (< 100 tokens): specific but loses context; model cannot understand without surrounding text
- Target range (200–600 tokens): good balance of specificity and context
- Too large (> 800 tokens): high recall but low precision; buries the relevant passage

**Metadata attached to each chunk:**
```json
{
  "chunk_id": "uuid",
  "source_id": "uuid of parent document",
  "source_type": "jira_ticket | crm_note | support_ticket | transcript | doc",
  "entity_tags": ["account:ACC-001", "product_area:reporting", "customer_tier:enterprise"],
  "date": "ISO 8601",
  "author": "anonymised if PII-sensitive",
  "chunk_index": 3,
  "total_chunks_in_document": 8,
  "language": "en"
}
```

---

### Stage 4: Embedding

**Purpose:** Convert each text chunk into a dense vector representation that captures semantic meaning, enabling similarity-based retrieval.

**Embedding model selection criteria:**
- **Dimension size:** 768 or 1536 dimensions are standard; larger = more expressive but more storage and compute
- **Domain adaptation:** General-purpose models (e.g., all-mpnet-base-v2, text-embedding-3-small) perform well for most enterprise B2B text; domain-specific fine-tuning may improve performance for highly specialised vocabularies
- **Context length:** Embedding model must support the target chunk size; models with 512 token context limit cannot meaningfully embed 800-token chunks
- **Cost:** Embedding is done at ingestion time; the per-token cost multiplied by total corpus size determines the ingestion cost

**BM25 index (parallel to dense embeddings):**
- Build a BM25 keyword index alongside the dense vector index
- BM25 complements dense retrieval — it is strong for exact-match queries (specific account names, ticket IDs, product feature names) where semantic similarity may underperform
- Store both indexes; run both at retrieval time for hybrid retrieval

---

### Stage 5: Vector Store and Metadata Index

**Purpose:** Store dense vectors and chunk metadata in a system that supports fast approximate nearest neighbour search with metadata filtering.

**Vector store options (capabilities overview):**
- Pinecone: managed, production-grade, strong metadata filtering, serverless tier
- Weaviate: open-source, strong hybrid search, GraphQL API
- Qdrant: open-source, strong metadata filtering, good performance at scale
- pgvector: Postgres extension; good for teams already on Postgres; simpler operations
- ChromaDB: good for prototyping; not recommended for production at scale

**Key requirements for enterprise use:**
- Metadata filtering: filter by source_type, entity_tag, date range, customer_tier — before or after retrieval, without returning and then filtering in application code
- Namespace or collection isolation: separate namespaces per customer for multi-tenant deployments
- Access control at the query layer: different users can query different namespaces
- Persistence and backup: vectors must be durable and backed up on a schedule

---

## Retrieval-Generation Pipeline

### Stage 6: Query Processing

**Purpose:** Transform the user's query into one or more retrieval queries that maximise the chance of retrieving the most relevant chunks.

**Query processing techniques:**
- **Embedding the raw query:** The simplest approach; works well for specific, well-formed queries
- **Query expansion:** Generate N rephrasings of the query using an LLM; run retrieval for each; merge results. Improves recall for ambiguous or under-specified queries.
- **HyDE (Hypothetical Document Embedding):** Generate a hypothetical answer to the query and embed that answer for retrieval. The embedding of a hypothetical answer is often more similar to the embedding of the actual answer document than the embedding of the question itself.
- **Step-back prompting:** Generate a more abstract version of the query (the "step back") to retrieve conceptual background before retrieving specific facts

---

### Stage 7: Hybrid Retrieval

**Purpose:** Retrieve the most relevant chunks from the index using a combination of dense semantic search and sparse keyword search.

**Hybrid retrieval algorithm:**
1. Embed the query (and query variants if using expansion)
2. Run dense ANN search: retrieve top-K chunks by cosine similarity
3. Run BM25 keyword search: retrieve top-K chunks by BM25 score
4. Deduplicate results across both searches
5. Apply metadata filters (date range, entity tag, access control scope)
6. Merge and score: Reciprocal Rank Fusion (RRF) or weighted linear combination of dense + sparse scores

**Retrieval hyperparameters (configurable per use case):**
- K_dense: number of chunks from dense retrieval (default: 20)
- K_sparse: number of chunks from BM25 retrieval (default: 20)
- Date range filter: default 90 days for most use cases; 180 days for historical synthesis
- Entity scope filter: always applied for multi-tenant or account-scoped queries

---

### Stage 8: Reranking

**Purpose:** Re-score the merged retrieval results using a more compute-intensive model that better captures relevance for the specific query.

**Reranker options:**
- **Cross-encoder reranker:** Takes the query + each candidate chunk as a pair; scores relevance directly. More accurate than bi-encoder similarity but O(N) compute per query (N = number of candidates). Best for N ≤ 50.
- **ColBERT-style late interaction:** Faster than cross-encoder; stores per-token embeddings for each chunk; enables token-level matching at retrieval time. Better at exact-term matching.
- **LLM-based reranking:** Use an LLM to rate the relevance of each candidate chunk. High quality but high latency and cost; appropriate for offline or low-latency-tolerant use cases.

**Output:** Top-N reranked chunks (N is typically 5–10 for the context window; depends on total token budget).

---

### Stage 9: Context Assembly

**Purpose:** Format the retrieved chunks into a context window that the generation model can use effectively.

**Context assembly decisions:**
- **Ordering:** Most recent first? Most relevant first? By source type? Research suggests recency bias in attention — the model attends more strongly to context at the beginning and end of the window. Place the most important chunks first and last.
- **Source metadata inclusion:** Include source metadata (source_type, date, entity_tag) before each chunk. This enables the model to reason about source authority and recency.
- **Token budget management:** The assembled context must fit within the model's context window minus the system prompt, few-shot examples, and expected output length. If retrieved chunks exceed the budget, prioritise by reranker score.
- **Chunk deduplication:** If the same information appears in multiple chunks (from different ingestion events), deduplicate before assembly.

**Context template:**
```
[SOURCE: Zendesk Ticket #4521 | Date: 2026-05-15 | Account: Acme Corp | Type: support]
Customer reports that export is timing out for datasets > 10,000 rows. 
Error message: "Export failed: timeout after 30s."

[SOURCE: Gong Call Transcript | Date: 2026-05-20 | Account: Acme Corp | Speaker: CSM]
Customer mentioned during check-in that the export issue is affecting 
their end-of-month reporting workflow. They are considering a workaround.
```

---

### Stage 10: Answer Generation

**Purpose:** Generate a response grounded in the assembled context.

**System prompt structure:**
```
You are an enterprise AI assistant for [PRODUCT_NAME]. Your task is [TASK_DESCRIPTION].

Rules:
1. Ground every factual claim in the provided context. 
2. Cite each claim with [SOURCE_ID].
3. If the context does not contain enough information to answer the question, 
   say explicitly: "I do not have sufficient information to answer this."
4. Do not speculate or infer beyond what the context states.
5. [ADDITIONAL_RULES_PER_USE_CASE]
```

**Output format:** Defined per use case (JSON schema, structured prose, etc.). Output format validation should be applied post-generation.

---

### Stage 11: Post-Processing

**Purpose:** Validate and annotate the generated output before delivery.

**Post-processing steps:**
1. **Citation mapping:** Parse [SOURCE_ID] markers in the output; resolve to full source metadata
2. **Hallucination check:** For each cited claim, verify the source chunk contains supporting text (using a lightweight classifier or a secondary LLM call)
3. **Schema validation:** If output must conform to a JSON schema, validate and re-prompt if validation fails
4. **Content policy check:** Screen output for policy violations (PII leakage, competitive claims that violate guidelines, etc.)
5. **Confidence scoring:** Assign an overall confidence score based on: number of supporting citations, reranker scores of cited chunks, any "I don't know" hedges in the output
6. **Delivery:** Return output to the requesting agent or user interface

---

## Multi-Tenant Architecture Considerations

For a B2B SaaS platform where each customer's data must be isolated:

- **Namespace isolation:** Separate vector store namespace per customer. Queries cannot cross namespace boundaries.
- **Access control at retrieval layer:** Not just at the application layer. The query must include the customer namespace filter at the vector store API level.
- **Ingestion isolation:** Each customer's ingestion pipeline writes to their namespace only. A bug in one customer's ingestion cannot pollute another customer's index.
- **PII handling:** Each customer's PII handling configuration (redaction rules, retention policy) is applied at ingestion time for their namespace.

---

## Cost Architecture

| Stage | Primary Cost Driver | Cost Optimisation |
|---|---|---|
| Ingestion (embedding) | Tokens × embedding model price | Embed only new or changed documents; cache embeddings |
| Vector storage | Vectors stored × storage price | Prune old/irrelevant vectors; use compression |
| Retrieval (embedding query) | Queries × embedding model price | Cache embeddings for repeated queries |
| Retrieval (reranking) | Reranker inference per query | Only rerank the top-K from first-pass retrieval |
| Generation | Tokens × LLM price | Context window management; prompt caching; smaller models for simple tasks |

---

*See also: [Chunking, Embeddings, and Retrieval](/rag-workflow-documentation/chunking-embeddings-retrieval.md) · [Reranking and Citation Grounding](/rag-workflow-documentation/reranking-citation-grounding.md) · [RAG Failure Modes](/rag-workflow-documentation/rag-failure-modes.md)*
