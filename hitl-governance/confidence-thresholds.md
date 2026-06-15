# Confidence Thresholds

**Setting and calibrating confidence thresholds for AI-governed actions in enterprise products**

---

## What Confidence Means in Practice

When an AI system reports a confidence score of 0.85 for a finding or recommendation, what does that mean? In most deployed AI systems, the honest answer is: it depends on how the confidence was calculated, whether the confidence has been calibrated, and whether 0.85 confidence is appropriate for the decision context.

Confidence scores are not inherently meaningful. They become meaningful when:
1. They are calculated from a consistent, documented methodology
2. They are calibrated against actual outcomes (a system that says "0.85 confidence" should be correct about 85% of the time — not 60% and not 99%)
3. The thresholds that determine routing decisions (approve vs. review vs. escalate) are set based on the stakes of each action type, not on an arbitrary default

---

## Sources of Confidence

### Model-Reported Confidence (Softmax Probabilities)

For classification tasks, the model reports a probability distribution over classes. The highest probability class is selected as the output; that probability is used as the "confidence."

**Problem:** Softmax probabilities are not calibrated confidences. A model with 0.90 softmax probability for a class may only be correct 70% of the time on your domain. Overconfident models (common with large LLMs fine-tuned on specific tasks) consistently report high confidence even when wrong.

**Mitigation:** Calibrate the model's reported probabilities against your golden evaluation set using Platt scaling or temperature scaling.

---

### Retrieval-Based Confidence

Confidence derived from the quality of retrieved evidence:
- **Number of supporting chunks:** More independent sources → higher confidence
- **Reranker score of top chunk:** Higher relevance score → higher confidence
- **Recency of evidence:** More recent sources → higher confidence for time-sensitive claims
- **Source authority:** Higher-authority sources → higher confidence

**Advantage:** Interpretable — the confidence score reflects the evidence base, which the human reviewer can also see.

**Formula example:**
```
confidence = w1 × (citations_count / max_expected_citations)
           + w2 × (avg_reranker_score / 1.0)
           + w3 × (recency_score)
           + w4 × (authority_score)
```

Where w1 + w2 + w3 + w4 = 1.0 (configurable weights per use case).

---

### Rule-Based Confidence

For specific, well-defined signal combinations, confidence can be determined by rule rather than by model probability:
- If the account has 3+ independent churn signals → confidence HIGH (0.85)
- If the account has 1 churn signal and no expansion signals → confidence MEDIUM (0.65)
- If the account has 0 churn signals → confidence LOW (0.20) or NOT_APPLICABLE

Rule-based confidence is more interpretable and more reliable than model-reported probabilities for well-understood patterns, but it does not generalise to novel patterns.

---

## Calibration Methodology

Calibration answers the question: when the system reports 80% confidence, is it actually right 80% of the time?

### Calibration Dataset

To calibrate, you need a dataset of (input, predicted_confidence, actual_outcome) triples:
1. Run the system on a held-out evaluation set
2. Record the confidence score for each output
3. Have human experts label each output as correct / incorrect
4. Plot reliability diagrams: bucket outputs by confidence (0-0.1, 0.1-0.2, ..., 0.9-1.0); for each bucket, calculate the actual accuracy rate

### Expected Calibration Error (ECE)

ECE = Σ (|bucket_confidence - bucket_accuracy| × bucket_weight)

A perfectly calibrated system has ECE = 0. A system that consistently overestimates confidence has ECE > 0 with accuracy < confidence in each bucket.

**Target:** ECE < 0.10 for production AI systems making consequential decisions.

### Recalibration

If ECE is too high (overconfident system), apply post-hoc calibration:
- **Temperature scaling:** Divide logits by a temperature parameter T > 1 to flatten the distribution (reduces overconfidence)
- **Platt scaling:** Fit a logistic regression model to map raw model probabilities to calibrated probabilities

### Recalibration Schedule

Run calibration assessment quarterly, or whenever:
- The underlying model is updated
- The retrieval configuration changes significantly
- A significant distribution shift is detected in production inputs

---

## Setting Thresholds

Thresholds determine which confidence level triggers which routing decision. They should be set based on the stakes of each action type, not based on an arbitrary universal default.

### Threshold Design Framework

For each action type, answer:

**1. What is the cost of a false positive?**
A false positive (high confidence, wrong recommendation) that leads to an approved action. What is the worst-case outcome?

**2. What is the cost of a false negative?**
A false negative (low confidence, correct recommendation rejected or escalated to human). What is the cost of the additional human review time?

**3. What is the acceptable false positive rate?**
Given the cost of a false positive, what fraction of high-confidence recommendations can be wrong before this is unacceptable?

**Example: Expansion Brief Routing**
- False positive cost: CSM wastes 20 minutes reviewing an expansion brief for an account that is not actually ready to expand. Low cost.
- False negative cost: A real expansion opportunity is missed because confidence was too low to surface. Medium cost (missed revenue).
- Acceptable false positive rate: High — 30% false positive rate is acceptable for expansion briefs because the cost of a false positive is low and the benefit of catching a real expansion is high.
- Threshold: SURFACE if confidence ≥ 0.50

**Example: Churn Risk Alert Routing**
- False positive cost: CSM spends time on an account that is not actually at risk. Medium cost.
- False negative cost: A real churn risk is missed; account churns. High cost.
- Acceptable false positive rate: Medium — 20% false positive rate is acceptable; we prefer more alerts to fewer missed churns.
- Threshold: SURFACE if confidence ≥ 0.65

**Example: Autonomous Action Execution (Level 4)**
- False positive cost: Wrong action taken autonomously; potentially irreversible. Very high cost.
- Acceptable false positive rate: Very low — <2%.
- Threshold: EXECUTE AUTONOMOUSLY only if confidence ≥ 0.92

---

## Threshold Configuration Table (Illustrative)

| Action Type | SURFACE Threshold | REQUIRE APPROVAL Threshold | ESCALATE (vs. dismiss) Threshold |
|---|---|---|---|
| Expansion brief | ≥0.50 | All expansion briefs | — |
| Churn risk flag | ≥0.65 | HIGH: always; MEDIUM: ≥0.75 | HIGH + confidence <0.80 |
| Follow-up email draft | Always draft | Always require approval | — |
| QBR section draft | Always draft | Always require approval | — |
| Auto-CRM note (Level 3) | — | ≥0.88 | <0.88 → require approval |
| Autonomous data update (Level 4) | — | ≥0.95 | <0.95 → require approval |

---

## Threshold Monitoring and Adjustment

Thresholds are not set-and-forget. Monitor:

**Precision at threshold:** Of items surfaced above the threshold, what fraction are confirmed correct by human reviewers?

**Recall at threshold:** Of items that are confirmed correct (by post-hoc human review), what fraction were above the threshold and surfaced?

**Threshold drift:** If the model or retrieval pipeline changes, the confidence distribution may shift — a threshold calibrated for the old model may be wrong for the new model.

**Adjustment process:**
1. Run a calibration assessment after any model or retrieval change
2. Check precision and recall at the current threshold
3. If precision is too low (too many incorrect high-confidence outputs): raise the threshold
4. If recall is too low (too many correct outputs being suppressed): lower the threshold
5. Document the change and run a regression test before deploying the new threshold

---

*See also: [Human-in-the-Loop Design](human-in-the-loop-design.md) · [Escalation Logic](escalation-logic.md) · [Hallucination Control](/risk-guardrails/hallucination-control.md)*
