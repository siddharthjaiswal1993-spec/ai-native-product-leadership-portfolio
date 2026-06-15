# Token Cost Model Template

**Calculate the cost of an AI workflow: input tokens, output tokens, model pricing, cost per workflow, cost per user per month**

*All prices in this template are illustrative/synthetic placeholders. Replace with actual provider pricing from your model provider's pricing page before using for financial planning.*

---

## How to Use This Template

1. Fill in the workflow parameters for your specific use case
2. Enter the actual model pricing from your provider (NOT the illustrative prices below)
3. Calculate cost per workflow run
4. Multiply by usage frequency to get cost per user per month
5. Compare against seat revenue to assess margin impact

---

## Token Cost Model Worksheet

### Step 1: Define the Workflow

| Field | Your Value |
|---|---|
| Workflow name | e.g., "Pre-Visit Brief Generation" |
| Trigger | e.g., "24h before each scheduled visit" |
| Frequency | e.g., "8 times per FC per month" |
| Output type | e.g., "Structured JSON + TTS audio" |

---

### Step 2: Input Token Count

| Component | Token Estimate | Notes |
|---|---|---|
| System prompt (fixed) | [TOKENS] | Count once; same for every run |
| Retrieval context (variable) | [TOKENS] | Avg tokens × avg chunks retrieved |
| User query / task specification | [TOKENS] | |
| Few-shot examples (if used) | [TOKENS] | |
| **Total input tokens** | **[SUM]** | |

**Example (illustrative):**

| Component | Token Count |
|---|---|
| System prompt | 600 |
| Retrieval context (5 signal types × 400 tokens) | 2,000 |
| Structured task specification | 100 |
| Total input tokens | 2,700 |

---

### Step 3: Output Token Count

| Component | Token Estimate | Notes |
|---|---|---|
| Primary output (generated text) | [TOKENS] | Avg length of generated output |
| Structured output overhead (JSON formatting) | [TOKENS] | Schema adds tokens |
| **Total output tokens** | **[SUM]** | |

**Example (illustrative):**

| Component | Token Count |
|---|---|
| Generated brief text | 700 |
| JSON schema formatting overhead | 150 |
| Total output tokens | 850 |

---

### Step 4: Model Pricing (SYNTHETIC — Replace with Actual)

*The following prices are illustrative placeholders for framework demonstration only. Do not use for actual financial planning. Check your model provider's current pricing page.*

| Model Tier | Illustrative Input Price | Illustrative Output Price |
|---|---|---|
| Frontier / Large (synthetic) | $15.00 / 1M tokens | $60.00 / 1M tokens |
| Mid / Medium (synthetic) | $3.00 / 1M tokens | $12.00 / 1M tokens |
| Small / Fast (synthetic) | $0.50 / 1M tokens | $2.00 / 1M tokens |
| Embedding (synthetic) | $0.02 / 1M tokens | N/A |

---

### Step 5: Generation Cost Per Run

```
Generation cost = (input_tokens / 1,000,000) × input_price 
                + (output_tokens / 1,000,000) × output_price
```

**Example (illustrative, using synthetic frontier model prices):**
```
Generation cost = (2,700 / 1,000,000) × $15.00 
                + (850 / 1,000,000) × $60.00
               = $0.0405 + $0.0510
               = $0.0915 per run (illustrative)
```

---

### Step 6: Retrieval and Other Costs Per Run

| Component | Formula | Illustrative Cost |
|---|---|---|
| Query embedding | (query_tokens / 1M) × embed_price × num_queries | [CALC] |
| Vector store queries | num_queries × query_fee | [CALC] |
| Reranking | num_candidates × rerank_cost | [CALC] |
| Citation validation | (num_claims / 1M) × small_model_price × 2 | [CALC] |
| TTS (if applicable) | char_count × tts_price | [CALC] |
| **Total non-generation cost** | | [SUM] |

---

### Step 7: Total Cost Per Workflow Run

```
Total cost per run = Generation cost + Retrieval and other costs
```

**Example (illustrative):**
```
Total cost = $0.0915 (generation) + $0.015 (retrieval) + $0.020 (citation) + $0.060 (TTS)
           = $0.1865 per run (illustrative)
```

---

### Step 8: Cost Per User Per Month

```
Monthly cost per user = Cost per run × Runs per user per month
```

**Example (illustrative):**
```
8 visits/month × $0.1865/run = $1.49/FC/month (illustrative)
```

---

### Step 9: Multiple Workflows Per User

Real products have multiple AI workflows. Sum across all workflows:

| Workflow | Runs/Month | Cost/Run (Illust.) | Monthly Cost (Illust.) |
|---|---|---|---|
| Pre-visit brief | 8 | $0.19 | $1.52 |
| In-visit Q&A (avg 10 queries) | 80 queries | $0.02 | $1.60 |
| Post-visit action extraction | 8 | $0.04 | $0.32 |
| **Total AI cost per FC/month** | | | **$3.44 (illustrative)** |

---

### Step 10: Margin Analysis

| Metric | Value |
|---|---|
| Seat price per month | $[PRICE] |
| Total AI cost per seat per month | $[COST] (from Step 9) |
| AI cost as % of seat revenue | [COST / PRICE × 100]% |
| Non-AI COGS (hosting, support) | $[COGS] |
| Gross margin after AI cost | [(PRICE - COST - COGS) / PRICE × 100]% |

**Targets:** AI cost < 15% of seat revenue is generally healthy for an AI-first product. Above 20%, investigate optimisation opportunities.

---

## Optimisation Impact Calculator

If you apply a specific optimisation, how much does it save?

| Optimisation | Levers | Illustrative Saving |
|---|---|---|
| Prompt caching (50% hit rate) | System prompt cache hits = 50% × input_tokens × cache_discount | ~30% reduction in generation cost |
| Small model for Q&A | Route Q&A from frontier to small model | ~90% reduction in Q&A generation cost |
| Context compression (60%) | Reduce context from 2,000 to 800 tokens | ~25% reduction in total input tokens |
| Reduce TTS (opt-in only) | FC chooses whether to generate audio | ~100% reduction in TTS cost for opt-out users |

---

## Usage Cap Design

At what usage level does a customer become unprofitable?

```
Max profitable monthly runs = (Seat revenue - Non-AI COGS) / Cost per run
```

**Example (illustrative):**
```
Max profitable brief generations = ($50 seat - $10 non-AI COGS) / $0.19 = 210 runs/month
```

At 8 briefs/month per FC, this gives a 26× headroom. Even at 10× higher usage (80 briefs/month), the feature remains profitable.

For high-token-cost features with thin headroom, consider usage-based overage pricing above a tier threshold.

---

*See also: [AI Unit Economics](ai-unit-economics.md) · [Model Routing Strategy](model-routing-strategy.md) · [Caching and Cost Optimisation](caching-and-cost-optimization.md)*
