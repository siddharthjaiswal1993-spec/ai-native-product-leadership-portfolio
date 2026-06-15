# LLM-as-Judge Rubrics

**Designing, calibrating, and safely using LLM-as-judge for AI product quality evaluation**

---

## What LLM-as-Judge Is

LLM-as-judge is an evaluation technique where a language model is used as an automated evaluator to assess the quality of another model's output. Given a question, a context (retrieved documents), and a generated response, the judge model rates quality on one or more dimensions.

LLM-as-judge enables scalable automated evaluation that would otherwise require expensive and slow human annotation. It is particularly useful for:
- Monthly quality sweeps across a large production sample
- Regression testing on generation quality after prompt or model changes
- Evaluating dimensions that are hard to automate with rule-based checks (relevance, tone, completeness)

The critical discipline: LLM-as-judge is a tool for scaling evaluation, not a replacement for human judgment. Judge results must be calibrated against human annotation and interpreted with appropriate scepticism.

---

## Rubric Design Principles

### Principle 1: One Rubric Per Dimension

Do not ask the judge to evaluate multiple quality dimensions in a single score. Faithfulness and relevance are different dimensions — a response can be perfectly faithful to the retrieved context while being irrelevant to the question.

Create a separate rubric for each dimension: faithfulness, answer relevance, citation accuracy, completeness, format compliance, tone appropriateness.

### Principle 2: Rubrics Must Be Binary or Calibrated Scale

**Binary rubrics (recommended for high-stakes dimensions):**
- Faithfulness per claim: supported / not_supported / partially_supported
- Citation accuracy: correct / incorrect / partially_correct
- Schema compliance: valid / invalid

**Calibrated scale rubrics (recommended for holistic quality dimensions):**
- Answer relevance: 1–5 scale with explicit anchors
- Completeness: 1–5 scale with explicit anchors

**Avoid vague scales** like "rate from 1 to 10." Without explicit anchors, the judge will make inconsistent interpretations of what 7 vs. 8 means.

### Principle 3: Explicit Anchors

Every scale point must have an explicit description of what qualifies. Do not rely on the judge to interpret "3 out of 5" without guidance.

**Example: Answer Relevance Rubric**
```
5 — Fully answers the question. All key aspects of the question are addressed. 
    No significant gaps or tangential information.
4 — Mostly answers the question with minor gaps. The user would likely get what they need, 
    but one aspect is partially addressed or slightly underdeveloped.
3 — Partially answers the question. The main point is addressed but significant aspects 
    are missing or the response is too generic to be immediately actionable.
2 — Marginally relevant. Addresses a related topic but does not actually answer the question asked.
1 — Not relevant. Does not address the question at all, or is entirely off-topic.
```

### Principle 4: Chain-of-Thought Before Score

Require the judge to reason before scoring. "Think step by step before giving your score." This improves scoring consistency and makes the judge's reasoning inspectable.

### Principle 5: Don't Use the Same Model for Generation and Judgment

A model evaluating its own outputs is biased toward rating its outputs more favourably. Use a different model (or at minimum a different prompt configuration) as the judge from the model used for generation.

---

## Core Rubrics

### Rubric 1: Faithfulness (Per-Claim)

**What it measures:** Is each factual claim in the output supported by the retrieved context?

**Judge prompt:**
```
You are evaluating whether a specific claim from an AI-generated answer is 
supported by the provided source documents.

CLAIM: {{CLAIM_TEXT}}

SOURCE DOCUMENTS:
{{RETRIEVED_CONTEXT}}

Instructions:
1. Read the claim carefully.
2. Search the source documents for text that supports or contradicts this claim.
3. Think step by step before scoring.

Rate the claim as one of:
- SUPPORTED: The source documents clearly and directly support this claim.
- PARTIALLY_SUPPORTED: The source documents provide some evidence for this claim 
  but do not fully confirm it.
- NOT_SUPPORTED: The source documents do not contain evidence for this claim.
- CONTRADICTED: The source documents explicitly contradict this claim.

Return:
{
  "claim": "string",
  "verdict": "SUPPORTED | PARTIALLY_SUPPORTED | NOT_SUPPORTED | CONTRADICTED",
  "supporting_text": "verbatim text from source documents (or null if not supported)",
  "reasoning": "string"
}
```

**Faithfulness score calculation:** (SUPPORTED claims) / (total claims) × 100%

**Target:** >95% faithfulness rate for citation-enforced outputs

---

### Rubric 2: Answer Relevance

**What it measures:** Does the output actually answer the user's question?

**Judge prompt:**
```
You are evaluating whether an AI-generated answer relevantly addresses the user's question.

USER QUESTION: {{QUESTION}}

AI ANSWER: {{ANSWER}}

Instructions:
1. Read the question carefully and identify what the user is asking for.
2. Read the answer and assess the degree to which it addresses what was asked.
3. Think step by step.
4. Rate from 1 to 5 using the scale below.

Scale:
5: Fully answers the question with no significant gaps
4: Mostly answers with minor gaps; user gets what they need
3: Partially answers; main point addressed but aspects missing
2: Marginally relevant; addresses a related but different topic
1: Not relevant; does not address the question

Return:
{
  "score": number (1-5),
  "rationale": "string (specific explanation citing aspects of the question that were/were not addressed)",
  "gaps": ["string (specific aspects not addressed)"]
}
```

**Target:** Average score >4.0 on production sample

---

### Rubric 3: Citation Accuracy

**What it measures:** Does a cited source actually contain information supporting the cited claim?

**Judge prompt:**
```
Evaluate whether the cited source supports the claim that cites it.

CLAIM: {{CLAIM_TEXT}}
CITATION ID: {{CITATION_ID}}
CITED SOURCE TEXT: {{CITATION_SOURCE_TEXT}}

Does the cited source text contain information that supports this claim?

Return:
{
  "verdict": "SUPPORTED | PARTIALLY_SUPPORTED | NOT_SUPPORTED",
  "relevant_excerpt": "string (the specific part of the source that supports/does not support the claim)",
  "reasoning": "string"
}
```

**Target:** >95% of citations correctly support their claims

---

### Rubric 4: Completeness

**What it measures:** Does the output include all the information the user would reasonably need?

**Judge prompt:**
```
Evaluate the completeness of an AI-generated answer relative to the question and available context.

USER QUESTION: {{QUESTION}}
RETRIEVED CONTEXT (summary): {{CONTEXT_SUMMARY}}
AI ANSWER: {{ANSWER}}

Instructions:
1. Based on the user's question, what information would a complete answer include?
2. Identify what the AI answer covers and what it does not.
3. Consider whether the gaps are due to missing context or missing coverage of available context.

Rate from 1 to 5:
5: Complete — covers all reasonably expected aspects given available context
4: Mostly complete — one minor gap that does not significantly affect usefulness
3: Partially complete — 2-3 meaningful gaps but core information present
2: Incomplete — key information is missing
1: Substantially incomplete — most expected content is absent

Return:
{
  "score": number (1-5),
  "present_elements": ["string (what the answer covers)"],
  "missing_elements": ["string (what was missing)"],
  "missing_in_context": "boolean (were the missing elements absent from the context too?)"
}
```

---

### Rubric 5: Format Compliance

**What it measures:** Does the output conform to the required output schema?

**Note:** This is better evaluated with a schema validator than with an LLM judge. Use the LLM only for semi-structured format compliance that cannot be easily schema-validated (e.g., "does the response include at least one recommended action in the second paragraph?").

```
Evaluate whether the AI response conforms to the required format.

REQUIRED FORMAT: {{FORMAT_REQUIREMENTS}}
AI RESPONSE: {{RESPONSE}}

Return:
{
  "compliant": "boolean",
  "violations": ["string (each format requirement that was violated)"],
  "partial_compliance": "boolean (true if mostly compliant with minor violations)"
}
```

---

## Calibration Against Human Judgment

Before using LLM-as-judge in production, calibrate:

**Step 1:** Select 50 items from the golden dataset with known human-annotated quality scores.

**Step 2:** Run the LLM judge on the same 50 items.

**Step 3:** Calculate agreement:
- For continuous scores (1–5): Pearson correlation and mean absolute error
- For categorical verdicts (SUPPORTED/NOT_SUPPORTED): Cohen's Kappa, precision, recall

**Acceptable calibration thresholds:**
- Continuous scale: Pearson r > 0.75, MAE < 0.8
- Categorical: Cohen's Kappa > 0.65

**If calibration fails:** Review judge prompt. Common issues:
- Ambiguous scale anchors
- Judge misinterpreting the task
- Distribution mismatch between calibration set and production distribution

---

## Limitations and Anti-Patterns

**Limitation: Position bias.** LLM judges tend to rate responses that appear first or that are longer more favourably. Mitigate by: randomising the position of evaluated outputs, using both short and long gold answers in calibration.

**Limitation: Self-enhancement bias.** Models favour their own outputs. Mitigate by: using a different model as judge than the one generating outputs.

**Limitation: Sycophancy.** Some judge models agree with claimed authority or hedge in ways that produce uninformative scores. Mitigate by: using a rubric that requires the judge to identify specific evidence for its verdict.

**Anti-pattern: Single-score holistic rating.** Asking the judge to rate "overall quality" produces noisy, uninterpretable scores. Always rate specific, named dimensions.

**Anti-pattern: Trusting judge scores without calibration.** LLM-as-judge scores are only meaningful if calibrated against human judgment. Never use uncalibrated judge scores as quality gates.

---

*See also: [Golden Dataset Design](golden-dataset-design.md) · [RAG Evaluation](/ai-evaluation-frameworks/rag-evaluation.md) · [Online AI Product Metrics](online-ai-product-metrics.md)*
