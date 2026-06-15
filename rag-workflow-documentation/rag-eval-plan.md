# RAG Evaluation Plan

**How to evaluate a RAG system: retrieval metrics, generation metrics, evaluation datasets, and continuous monitoring**

---

## Why Evaluation Is a Product Responsibility

RAG system quality degrades in ways that are not always visible to users — retrieval recall can drop when the corpus grows and the index becomes noisier, citation accuracy can drift as the prompt is updated without re-testing, and specific failure modes may be clustered in a category of queries that the engineering team never tests because they are not the primary use case.

Evaluation is not a one-time activity. It is a continuous product function, analogous to monitoring dashboards in a data product. The PM's role is to define what quality looks like, ensure the evaluation infrastructure is built, and review the results regularly.

---

## The Three Layers of RAG Evaluation

### Layer 1: Retrieval Evaluation (How good is what we retrieved?)

Measures the quality of the retrieval pipeline independently of the generation pipeline. If retrieval is bad, generation will be bad — no generation quality improvement can compensate for missing or irrelevant retrieved chunks.

### Layer 2: Generation Evaluation (How good is what we generated from what we retrieved?)

Measures the quality of the generated output given the retrieved context. Includes faithfulness (does the output match the retrieved context?), answer relevance (does the output answer the question?), and citation accuracy (do the citations correctly map to sources?).

### Layer 3: End-to-End Task Evaluation (Did the user accomplish their goal?)

Measures whether the user's actual goal was accomplished. A technically faithful answer that is not useful to the user is still a failure. This layer connects the AI system's output to business outcomes.

---

## Layer 1: Retrieval Evaluation

### Metrics

**Recall@K**
- Definition: Of all chunks in the corpus that are relevant to a query, what fraction appear in the top-K retrieved results?
- Formula: (number of relevant chunks in top-K) / (total number of relevant chunks for the query)
- Target: >80% Recall@10 on the golden query set
- Why it matters: If the relevant chunk is not retrieved, the model cannot use it. Recall is the foundational retrieval metric.

**Precision@K**
- Definition: Of the top-K retrieved results, what fraction are actually relevant to the query?
- Formula: (number of relevant chunks in top-K) / K
- Target: >70% Precision@5
- Why it matters: Low precision means noisy context — irrelevant chunks that may confuse the model or dilute the relevant context.

**MRR (Mean Reciprocal Rank)**
- Definition: The average of the reciprocal rank of the first relevant result across all queries in the evaluation set. MRR = (1/Q) × Σ(1/rank_of_first_relevant_result)
- Target: >0.7
- Why it matters: Rewards systems that surface relevant results near the top of the ranking, not just somewhere in the top-K.

**NDCG@K (Normalised Discounted Cumulative Gain)**
- Definition: Measures ranking quality by giving progressively less credit to relevant results that appear lower in the ranking.
- Why it matters: More nuanced than recall or precision — accounts for the position of relevant results in the ranked list.
- When to use: When some chunks are more relevant than others (e.g., the exact ticket is more relevant than a related ticket).

---

### Building the Golden Retrieval Dataset

A golden retrieval dataset is a set of (query, relevant_chunk_ids) pairs used to evaluate retrieval quality.

**How to build it:**
1. Sample 100–300 representative queries from production logs (or create them manually for a new system)
2. For each query, have a human expert identify which chunks in the corpus are relevant
3. Record: query_text, relevant_chunk_ids[], difficulty_label (easy / medium / hard), query_category (account-specific / topic-based / temporal / entity-ambiguous)
4. Include hard negatives: chunks that appear relevant but are not (same account, different time period; related feature, different product area)

**Annotation process:**
- Two human annotators per query; resolve disagreements via a third annotator or by consensus
- Annotators should be domain experts (CSMs for CS queries, PMs for product queries), not generic annotators
- Annotator agreement rate is itself a quality metric — low agreement indicates ambiguous queries that may need to be split or excluded

**Dataset size:** 200 query-answer pairs is sufficient for regular evaluation. 50 per query category minimum.

**Update cadence:** Add new queries to the golden set when: (a) a new use case is added to the system, (b) a novel failure mode is discovered in production, (c) the corpus expands significantly with a new data source.

---

## Layer 2: Generation Evaluation

### Metrics

**Faithfulness**
- Definition: The fraction of claims in the generated output that are grounded in the retrieved context. A claim is "faithful" if it can be traced to a specific retrieved chunk.
- Measurement: For a sample of outputs, have a human annotator (or LLM-as-judge) rate each claim as: fully supported / partially supported / not supported by the retrieved context.
- Target: >95% of claims fully supported (for citation-enforced outputs)
- Automated: Use LLM-as-judge with a faithfulness rubric calibrated against human annotation

**Answer Relevance**
- Definition: Does the generated output actually answer the user's question?
- Measurement: Human rating (1–5 scale) or LLM-as-judge rating of whether the output addresses the question asked.
- Target: >4.0/5.0 average (human-rated), on a representative sample
- Note: High faithfulness + low answer relevance indicates the system is retrieving and faithfully reporting information that is not what the user needed (retrieval precision problem)

**Citation Accuracy**
- Definition: The fraction of cited claims where the cited source actually supports the claim.
- Measurement: For each citation, retrieve the source chunk and assess whether the source text contains information supporting the claim.
- Target: >95%
- Automated: LLM-as-judge with citation accuracy rubric: "Does the following source text support this claim? Return: supported / not_supported / partially_supported"

**Context Utilisation**
- Definition: What fraction of the retrieved context was used in generating the output? Low utilisation may indicate that the retrieval is surfacing irrelevant chunks (noise).
- Measurement: Parse the output citations; calculate the fraction of retrieved chunks that were cited at least once.
- Note: 100% utilisation is not the goal — some retrieved chunks will be redundant. But <20% utilisation may signal a retrieval quality problem.

**Hallucination Rate**
- Definition: The fraction of generated outputs that contain at least one claim not supported by the retrieved context.
- Target: <3% of outputs contain any unsupported claim (for citation-enforced systems)
- Measurement: Citation accuracy evaluation on a sample; flag any output with a "not supported" citation

---

### LLM-as-Judge for Generation Quality

For scalable automated evaluation, use an LLM as an evaluator ("LLM-as-judge"). The LLM is given the query, the retrieved context, and the generated output, and asked to evaluate specific quality dimensions.

**Faithfulness rubric prompt:**
```
You are an expert quality evaluator. Your task is to assess whether a generated answer 
is faithful to the provided context.

Context:
[RETRIEVED_CONTEXT]

Generated Answer:
[GENERATED_ANSWER]

For each factual claim in the Generated Answer, assess:
- Is this claim explicitly supported by the provided Context? (supported / partially_supported / not_supported)
- If supported, which source in the Context supports it?

Return your assessment as a JSON array, one entry per claim.
```

**Calibration:** Before using LLM-as-judge in production, calibrate against human annotations. Run 50 examples through both human annotation and LLM-as-judge. Measure agreement. If agreement is below 80%, review and refine the judge rubric.

**Anti-patterns to avoid:**
- Using the same model for generation and evaluation (the judge may be biased toward its own generation style)
- Using the judge to evaluate without providing the retrieved context (it cannot assess faithfulness without the context)
- Using a single judge dimension for all failure modes (use separate rubrics for faithfulness, relevance, and citation accuracy)

---

## Layer 3: End-to-End Task Evaluation

End-to-end evaluation measures whether the AI system actually helps users accomplish their goals. This is harder to automate and requires instrumentation at the product layer.

### Online Metrics for RAG-Powered Products

**Acceptance rate:** What fraction of AI-generated outputs are accepted (approved, used, forwarded) by users without substantial modification?

**Override rate:** What fraction of AI-generated outputs are substantially modified before use? Which sections or claim types are overridden most?

**"Not enough information" rate:** What fraction of queries return a "no information" response? By query category?

**Task completion rate:** For agent-driven workflows using RAG, what fraction of tasks complete successfully end-to-end?

**User-reported accuracy:** Post-session prompt ("Was the information in this response accurate?") — optional binary or 1-5 scale.

**Business outcome attribution:** For AI-assisted decisions (e.g., CSM takes action based on AI recommendation), does the outcome improve vs. the baseline? (Requires A/B testing or before/after analysis.)

---

## Continuous Monitoring Plan

| Metric | Frequency | Method | Alert Threshold |
|---|---|---|---|
| Recall@10 (retrieval) | Weekly (automated) | Golden retrieval dataset | <75% triggers review |
| Precision@5 (retrieval) | Weekly (automated) | Golden retrieval dataset | <60% triggers review |
| Faithfulness (generation) | Monthly (LLM-as-judge) | 50 random output sample | <90% triggers review |
| Citation accuracy | Monthly (LLM-as-judge + human spot-check) | 30 output sample | <92% triggers review |
| "Not enough information" rate | Daily (automated) | Production query log | >20% for any category triggers review |
| Data freshness per source | Daily (automated) | Source connector monitoring | Staleness >SLA triggers alert |
| User acceptance rate | Weekly (automated) | Product instrumentation | <60% triggers review |

---

## Evaluation Infrastructure Requirements

**What the PM should define and ensure is built:**

1. **Golden dataset storage:** Version-controlled dataset repository (not a spreadsheet). Every evaluation run records which version of the golden dataset was used.

2. **Evaluation pipeline:** Automated pipeline that runs retrieval evaluation on a schedule and reports results to a dashboard.

3. **LLM-as-judge pipeline:** Automated pipeline that samples production outputs, runs them through the judge rubric, and reports faithfulness and citation accuracy.

4. **Production instrumentation:** User acceptance rate and override rate tracked at the product layer (not inferred from server logs).

5. **Evaluation run log:** Every evaluation run (date, dataset version, metric results, model/index version evaluated) is logged for trend analysis.

---

*See also: [RAG Failure Modes](/rag-workflow-documentation/rag-failure-modes.md) · [LLM-as-Judge Rubrics](/ai-evaluation-frameworks/llm-as-judge-rubrics.md) · [Online AI Product Metrics](/ai-evaluation-frameworks/online-ai-product-metrics.md)*
