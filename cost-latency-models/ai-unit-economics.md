# AI Feature Unit Economics

**Framework for AI feature unit economics: cost per AI action, revenue impact, margin analysis, break-even model**

*All numbers in this document are illustrative/synthetic. They are examples for the purpose of demonstrating the analytical framework, not actual product economics.*

---

## Why PMs Must Own AI Unit Economics

Traditional SaaS features have near-zero marginal cost once built. Adding another user to a Jira project costs essentially nothing. Adding another user to an AI product that runs a multi-agent synthesis pipeline on their behalf costs money — in tokens, in inference, in storage, in retrieval compute.

If the PM does not model the unit economics of AI features, the company will discover at scale that the feature's gross margin is unacceptably low, or that specific usage patterns by large customers are economically destructive.

The PM who ships an AI feature without understanding its unit economics is the PM who creates an urgent problem for finance six months after launch.

---

## The AI Feature Cost Stack

### Cost Component 1: Ingestion Cost (Per Document, One-Time)

The cost of processing a new document into the indexed corpus:
- **Parsing:** Negligible (compute-side)
- **Embedding:** Cost = (token count of document) × (embedding model price per 1M tokens)
- **Storage:** Vectors stored in the vector database × (storage price per GB/month)

**Illustrative example (synthetic):**
- Average support ticket: 200 tokens
- Embedding cost (synthetic pricing): $0.02 / 1M tokens → $0.000004 per ticket
- 10,000 tickets/month → $0.04/month in embedding costs

Ingestion cost is typically very small compared to inference cost. The larger concern is storage accumulation over time.

---

### Cost Component 2: Retrieval Cost (Per Query)

The cost of retrieving relevant chunks for a query:
- **Embedding the query:** (query token count) × (embedding model price)
- **Vector search:** Vector store query cost (varies by provider; some charge per query, some per stored vector, some bandwidth-based)
- **Reranking:** (number of candidates) × (reranker inference cost)

**Illustrative example (synthetic):**
- Query embedding: 50 tokens × $0.02/1M = $0.000001
- Vector search: $0.001 per query (synthetic illustrative rate)
- Reranker (20 candidates, cross-encoder): $0.002 per query (synthetic)
- Total retrieval cost per query: ~$0.003

---

### Cost Component 3: Generation Cost (Per Output)

The largest cost component for most AI features:
- **Input tokens:** System prompt + context window (retrieved chunks) + user query
- **Output tokens:** Generated response length
- **Model pricing:** Input and output tokens priced differently (output tokens are typically 3-5× more expensive than input tokens)

**Illustrative example — Pre-Visit Brief (synthetic):**
- System prompt: 500 tokens
- Retrieved chunks (5 signal sources × 400 tokens each): 2,000 tokens
- Total input: 2,500 tokens
- Generated brief: 800 tokens output
- Synthetic model pricing: $15/1M input tokens, $60/1M output tokens (illustrative — refer to actual provider pricing)
- Input cost: 2,500 × $0.000015 = $0.0375
- Output cost: 800 × $0.000060 = $0.0480
- **Generation cost per brief: ~$0.085 (illustrative)**

---

### Cost Component 4: Post-Processing Cost

Citation validation, schema validation, content scanning:
- Additional LLM calls for citation validation: (number of claims) × (small model inference cost)
- TTS generation for audio summary (if applicable): (character count) × (TTS price per character)

**Illustrative: TTS audio summary**
- Brief text: 600 words / 4,000 characters
- Synthetic TTS pricing: $15 per 1M characters
- TTS cost per brief: $0.06 (illustrative)

---

## Total Cost Per AI Action: Worked Example

**AI Field Consultant Pre-Visit Brief — Full Cost Model (Synthetic)**

| Component | Cost (Illustrative) |
|---|---|
| Query embedding (5 signal retrieval queries) | $0.015 |
| Vector search (5 queries) | $0.005 |
| Reranking (20 candidates per query) | $0.010 |
| Context assembly | Negligible |
| Generation (large model, frontier tier) | $0.085 |
| Citation validation (small model, 10 claims) | $0.020 |
| TTS audio generation | $0.060 |
| Storage (amortised) | $0.002 |
| **Total cost per pre-visit brief** | **~$0.197** |

*All numbers are illustrative and synthetic. Actual costs depend on model pricing, usage volumes, and caching efficiency.*

---

## Revenue Model and Margin Analysis

**Illustrative pricing model:**
- Product pricing: $50 / FC seat / month
- Average visits per FC per month: 8
- AI briefs generated per visit: 1
- Pre-visit brief cost per brief: $0.197

**Margin calculation:**
- Revenue per FC per month: $50
- AI cost per FC per month (briefs only): 8 × $0.197 = $1.58
- AI cost as % of seat revenue: 3.2%
- Gross margin impact of pre-visit brief: acceptable

**But what about at scale?**
If FCs use in-visit Q&A heavily (20 queries per visit):
- Q&A cost per query: ~$0.025 (small model, short context)
- Q&A cost per month per FC: 8 visits × 20 queries × $0.025 = $4.00
- Total AI cost per FC per month: $1.58 (briefs) + $4.00 (Q&A) = $5.58
- AI cost as % of seat revenue: 11.2%

At this level, AI cost is still acceptable but worth monitoring. If the model used for Q&A is the frontier large model instead of a small model, the Q&A cost increases 5-10× — changing the economics significantly.

---

## Break-Even Model

At what usage volume does the AI feature pay for itself (in value delivered to the customer)?

**Illustrative break-even analysis:**
- Baseline: FC spends 60 minutes per week on research and post-visit administration
- AI saves: 45 minutes per week (75% reduction)
- FC hourly cost to the franchisor: $75/hour (illustrative, loaded cost)
- Weekly value delivered: 0.75 hours × $75 = $56.25
- Monthly value delivered: ~$225 per FC seat

- AI feature cost to the franchisor: $50/month per FC seat
- Savings net of cost: $225 - $50 = $175/month per FC

**Break-even:** The feature generates immediate positive ROI at any usage level that saves >40 minutes per FC per month. Even with the most conservative efficiency assumption (15 minutes saved per week), the feature breaks even.

This analysis supports confident enterprise pricing and customer ROI justification.

---

## Cost Optimisation Opportunities

**1. Model routing:** Use a small, fast model for in-visit Q&A (where latency matters more than quality) and a frontier model for pre-visit brief generation (where quality matters more than latency). See [Model Routing Strategy](model-routing-strategy.md).

**2. Prompt caching:** If the system prompt and context (location taxonomy, signal type definitions) are the same across many brief generations, prompt caching reduces the input token cost significantly for providers that support it. See [Caching and Cost Optimisation](caching-and-cost-optimization.md).

**3. Context compression:** Instead of passing full 400-token chunks to the generation context, compress chunks to the most relevant 150 tokens before assembly. Reduces input tokens by ~60% with modest quality impact for well-structured content.

**4. Retrieval batching:** For the nightly brief generation run (all FCs' visits tomorrow), batch the retrieval queries rather than running them individually. Vector search can be batched for cost efficiency.

---

*See also: [Token Cost Model Template](token-cost-model-template.md) · [Model Routing Strategy](model-routing-strategy.md) · [Caching and Cost Optimisation](caching-and-cost-optimization.md)*
