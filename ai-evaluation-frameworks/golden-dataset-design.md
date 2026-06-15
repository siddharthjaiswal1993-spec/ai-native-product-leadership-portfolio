# Golden Dataset Design

**How to design, collect, annotate, version, and maintain a golden evaluation dataset for AI products**

---

## What Is a Golden Dataset

A golden dataset is a curated set of inputs with known-correct outputs (or known-correct output properties), used to evaluate an AI system's quality in a repeatable, comparable way. It is the foundation of offline evaluation.

Without a golden dataset, you cannot:
- Set a quality baseline before shipping
- Detect regressions when models or prompts change
- Compare quality across different model or retrieval configurations
- Communicate quality to stakeholders with evidence

A golden dataset is not a test suite in the traditional sense. It is more like a calibration instrument: a well-constructed sample that represents the real distribution of inputs the system will encounter, with ground truth established by human domain experts.

---

## Design Principles

### 1. Represent the Real Input Distribution

The golden dataset must reflect what real users actually ask or input — not what you think they will ask. The best source is production logs (anonymised) from a pilot or beta deployment. If you do not have production logs yet, use:
- Domain expert input: have 3–5 domain experts (CSMs, PMs, field consultants — whoever uses the system) generate representative queries
- Use case coverage matrix: enumerate the primary use cases and ensure each is represented in the dataset

### 2. Include Edge Cases, Not Just Happy Paths

If your golden dataset only covers straightforward, well-formed inputs, it will not catch failures in the cases that actually matter most. Required edge case categories:
- **Ambiguous inputs:** Queries where the intent is not clear
- **Out-of-scope inputs:** Queries the system should decline gracefully
- **Low-signal inputs:** Queries where the corpus does not have enough information to answer well
- **Adversarial inputs:** Inputs designed to probe failure modes (see [Eval Generation Prompts](/prompt-library/eval-generation-prompts.md))
- **Entity-ambiguous inputs:** Queries where the entity is underspecified or could match multiple entities

### 3. Calibrate Difficulty

Include a mix of easy, medium, and hard inputs. A dataset that is all easy will show high scores that do not reflect real-world performance. A dataset that is all hard will produce discouragingly low scores that do not reflect normal use quality.

**Suggested distribution:** 30% easy, 50% medium, 20% hard.

### 4. Separate by Dimension

Not all quality dimensions can be evaluated with the same test cases. Design separate subsets for:
- Retrieval quality (query → relevant chunks)
- Generation faithfulness (retrieved context + query → output grounded in context)
- Answer relevance (query + output → did it answer the question?)
- Citation accuracy (output + sources → are citations correct?)
- Format compliance (output → does it match the schema?)

---

## Dataset Size Guidelines

| System Maturity | Minimum Dataset Size | Notes |
|---|---|---|
| Pre-launch (alpha/beta) | 50–100 items | Focus on core happy paths and critical edge cases |
| Post-launch (early production) | 150–300 items | Add edge cases from production failure reports |
| Mature production | 300–500 items | Full coverage of use cases; separate subsets per dimension |
| Large scale | 500+ items | Consider stratified sampling to manage annotation cost |

**Warning:** More is not always better if the additional items are redundant or poorly annotated. A 100-item golden dataset with high-quality annotations is more valuable than a 1000-item dataset with inconsistent annotations.

---

## Collection Process

### Step 1: Define Coverage Matrix

Before collecting items, define what needs to be covered:

| Dimension | Use Case | Target Coverage |
|---|---|---|
| Query type | Account-specific, topic-based, temporal, comparative | 5 items each minimum |
| Difficulty | Easy, medium, hard | 30/50/20 split |
| Signal type | Usage, support, CRM, transcript | 20% from each source type |
| Edge cases | Adversarial, out-of-scope, ambiguous | 10–15 items |

### Step 2: Source Collection

**From production logs (preferred):**
1. Sample from production query logs (anonymise account names where needed)
2. Stratify by query type and ensure coverage across all categories in the matrix
3. Exclude queries from the pilot users who will do annotation (avoid circular evaluation)

**Manual generation (for pre-launch systems):**
1. Run a "query generation session" with 3–5 domain experts: "Write 10 queries you would actually want to ask this system"
2. Add coverage-matrix-driven items for categories not covered by expert queries
3. Add adversarial inputs using [Eval Generation Prompts](/prompt-library/eval-generation-prompts.md)

### Step 3: Ground Truth Establishment

For each item in the dataset, establish the ground truth:

**For retrieval evaluation:**
- Identify which chunk IDs in the corpus are relevant to this query
- Annotate relevance as: HIGHLY_RELEVANT / RELEVANT / MARGINALLY_RELEVANT / NOT_RELEVANT
- The "relevant_chunk_ids" field should include HIGHLY_RELEVANT and RELEVANT chunks

**For generation evaluation:**
- Write or identify the gold answer — the correct, complete response to this query
- Note: this does not need to be a verbatim string match. For non-deterministic outputs, define the PROPERTIES the answer must have: "must mention the Q4 usage decline", "must cite a source from the last 90 days", "must include a specific recommended action"

**For adversarial/out-of-scope inputs:**
- Document the CORRECT BEHAVIOR — what the system should do (usually: graceful refusal or "I don't have information about this")
- Document what a FAILURE looks like (hallucination, scope violation, fabricated answer)

---

## Annotation Process

### Annotator Selection

Annotators must be domain experts who understand the task. For a CS intelligence system, annotators should be experienced CSMs. For a product signal synthesis system, annotators should be experienced PMs. Generic annotators without domain knowledge produce lower-quality annotations.

**Minimum:** 2 annotators per item. Use a third annotator or adjudication session to resolve disagreements.

### Annotation Interface

Annotations should be captured in a structured format that is easy to review and version. A simple spreadsheet with defined columns works. Dedicated annotation tools (Label Studio, Argilla) improve quality at scale.

**Required fields per item:**
```
item_id | query | relevant_chunk_ids | gold_answer | gold_answer_properties | difficulty | query_category | edge_case_type | annotator_id | annotation_date | annotator_confidence | notes
```

### Inter-Annotator Agreement (IAA)

Measure IAA to assess annotation quality:
- **For binary relevance (relevant / not relevant):** Cohen's Kappa or Percent Agreement. Target: Kappa > 0.7
- **For quality scores (1–5 scales):** Weighted Kappa or Pearson correlation. Target: r > 0.7
- **For free-text gold answers:** Measure semantic similarity between annotators' answers (embedding cosine similarity). Target: >0.85

Low IAA on a category indicates ambiguous annotation criteria. Resolve by: (a) refining the annotation guideline for that category, (b) holding an adjudication session, (c) removing the ambiguous items from the dataset.

---

## Versioning

Every release of the golden dataset should be version-controlled:
- `v1.0.0`: Initial dataset (50 items, pre-launch)
- `v1.1.0`: Added edge cases from alpha feedback (75 items)
- `v2.0.0`: Major expansion with production log sampling (200 items)
- `v2.0.1`: Fixed annotation errors in 5 items

**Version control requirements:**
- Store dataset in a version-controlled repository (git with LFS for large datasets, or a dataset registry)
- Each evaluation run records which dataset version was used
- Older versions are retained for trend comparison

**Update triggers:**
- New use case added to the system → add coverage items for the new use case
- Novel failure mode discovered in production → add the failure mode to the dataset
- Significant corpus change (new data source added) → add queries that exercise the new source
- IAA drops below threshold → audit and revise annotation guidelines

---

## Coverage Audit

Conduct a coverage audit quarterly: review the dataset against the current use case coverage matrix and identify gaps. Add items for undercovered categories before the next major eval run.

**Coverage audit checklist:**
- [ ] All primary use cases have at least 5 items
- [ ] Edge case categories (adversarial, ambiguous, low-signal) have at least 10 items total
- [ ] Items from production failure reports added within 2 weeks of discovery
- [ ] Dataset version matches the most recent version in the eval pipeline configuration
- [ ] IAA measured and > threshold on the most recent annotation batch

---

*See also: [LLM-as-Judge Rubrics](llm-as-judge-rubrics.md) · [RAG Evaluation](/rag-workflow-documentation/rag-eval-plan.md) · [Eval Generation Prompts](/prompt-library/eval-generation-prompts.md)*
