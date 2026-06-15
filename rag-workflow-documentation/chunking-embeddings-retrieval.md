# Chunking, Embeddings, and Retrieval: A Deep Dive

---

## Why Chunking Is a Product Decision

Chunking is where most RAG systems fail. Engineers default to fixed-size chunking because it is easy to implement. Product managers rarely question it because it sounds like an infrastructure detail. But the chunking strategy determines what information unit the retrieval system returns — and the wrong chunking choice can make a RAG system useless regardless of how good the embedding model or the LLM is.

The principle: a good chunk is the smallest unit of text that contains a complete, coherent piece of information that can be retrieved independently and still be useful to the language model when placed in context.

---

## Chunking Strategies

### Fixed-Size Chunking

**How it works:** Split the document every N tokens, with M tokens of overlap between consecutive chunks.

**Example:** Chunk size = 512 tokens, overlap = 50 tokens. Every chunk is 512 tokens. Chunks 1 and 2 share their last/first 50 tokens.

**Advantages:** Simple to implement. Deterministic. Works reasonably well for uniform content (structured records, database exports, forms).

**Disadvantages:** Splits topic boundaries arbitrarily. A chunk may start mid-sentence or mid-argument. Important context that crosses a chunk boundary is split. Overlap mitigates this partially but not fully.

**When to use:** Structured, uniform content where topic boundaries are not a concern. Not recommended for conversational content, long-form documents, or transcripts.

---

### Sentence-Window Chunking

**How it works:** Index individual sentences as retrieval units. When a sentence is retrieved, include the surrounding N sentences as context (the "window") in the context assembly stage.

**Example:** Index each sentence independently. Retrieve "The export feature is timing out for datasets > 10,000 rows." Return the sentence + 2 sentences before and 2 sentences after as the context window.

**Advantages:** High retrieval precision — individual sentences are highly specific. Surrounding context provides enough information for the model to use the retrieved passage. Works very well for support tickets, CRM notes, and other short, factual content.

**Disadvantages:** Large index size (many more chunks per document). Sentence splitting on noisy text (informal Slack messages, casual emails) can produce poor splits. Retrieved context window may still miss important context from earlier in the document.

**When to use:** Support tickets, CRM account notes, Slack messages, short factual records.

---

### Semantic Chunking

**How it works:** Split the document at points where the semantic content changes significantly. Detect topic boundaries by comparing the embedding similarity of consecutive text units — when similarity drops below a threshold, it indicates a topic boundary.

**Example:** A product specification document covers four distinct features. Semantic chunking identifies the four sections as separate chunks, even if they are not explicitly labeled with headers.

**Advantages:** Chunks follow topic boundaries rather than arbitrary size limits. More coherent chunks mean better retrieval precision. Particularly effective for long-form content with identifiable sections.

**Disadvantages:** More computationally expensive at ingestion time (requires embedding each sentence or paragraph to detect boundaries). Similarity threshold is a hyperparameter that requires tuning. May produce uneven chunk sizes that complicate token budget management.

**When to use:** Product documentation, strategy documents, long-form technical documentation, PRDs, meeting summaries with distinct agenda items.

---

### Parent-Child Chunking

**How it works:** Maintain two granularities: large parent chunks (400–800 tokens) and small child chunks (100–200 tokens) derived from the parents. Index only the child chunks for retrieval; return the parent chunk as the context when a child is retrieved.

**Example:** A support ticket thread is divided into individual messages (child chunks). When a message is retrieved, the entire ticket thread (parent chunk) is returned as the context.

**Advantages:** Small children → high retrieval precision (specific passages found accurately). Large parents → sufficient context for the model to understand and use the retrieved information. Eliminates the precision-context tradeoff of other approaches.

**Disadvantages:** Requires maintaining a parent-child mapping, which adds index complexity. Parent chunks may exceed the token budget if they are too large. Not all document structures have a natural parent-child hierarchy.

**When to use:** Threaded discussions (Slack, email, support tickets), documents with natural hierarchies (legal contracts with clauses and sub-clauses, meeting notes with agenda items and discussion points).

---

### Recursive Character Splitting

**How it works:** Split at progressively smaller text units (paragraph → sentence → word) until the chunk reaches the target size. This respects natural text boundaries while still hitting a target size.

**Advantages:** Respects natural text boundaries for most content. Works well as a general-purpose default when content type is mixed. Easy to implement.

**When to use:** Mixed-format documents where content type is unknown at ingestion time. A reasonable default for a system ingesting diverse content types.

---

## Embedding Model Selection

The embedding model converts text into a dense vector representation. The quality of the embedding model determines how well the semantic similarity search works — how accurately it retrieves chunks that are semantically related to the query.

### Key Selection Criteria

**Context length:** The embedding model must support the chunk size you are using. Models with a 512-token context limit cannot meaningfully embed 700-token chunks — they truncate or average the excess tokens, degrading quality.

**Dimension size:** Standard options are 768 or 1536 dimensions. Higher dimensions capture more semantic nuance but require more storage and compute for retrieval. For most enterprise B2B text (English-language, domain-standard vocabulary), 768 dimensions is sufficient.

**Performance on your domain:** General-purpose embedding models are trained on broad web text. If your corpus is highly specialised (legal contracts, medical records, semiconductor design specifications), a domain-adapted model may outperform a general-purpose model significantly. For typical enterprise SaaS text (CRM notes, support tickets, Slack messages), general-purpose models perform well.

**Batch throughput:** Ingestion requires embedding large volumes of text in batch. Evaluate the model's throughput for bulk embedding, not just single-inference latency.

**API stability and cost:** If using an embedding API (rather than self-hosted), consider provider stability and cost at production scale. Embedding cost is paid at ingestion time, not at query time — so the cost is proportional to corpus size × update frequency, not query volume.

### Embedding Models Overview

| Model | Provider | Context | Dimensions | Notes |
|---|---|---|---|---|
| text-embedding-3-small | OpenAI | 8,191 tokens | 1,536 | Cost-effective; strong general performance |
| text-embedding-3-large | OpenAI | 8,191 tokens | 3,072 | Higher quality; higher cost |
| all-mpnet-base-v2 | HuggingFace/open source | 384 tokens | 768 | Strong open-source option; good for shorter chunks |
| BGE-large-en | BAAI/open source | 512 tokens | 1,024 | Strong retrieval performance on BEIR benchmarks |
| Cohere Embed v3 | Cohere | 512 tokens | 1,024 | Strong for retrieval; good multilingual support |

*Use illustrative benchmarks for selection; run your own eval on a sample of your actual corpus before committing to a model.*

---

## Retrieval Approaches

### Dense Retrieval (Semantic Search)

**How it works:** Embed the query using the same embedding model used at ingestion. Search the vector index for the K nearest neighbours by cosine similarity (or dot product for normalised vectors).

**Strengths:** Finds semantically similar content even when the query and the relevant chunk use different words. Handles synonyms, paraphrases, and conceptual similarity well.

**Weaknesses:** Poor at exact-term matching. If the query is "ticket #4521", dense retrieval may return semantically similar tickets but not necessarily ticket #4521 specifically. Also struggles with rare terms that are not well-represented in the embedding model's training distribution.

---

### Sparse Retrieval (BM25 / Keyword Search)

**How it works:** Build an inverted index of terms across all chunks (similar to a traditional search engine). Score each chunk using BM25 (a probabilistic relevance function that weights term frequency, inverse document frequency, and document length normalisation).

**Strengths:** Exact-term matching. If the query contains specific identifiers (account names, ticket IDs, product feature names, technical terms), BM25 will find documents containing those exact terms reliably.

**Weaknesses:** No semantic understanding. Cannot find documents about "billing issues" when the query says "invoice problems" unless the query is expanded.

---

### Hybrid Retrieval

**How it works:** Run both dense and sparse retrieval independently. Merge the results using Reciprocal Rank Fusion (RRF) or a weighted linear combination.

**Reciprocal Rank Fusion formula:**
```
RRF_score(document, query) = Σ_over_rankings: 1 / (rank + k)
where k = 60 (common default)
```

RRF rewards documents that appear in the top ranks of multiple retrieval methods. A document ranked #3 by dense and #5 by sparse gets a higher combined score than one ranked #1 by dense and not appearing in sparse results at all.

**Why hybrid outperforms either alone:**
- Dense retrieval catches semantically related content that sparse misses
- Sparse retrieval catches exact-term matches that dense may underrank
- RRF fusion ensures that documents relevant by both methods get a strong combined score

**Hybrid retrieval is the recommended default for enterprise RAG.** The cost overhead is minimal (running both searches in parallel) and the quality improvement is consistent across document types.

---

### Metadata-Filtered Retrieval

**How it works:** Apply metadata filters to the retrieval query before or after vector search. Example filters: account_id = "ACC-001", date >= "2026-01-01", source_type = "support_ticket".

**Pre-filtering:** Apply metadata filter before vector search. Reduces the search space; faster and more cost-efficient. May miss relevant chunks if the filter is too narrow.

**Post-filtering:** Apply metadata filter after retrieving top-K. More recall (does not narrow the search space) but more expensive (must retrieve more chunks to account for those that will be filtered out).

**Indexed metadata filtering:** The best approach — vector stores that support filtering at the index level (e.g., Pinecone, Weaviate, Qdrant) apply the filter as part of the ANN search, not as a post-hoc step. This gives you the precision of pre-filtering and the performance of native index search.

---

## Retrieval Quality Evaluation

The primary retrieval metrics are:

**Recall@K:** Of all relevant chunks for a query, what fraction appear in the top-K retrieved results? High recall is critical — if the relevant chunk is not in the retrieved set, the model cannot use it, regardless of how good the generation is.

**Precision@K:** Of the top-K retrieved results, what fraction are actually relevant? Low precision means noise in the context window — irrelevant chunks that may confuse the model.

**MRR (Mean Reciprocal Rank):** The average of the reciprocal ranks of the first relevant result across all queries. MRR = 1/rank_of_first_relevant_result. High MRR means relevant results appear near the top.

**Target benchmarks (illustrative):**
- Recall@10: >80% on a representative golden query set
- Precision@5: >70%
- MRR: >0.7

See [RAG Evaluation Plan](/rag-workflow-documentation/rag-eval-plan.md) for full evaluation methodology.

---

*See also: [Enterprise RAG Architecture](/rag-workflow-documentation/enterprise-rag-architecture.md) · [Reranking and Citation Grounding](/rag-workflow-documentation/reranking-citation-grounding.md) · [RAG Failure Modes](/rag-workflow-documentation/rag-failure-modes.md)*
