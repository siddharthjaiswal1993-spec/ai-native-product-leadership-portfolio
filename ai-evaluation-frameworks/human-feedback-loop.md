# Human Feedback Loop

**Designing the human feedback loop: collection, data quality, preference pairs, and connecting feedback to model improvement**

---

## Why the Feedback Loop Is the Product's Compounding Mechanism

An AI product without a human feedback loop is a static product. It performs at its launch quality and degrades gradually as the world changes and user expectations evolve.

An AI product with a well-designed human feedback loop compounds. Every rejection teaches the system something. Every edit creates a preference pair. Every approval confirms what is working. Over time, the system gets better at predicting what each user team will accept — and that predictive accuracy is a switching cost that competitors cannot easily replicate.

The feedback loop is not a "nice to have." It is the mechanism that determines whether the AI product creates lasting value or decays.

---

## Feedback Collection Points

### 1. Thumbs Up / Thumbs Down

The simplest feedback signal. Binary: the user signals that the output was good or not good.

**What it measures:** Global output satisfaction. The weakest form of feedback — tells you there is a problem but not where or why.

**Best for:** Measuring overall output quality trend over time. A declining thumbs-up rate is an early warning signal.

**Design notes:**
- Place the thumbs up/down control after the user has read the output, not before
- Do not ask for thumbs up/down before the user has had a chance to use the output (they may not yet know if it was good)
- Optional one-line text box after thumbs down: "What was wrong?" Increases signal quality significantly without much friction

---

### 2. Structured Rejection Reasons

When a user dismisses, rejects, or substantially overrides an AI output, capture why.

**Rejection reason taxonomy (customise per use case):**

| Category | Examples |
|---|---|
| Factual error | "The data cited is wrong", "This metric is from the wrong account" |
| Missing information | "Didn't include the most important issue", "Missing the renewal date" |
| Wrong tone | "Too formal", "Too casual for this customer" |
| Not actionable | "Too vague to act on", "Doesn't say what to actually do" |
| Scope error | "Answered the wrong question", "Mixed up two accounts" |
| Citation issue | "Citation doesn't support the claim", "Cited the wrong source" |
| Hallucination | "This claim is fabricated / not true" |
| Format issue | "Wrong structure", "Too long", "Missing required section" |

**Design notes:**
- Limit to 5–8 options maximum — too many options reduces completion rate
- Allow multi-select (an output can fail for multiple reasons)
- An "other" option with a free text field captures feedback not covered by the taxonomy
- Make rejection reason capture mandatory for HITL approval rejections (high-quality signal, high enough stakes to require annotation)
- Make it optional for casual feedback (the friction is too high to require for every dismissal)

---

### 3. Section-Level Approval and Editing

For multi-section outputs (PRDs, QBRs, reports), track which sections are approved and which are edited.

**What it measures:** Section-level quality. Which parts of the generated output are reliably good vs. reliably needing human correction?

**How to use it:**
- Section edit rate by section type → identifies which prompts or retrieval strategies need improvement
- Edit magnitude (character-level edit distance) → distinguishes minor polishing from substantial rewrites
- Longitudinal trend: is the edit rate for a specific section type declining over time? If yes, the system is improving on that dimension.

---

### 4. Preference Pairs

When a user edits an AI-generated output and approves the edited version, you have a preference pair: (original, edited) with a signal that the edited version is better.

**Preference pairs are the highest-quality feedback signal.** They can be used to:
- Fine-tune the generation model on your domain and user preferences
- Improve prompt quality (by analysing patterns in what gets edited)
- Calibrate the LLM-as-judge (the judge should prefer the "edited" version over the "original")

**Data schema for preference pairs:**
```json
{
  "pair_id": "uuid",
  "session_id": "uuid",
  "user_id": "hashed (not raw)",
  "task_type": "string",
  "input": "string (the query or task)",
  "context": "string (the retrieved context, if applicable)",
  "original_output": "string",
  "preferred_output": "string (the human-edited version)",
  "rejection_reasons": ["string"],
  "edit_magnitude": "number (character edit distance)",
  "timestamp": "ISO 8601",
  "annotator_type": "end_user | expert_reviewer",
  "consent_given": "boolean"
}
```

**Data quality requirements:**
- Only capture preference pairs where the user explicitly approved the edited version — do not infer preferences from passive behavior
- Capture the full context (what was retrieved) alongside the preference pair — the preferred output may only be correct given specific context
- Exclude pairs where `edit_magnitude` is below a minimum threshold (trivial edits like fixing a typo are not meaningful preference signals)

---

### 5. Expert Annotation Sessions

Periodic structured feedback sessions with domain expert users (not just AI product users) who review a sample of AI outputs against a quality rubric.

**Format:** 
- 30–60 minute session with 2–3 domain experts
- Review 10–20 AI-generated outputs from the last month's production sample
- Rate each output on the quality dimensions from the eval framework
- Discuss edge cases and failure modes observed in practice

**Why expert annotation matters beyond user feedback:**
- End users often do not know what good looks like in a domain they are not expert in
- Expert annotators can identify subtle quality issues (hallucinated claims, missing context) that end users would accept
- Expert annotation provides a calibration anchor for the LLM-as-judge

---

## Feedback Data Quality

High-volume feedback is only valuable if the data is high quality. Common feedback data quality issues:

**Feedback fatigue:** Users stop providing structured feedback after the first few sessions. Mitigate by: making feedback frictionless (2 taps maximum for basic feedback), rotating feedback prompts to avoid habituation, focusing structured feedback collection on the highest-leverage outputs.

**Selection bias:** Users who provide feedback may not be representative of all users. Power users and early adopters provide more feedback. Users who are frustrated but disengaged provide none. Monitor feedback completion rate by user cohort.

**Inconsistency:** The same output rated differently by different users due to different context, preferences, or expertise. Mitigate by: structuring rejection reasons (reduces inconsistency in categorisation), calibrating with expert annotation sessions.

**Consent and PII:** Feedback data may include customer-sensitive information in the "free text" fields. Apply PII detection and redaction to all free-text feedback before storage. Collect explicit consent for using feedback to improve the system.

---

## Connecting Feedback to Model Improvement

Feedback data creates value at three levels:

**Level 1: Prompt improvement (immediate, low cost):**
- Analyse rejection reason patterns → identify which prompt instructions are being violated
- Review high-edit-rate sections → identify which generation prompts produce the most-edited content
- Update the prompt and run regression tests before deploying

**Level 2: Retrieval improvement (medium term):**
- Analyse rejections tagged "missing information" or "wrong account" → often indicates retrieval failures
- Add the failure cases to the golden retrieval dataset
- Tune retrieval parameters or metadata filters

**Level 3: Model fine-tuning (long term, high cost):**
- Accumulated preference pairs can be used to fine-tune the generation model on your domain
- Only worthwhile after accumulating hundreds of high-quality preference pairs
- Requires ML engineering investment; PM should define the quality threshold before initiating

---

## Feedback Loop Governance

**User consent:** Every user must consent to their feedback being used for system improvement. This is a product requirement with a legal component. Consent should be explicit (not buried in terms of service), reversible, and granular (user can consent to feedback logging but not fine-tuning).

**Data retention:** Define how long feedback data is retained. Preference pairs used for fine-tuning may need to be retained indefinitely. Raw user feedback should have a defined retention period.

**Access control:** Feedback data is sensitive (it contains human-generated content about customer interactions). Access should be limited to the product and ML team, not available to all employees.

**Feedback attribution:** Never use specific user feedback (preference pairs, rejection reasons) to evaluate individual user performance. Feedback data is an aggregate quality signal, not a performance management tool.

---

*See also: [Online AI Product Metrics](online-ai-product-metrics.md) · [LLM-as-Judge Rubrics](llm-as-judge-rubrics.md) · [Evals Are the New Product Analytics](/strategy-memos/evals-are-the-new-product-analytics.md)*
