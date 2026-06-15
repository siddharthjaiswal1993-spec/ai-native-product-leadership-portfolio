# Model Routing Strategy

**Routing different tasks to different models: cost/quality tradeoffs and routing logic**

---

## The Core Insight: Not All Tasks Need the Same Model

Frontier large language models are capable of impressive reasoning, nuanced synthesis, and high-quality text generation. They are also expensive and relatively slow.

Many tasks in an AI product do not require frontier model capabilities. They require reliable structured extraction, short-context Q&A, schema validation, or classification — tasks that smaller, faster, cheaper models handle at comparable or acceptable quality.

Model routing is the practice of matching task complexity to model capability. The goal is not to minimise cost at the expense of quality — it is to ensure you are not paying frontier model prices for tasks that do not need frontier capabilities.

---

## Task Taxonomy by Model Requirement

### Requires Frontier Model (Large, High Quality)

These tasks involve complex multi-step reasoning, long-context synthesis, or high-quality creative generation where quality degradation from a smaller model would be visible and consequential.

| Task Type | Example | Why Frontier |
|---|---|---|
| Multi-source synthesis | Pre-visit brief from 5 signal sources | Requires coherent synthesis across many inputs |
| Complex root cause analysis | Churn risk with competing hypotheses | Requires nuanced causal reasoning |
| QBR section generation | Goals vs. actuals with narrative | Requires high-quality business writing |
| Opportunity brief writing | PRD opportunity synthesis | PM will read every word; quality matters |
| Adversarial input handling | Responding to edge cases and exceptions | Small models fail more often on unusual inputs |

---

### Suitable for Mid-Tier Model (Medium Quality, Lower Cost)

These tasks require good language understanding and structured generation but not the full capability of a frontier model. A 3-7 billion parameter model or a cost-optimised API tier handles these well.

| Task Type | Example | Why Mid-Tier |
|---|---|---|
| Short Q&A with retrieved context | In-visit location Q&A | Context provided; answer is short; quality acceptable at mid-tier |
| Structured extraction | Action item extraction from transcript | Well-defined extraction task; schema-validated output |
| Classification | Signal type classification at ingestion | Classification task; binary or small category space |
| Routine email drafting | Standard post-meeting follow-up | Template-like task; variation is low |
| Document summarisation | Summarising a support ticket thread | Single document, clear task |

---

### Suitable for Small/Fast Model (Speed-Optimised, Lowest Cost)

Tasks where latency is critical and quality requirements are met by even lightweight models.

| Task Type | Example | Why Small |
|---|---|---|
| Schema validation | Checking JSON output format | Rule-based; no language understanding needed |
| Sentiment classification | Positive/negative/neutral on a review | Binary or ternary classification |
| Simple regex or extraction | Extracting dates, emails, ticket IDs | Often better handled by non-LLM approaches |
| Citation ID extraction | Parsing inline citation markers | Regex is better than LLM for this |

---

## Routing Logic Design

### Static Routing (Simple, Low-Overhead)

Assign a fixed model tier to each task type at configuration time. No runtime decision-making required.

**Configuration example:**
```yaml
task_routing:
  pre_visit_brief_generation: frontier_model
  in_visit_qa: mid_tier_model
  action_item_extraction: mid_tier_model
  citation_validation: mid_tier_model
  signal_classification: small_model
  schema_validation: rule_based (no LLM)
```

**When to use:** When task types are well-defined and the quality/cost tradeoff is clear and stable.

---

### Dynamic Routing (Context-Dependent)

Route to a higher-quality model based on runtime signals: context length, confidence threshold, customer tier, or feature flag.

**Example: escalate to frontier model if:**
- Input context exceeds 3,000 tokens (complexity threshold)
- The query is about an Enterprise-tier customer (higher stakes)
- A prior generation attempt with the mid-tier model returned a low confidence score
- A feature flag is set for a specific customer who has requested premium quality

**Example routing function (pseudocode):**
```python
def route_model(task_type, context_tokens, customer_tier, confidence_required):
    base_model = TASK_ROUTING[task_type]  # from static config
    
    if context_tokens > 3000:
        return FRONTIER_MODEL  # escalate for long context
    
    if customer_tier == "ENTERPRISE" and task_type in HIGH_STAKES_TASKS:
        return FRONTIER_MODEL  # escalate for high-stakes enterprise
    
    if confidence_required > 0.85 and base_model != FRONTIER_MODEL:
        return FRONTIER_MODEL  # escalate for high-confidence requirement
    
    return base_model  # use static routing as default
```

---

### Cascade Routing (Quality Gated)

Run the task with the small model first. If the output meets quality criteria, use it. If not, escalate to a larger model.

**Example:**
1. Run citation validation with mid-tier model
2. If validation reports >1 failed citation, re-run brief generation with frontier model using enhanced citation enforcement
3. If the enhanced generation passes citation validation, use it; otherwise flag for human review

**When to use:** When most tasks are handled well by the small model but a minority require escalation. Reduces average cost while maintaining quality floor.

**Latency consideration:** Cascade routing adds latency for tasks that escalate (two model calls instead of one). Only appropriate for batch or async workflows, not real-time interactive tasks.

---

## Cost vs. Quality Tradeoff Analysis

| Model Tier | Relative Cost | Relative Quality | Best Use |
|---|---|---|---|
| Frontier (synthetic $15/$60 per 1M) | 100% | 100% | Complex synthesis, high-stakes outputs |
| Mid-tier (synthetic $3/$12 per 1M) | 20% | 85–90% | Short extraction, classification, Q&A |
| Small (synthetic $0.50/$2 per 1M) | 3% | 70–80% | Classification, regex-equivalents, batch pre-processing |

*Relative quality estimates are illustrative. Actual quality depends on the specific model and task. Always run your own quality evaluation before changing the routing configuration.*

---

## Routing Decision Checklist

When deciding which model tier to route a task to, answer:

1. **What is the output quality requirement?** Will users see this output directly? Will they act on it professionally? Will it be customer-facing?

2. **What happens if the output is low quality?** Is a poor output easily corrected by the user? Or does it cause downstream harm (incorrect action, damaged relationship, incorrect customer data)?

3. **What is the context length?** Small models degrade more quickly on long contexts. For context > 2,000 tokens, prefer mid-tier or frontier.

4. **What is the latency requirement?** Interactive tasks (<2 seconds) should use small or mid-tier models. Async batch tasks can afford frontier model latency.

5. **What is the cost sensitivity?** If this task runs 100,000 times per day, a 5× model upgrade is a significant cost increase. If it runs 10 times per day, the cost difference is negligible.

---

## A/B Testing Model Routing Changes

Before changing a routing configuration in production:

1. Define the quality metric to evaluate (e.g., override rate, acceptance rate, citation accuracy)
2. Run the current and proposed routing configurations side-by-side on a held-out eval set
3. If quality is acceptable at the proposed routing, deploy to 5% of traffic
4. Monitor quality metrics for 7 days
5. If no regression, expand to 100%

Never change routing configurations without a quality eval. Cost savings that come at the cost of an unnoticed quality regression are not savings.

---

*See also: [AI Unit Economics](ai-unit-economics.md) · [Token Cost Model Template](token-cost-model-template.md) · [Latency Budget by Use Case](latency-budget-by-use-case.md)*
