# Hallucination Control

**Types, detection methods, mitigation patterns, and monitoring approaches for enterprise AI products**

---

## Types of Hallucination

Not all hallucination is the same. Understanding the type is necessary for applying the right mitigation.

### Type 1: Factual Hallucination (Fabricated Facts)

The model asserts something that is false and not grounded in the retrieved context. It may sound plausible and authoritative.

**Example:** "Customer Acme Corp's renewal date is March 15, 2026." The retrieved context does not contain the renewal date. The model has generated a plausible-sounding date.

**Risk level in enterprise context:** High — any fabricated customer fact that is acted upon can cause relationship damage or business loss.

---

### Type 2: Attribution Hallucination (Wrong Source Cited)

The model cites a source that does not support the claimed fact. The citation looks legitimate but either does not exist or does not contain the claimed information.

**Example:** "Support ticket volume increased 40% [ZD-4521]." The cited ticket ZD-4521 does not contain volume statistics.

**Risk level:** High — the citation provides false confidence that the claim is grounded.

---

### Type 3: Confabulation (Coherent But Inaccurate Synthesis)

The model synthesises information from multiple retrieved sources in a way that is internally coherent but factually inaccurate — typically by extrapolating beyond what the sources actually say.

**Example:** Two retrieved sources mention a customer "exploring expansion to the EMEA market" and "increasing headcount." The model synthesises: "Customer is planning to launch EMEA operations in Q3 2026." Neither source stated a timeline.

**Risk level:** Medium — the output is plausible but contains inferred specifics that are not in the data.

---

### Type 4: Hallucination Under Uncertainty

When the model lacks sufficient context to answer a question, it generates a plausible answer rather than saying "I don't know." This is the most common and most dangerous type in RAG systems.

**Example:** Query: "What was discussed in the last executive call with Acme Corp?" No transcript is available in the retrieved context. Model generates: "The last executive call focused on the product roadmap and the upcoming renewal."

**Risk level:** High — the model appears to be answering from data when it is not.

---

## Detection Methods

### Method 1: Citation Validation (Automated)

For every factual claim in a generated output, validate that the cited source contains information that supports the claim. Run this as a post-generation step (Citation Agent).

**Implementation:**
- Parse claims and citation markers from the generated output
- For each cited claim: retrieve the source chunk, run a lightweight LLM judge: "Does this source text support this claim? Yes/No"
- Reject any output where >5% of claims are unsupported
- Log all unsupported claims for human review

**What this catches:** Attribution hallucination and factual hallucination where the model cites incorrectly. Does not catch confabulation where the model cites correctly but extrapolates.

---

### Method 2: Claim Extraction and Verification (Automated + Periodic Human)

Extract all factual claims (entities, numbers, dates, names, percentages) from a sample of generated outputs. Verify each claim against the source data independently.

**Automated:** Use a claim extraction model to identify and type all factual claims (date, metric, entity, causal statement). Cross-reference extracted claims against the indexed corpus.

**Human sampling:** Monthly review by a domain expert of 20–30 outputs, manually verifying the 3–5 most specific claims in each output against the source data.

---

### Method 3: Self-Consistency Checking

For important outputs, generate the same response multiple times with slightly different prompts or different random seeds. Compare the outputs for consistency. High variance in factual claims across multiple generations indicates low confidence.

**Use case:** Appropriate for high-stakes outputs (QBR sections, risk assessments) where the additional inference cost is justified by the output quality assurance value.

---

### Method 4: Confidence Threshold Gating

Generate a confidence score for each output based on the quality of the retrieval evidence (number of supporting sources, reranker scores). Surface outputs below the threshold only to human reviewers, not directly to end users.

**See:** [Confidence Thresholds](/hitl-governance/confidence-thresholds.md) for calibration methodology.

---

## Mitigation Patterns

### Pattern 1: Retrieval-First Architecture

**Principle:** The model should never be expected to provide facts from memory. All factual claims should come from the retrieved context. This is enforced by the system prompt: "Only use information from the provided context. Do not use your training knowledge."

**Effectiveness:** High for known facts in the corpus. Not effective for facts not in the corpus (where the model either correctly says "I don't know" or hallucinates).

---

### Pattern 2: Citation Enforcement

**Principle:** Every factual claim requires an inline citation to a retrieved source. Claims without citations are either removed or flagged as "[UNVERIFIED]." The Citation Agent validates all citations before output delivery.

**Effectiveness:** High for attribution hallucination. Partial for confabulation (the model may cite correctly while still extrapolating).

---

### Pattern 3: Explicit "I Don't Know" Instruction

**Principle:** The system prompt explicitly instructs the model to return "I don't have sufficient information to answer this" rather than generating a plausible response when the retrieved context is insufficient.

**System prompt language:** "If the provided context does not contain sufficient information to answer the question fully, say: 'I do not have sufficient information to answer this from the available data. The context available covers [describe what is available] but does not include [describe what is missing].' Do not generate plausible-sounding answers to fill gaps in the context."

**Effectiveness:** Moderate to high, depending on model instruction following quality. Test with adversarial inputs.

---

### Pattern 4: Output Comparison Against Source

**Principle:** For structured outputs (with specific fields), validate each field's value against the source data programmatically where possible.

**Examples:**
- Generated output contains account_renewal_date: "2026-03-15" → query the CRM API directly for this account's renewal date and compare
- Generated output contains health_score: "72" → compare against the health score computed by the health scoring module

**Effectiveness:** High for structured outputs with verifiable fields. Not applicable for generative text.

---

### Pattern 5: Human Review as the Final Mitigation

**Principle:** For all customer-facing outputs, require a human reviewer who has the domain expertise to recognise a hallucinated claim.

**Effectiveness:** High if reviewers have the appropriate expertise and are reviewing carefully. Low if reviewers approve without reading. Monitor approval time per item to detect rubber-stamping.

---

## Monitoring Approach

**Monthly automated sampling:**
- Sample 30 outputs from production
- Run citation validation on each
- Report: citation accuracy rate, citation failure type breakdown (no citation, wrong citation, partially supported)
- Compare to prior month's baseline — trend is more important than absolute value

**Alert thresholds:**
- Citation accuracy drops below 90% → immediate investigation
- Two consecutive months below 92% → prompt or model review required

**Annual adversarial test:**
- Run the system against a set of adversarial inputs designed to elicit hallucination (queries with no relevant data in the corpus, queries with contradictory data, queries about entities that don't exist in the data)
- Verify that the system correctly returns "I don't have information" rather than hallucinating

---

*See also: [AI Risk Register](ai-risk-register.md) · [Reranking and Citation Grounding](/rag-workflow-documentation/reranking-citation-grounding.md) · [RAG Failure Modes](/rag-workflow-documentation/rag-failure-modes.md)*
