# RAG Evaluation

**Retrieval precision, recall, faithfulness, answer relevance, and context utilisation**

---

## The Four RAG Eval Dimensions

RAG evaluation requires measuring quality at two stages: retrieval and generation. Failure at retrieval cannot be compensated at generation, and failures at generation may not be visible from retrieval metrics alone. Both must be measured.

| Dimension | Stage | Primary Metric |
|---|---|---|
| Retrieval recall | Retrieval | Recall@K |
| Retrieval precision | Retrieval | Precision@K |
| Faithfulness | Generation | % claims grounded in context |
| Answer relevance | Generation | Relevance score (1-5) |
| Context utilisation | Generation | % retrieved chunks cited |

---

## Retrieval Evaluation

### Recall@K

**Definition:** Of all chunks in the corpus that are relevant to a query, what fraction appear in the top-K retrieved results?

**Formula:** Recall@K = (number of relevant chunks in top-K) / (total number of relevant chunks for the query)

**Why it matters:** If the relevant chunk is not in the top-K, the model cannot use it. Low recall means the system is missing relevant information regardless of how well it generates.

**Target:** >80% Recall@10

**Measurement:**
1. Take a golden dataset with (query, relevant_chunk_ids) pairs
2. For each query, run retrieval and get the top-K chunk IDs
3. Calculate: intersection of retrieved IDs and relevant IDs, divided by total relevant IDs
4. Average across all queries in the dataset

**Recall@K by difficulty category:**

| Category | Target Recall@10 |
|---|---|
| Easy (single chunk, explicit answer) | >95% |
| Medium (multi-chunk synthesis) | >75% |
| Hard (implicit reasoning, edge cases) | >60% |

---

### Precision@K

**Definition:** Of the top-K retrieved results, what fraction are actually relevant to the query?

**Formula:** Precision@K = (number of relevant chunks in top-K) / K

**Why it matters:** Low precision means noisy context. Irrelevant chunks consume the context window budget and may confuse the generation model.

**Target:** >70% Precision@5

---

### Mean Reciprocal Rank (MRR)

**Definition:** The average of the reciprocal rank of the first relevant result across all queries.

**Formula:** MRR = (1/Q) × Σ(1/rank_of_first_relevant_result)

Where rank 1 → reciprocal rank 1.0, rank 2 → 0.5, rank 5 → 0.2, etc.

**Target:** >0.7

**Why MRR:** Rewards systems that surface the most relevant result at the top, not just somewhere in the top-K.

---

### Tracking Retrieval Metrics Over Time

Run retrieval eval on the same golden dataset weekly. Track:
- Absolute values (are we above threshold?)
- Week-over-week trend (are metrics improving, stable, or declining?)
- Performance by query category (which categories are weakest?)

Alert when any metric drops below threshold on two consecutive runs. Investigate root cause:
- Corpus growth causing index noise?
- Embedding model update with different representation space?
- Metadata filter configuration change?

---

## Generation Evaluation

### Faithfulness

**Definition:** The fraction of factual claims in the generated output that are grounded in the retrieved context.

**Measurement method (LLM-as-judge):** For a sample of production outputs, extract each factual claim and run through the Faithfulness Rubric (see [LLM-as-Judge Rubrics](llm-as-judge-rubrics.md)).

**Target:** >95% claims SUPPORTED or PARTIALLY_SUPPORTED

**Leading cause of faithfulness failures:**
1. Inadequate system prompt (model not constrained to context-only)
2. Context window overflow (relevant context truncated)
3. Confident model with strong training priors overriding retrieved context

---

### Answer Relevance

**Definition:** The degree to which the generated output answers the user's actual question.

**Measurement method (LLM-as-judge):** Run the Answer Relevance Rubric on a sample of (question, answer) pairs. Human calibration quarterly.

**Target:** Average score >4.0/5.0

**Key distinction from faithfulness:** A response can be perfectly faithful (every claim is grounded) while being irrelevant (it answers a different question than was asked). This often indicates a retrieval failure — the wrong chunks were retrieved and the model faithfully answered the question implied by those chunks.

---

### Context Utilisation

**Definition:** What fraction of the retrieved chunks were cited in the generated output?

**Measurement:** Parse citations from the output. Count unique cited chunk IDs. Divide by total retrieved chunks.

**Target:** There is no single "right" target for context utilisation. Relevant targets:
- If utilisation is very low (<20%): indicates retrieval is surfacing too many irrelevant chunks (precision problem)
- If utilisation is very high (>80%): can be a sign that too few chunks are being retrieved, or that the model is citing everything indiscriminately

**Track as a signal, not as a primary quality metric.** Changes in utilisation should prompt investigation.

---

### Citation Accuracy

**Definition:** The fraction of cited sources that actually contain information supporting the claim that cites them.

**Measurement method (LLM-as-judge):** For each cited claim, run the Citation Accuracy Rubric.

**Target:** >95% citations SUPPORTED or PARTIALLY_SUPPORTED

**Leading causes of citation inaccuracy:**
1. Model generating a plausible-sounding citation ID that does not correspond to a real source
2. Model correctly citing a source but for a claim that the source does not actually support
3. Model conflating information from two sources and citing only one

---

## RAGAs Framework (Reference)

RAGAs (Retrieval-Augmented Generation Assessment) is an open-source framework for evaluating RAG pipelines. It implements the core metrics described above with standard interfaces. Reference for implementation:

- **Faithfulness:** RAGAs uses an LLM judge to check if each statement in the answer can be inferred from the context
- **Answer Relevance:** RAGAs uses an LLM to generate questions from the answer and measures their similarity to the original question
- **Context Recall:** RAGAs checks what fraction of the ground truth answer can be attributed to the retrieved context
- **Context Precision:** RAGAs checks what fraction of the retrieved context is useful for answering the question

Note: RAGAs metrics provide a useful starting point but should be calibrated against human annotation for your specific domain. Off-the-shelf metric implementations may not perfectly reflect what quality means for your use case.

---

## Connecting Retrieval and Generation Evals

Run retrieval and generation evals on the same items where possible. This allows you to distinguish between two failure types:

| Failure Pattern | Retrieval Metric | Generation Metric | Root Cause |
|---|---|---|---|
| Retrieval miss | Low recall | Low answer relevance | Retrieval failure — the right content is not being found |
| Retrieval noise | Low precision | Low faithfulness (model cites wrong chunks) | Retrieval precision failure |
| Generation hallucination | Normal | Low faithfulness | Generation failure — model ignoring context |
| Context overflow | Normal | Low answer relevance (relevant chunks lost) | Architecture failure — context window too full |

---

*See also: [RAG Eval Plan](/rag-workflow-documentation/rag-eval-plan.md) · [Golden Dataset Design](golden-dataset-design.md) · [LLM-as-Judge Rubrics](llm-as-judge-rubrics.md)*
