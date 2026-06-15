# Evals Are the New Product Analytics

*For AI-native products, evaluation is the core instrumentation discipline — not an afterthought or an engineering task.*

---

## The Analogy

In the traditional SaaS era, the PM who could not read a Mixpanel dashboard or define a funnel conversion event was flying blind. Product decisions made without usage data were decisions made on intuition — and while intuition is valuable, it does not scale and it does not update in the face of evidence.

Product analytics became a core PM discipline. Not because PMs were data scientists, but because the decisions a PM makes — what to build, what to fix, when to declare something a success — required quantitative evidence to make defensibly.

Evaluation is the same shift for AI products.

An AI PM who cannot define a golden dataset, specify rubrics for LLM-as-judge evaluation, set acceptance rate targets, or interpret retrieval recall metrics is flying blind. The AI is generating outputs, users are accepting or overriding them, the model may be hallucinating in ways that users cannot detect — and none of this is visible to a PM who has not built evaluation into the product.

The analogy is not perfect — evaluation is more complex than product analytics, and requires different skills. But the strategic point is the same: it is the core instrumentation discipline for AI products, and the PM who defers it entirely to the engineering or ML team is not doing their job.

---

## Why Evals Are Different from Traditional QA

Traditional QA is deterministic. A button either works or it does not. A form either submits or it does not. A test either passes or it fails. Quality is binary.

AI quality is probabilistic and multidimensional. The model may produce a correct output 95% of the time, a plausible-but-wrong output 4% of the time, and a clearly wrong output 1% of the time. Whether "correct" means faithful to the sources, relevant to the query, appropriately formatted, free of hallucination, and safe from PII leakage — these are separate dimensions that can fail independently.

No single test covers all of this. Evaluation requires:
- A representative dataset of inputs and expected outputs (the golden dataset)
- Metrics for each quality dimension (faithfulness, relevance, citation accuracy, format compliance)
- A measurement process that samples and evaluates at appropriate cadence
- Comparison against a baseline to detect regression

This is more sophisticated than traditional QA — and it must be designed by the PM, not just implemented by engineering, because the PM is the one who knows what "good" looks like for the product's use cases.

---

## The Three Layers of Evaluation

### Layer 1: Offline Evaluation (Pre-Ship)

Before any AI feature ships, it should be evaluated on a golden dataset — a curated set of inputs with expected outputs and evaluation rubrics.

**What the PM owns:**
- Defining the scope of the golden dataset (how many examples? which use case categories?)
- Specifying evaluation dimensions and rubrics (what does faithful mean? what does a citation error look like?)
- Setting acceptance thresholds (what acceptance rate is good enough to ship?)
- Reviewing evaluation results and making the ship/no-ship call

The golden dataset is not a one-time artifact. It grows over time (as new use cases are discovered), is versioned (so you can compare performance across model or prompt changes), and is maintained as a living quality standard.

---

### Layer 2: Online Monitoring (Post-Ship)

After the feature ships, user behavior is the ground truth. Online metrics tell you whether the AI is performing well in the real world, not just on the golden dataset.

**Core online metrics:**
- **Acceptance rate:** What % of AI recommendations do users accept without modification? (Target: >75% for most use cases; <60% suggests low quality)
- **Override rate:** What % do users reject entirely? (Segmented by user type, account, action type — to identify where the AI consistently fails)
- **Edit rate:** What % do users accept after modification? (Measures quality ceiling: users want the output but have to fix it)
- **Task completion rate:** When the AI assists a task, is the task completed at a higher rate than without AI?
- **Time-to-action:** Does the AI reduce the time from "signal detected" to "action taken"?

These metrics are product analytics. They go in the PM's dashboard. They drive prioritisation decisions ("override rate for QBR drafts is 40% — this is the most important quality problem to fix in Q3").

---

### Layer 3: Human Feedback Loop (Ongoing)

Structured human feedback — not just thumbs up/down but specific rejection reasons, edited outputs, and expert annotation — provides the richest signal for understanding why the AI is failing and how to improve it.

**What the PM owns:**
- Designing the feedback collection interface (what rejection reasons to offer, how to capture edits)
- Defining when to trigger expert annotation (high-stakes output types, specific failure patterns)
- Ensuring feedback is routed to improvement processes (not just collected and forgotten)
- Measuring the delta: do eval metrics improve after feedback-driven prompt or dataset updates?

---

## Eval Design as a PRD Requirement

Every AI feature PRD should include an evaluation plan. At minimum:

**Offline eval section:**
- Golden dataset scope (size, category distribution, annotation protocol)
- Evaluation dimensions and rubrics
- Acceptance thresholds per dimension
- Baseline comparison methodology

**Online metrics section:**
- Primary online metric (usually acceptance rate or task completion rate)
- Segmentation: how will you disaggregate by user type, account tier, use case?
- Alert thresholds: at what metric level will you investigate?
- Review cadence: how often will you review online metrics?

**Human feedback section:**
- Feedback collection interface specification
- Annotation workflow for high-stakes output types
- Process for routing feedback to improvement

If the PRD does not include this section, the PM has not finished specifying the feature. The feature will ship into a measurement void — and when quality degrades, there will be no instrumentation to detect it until a customer complains.

---

## Evals as Competitive Intelligence

Your evaluation infrastructure tells you something your competitors don't know about your users. Every override, every structured rejection reason, every expert annotation is a data point about where the AI fails in your specific domain and for your specific users.

If you run a systematic evaluation program for 12 months and your competitor does not, you have made:
- More targeted prompt improvements (you know exactly which failure modes to fix)
- More informed golden dataset expansions (you know which edge cases are most common)
- Better model routing decisions (you know which task types are handled well by smaller models)

This compounds. The team with better evaluation discipline produces a better AI product over time, independent of model quality. It is the feedback flywheel that separates mature AI product teams from ones that are still guessing.

---

## The PM's Evaluation Vocabulary

To lead evaluation effectively, the PM must be able to use these terms precisely in conversations with engineers and ML teammates:

| Term | What It Means for the PM |
|---|---|
| Golden dataset | Curated test examples with expected outputs and rubrics; the PM defines scope and rubrics |
| Recall@K | Of all relevant documents in the corpus, how many did retrieval find in the top K? (Retrieval quality measure) |
| Faithfulness | Does the generated output only claim things supported by the retrieved context? (Hallucination measure) |
| Answer relevance | Is the generated output actually answering the question asked? |
| Acceptance rate | % of AI recommendations users act on without modification |
| Override rate | % of AI recommendations users reject |
| LLM-as-judge | Using a second LLM to evaluate the output of the first; requires calibration and rubric design |
| Regression test | Running the golden dataset on a new model or prompt to detect quality changes before shipping |

Knowing what these terms mean — and being able to use them in a PRD or in a product review — is table stakes for AI PM effectiveness.

---

## The Shift in PM Identity

PMs who built their identity around roadmapping, user research, and stakeholder management will find that AI products require a different kind of intellectual fluency. Not instead of those skills — in addition.

The PM who walks into a product review and says "our acceptance rate dropped from 78% to 64% after the last prompt change, and our retrieval recall on the domain-specific query category dropped from 0.82 to 0.71 — we think the issue is in the chunking update that went out last week" — that PM is doing their job.

The PM who walks in and says "the AI seems worse but I'm not sure why" has a measurement gap that needs to close.

Evals are how AI-native PMs know whether the product is working. Building that measurement discipline is not optional — it is the job.

---

*Related: [Online AI Product Metrics](../ai-evaluation-frameworks/online-ai-product-metrics.md) · [Golden Dataset Design](../ai-evaluation-frameworks/golden-dataset-design.md) · [LLM-as-Judge Rubrics](../ai-evaluation-frameworks/llm-as-judge-rubrics.md) · [How AI Agents Change SaaS PM](how-ai-agents-change-saas-pm.md)*
