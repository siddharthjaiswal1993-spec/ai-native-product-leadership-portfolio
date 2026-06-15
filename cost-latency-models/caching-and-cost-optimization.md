# Caching and Cost Optimisation for AI Products

**Semantic caching, prompt caching, response caching, KV cache, and when not to cache**

---

## Caching as a Cost Strategy

In traditional software, caching is a performance strategy. In AI products, caching is both a performance strategy and a cost strategy. Token costs are proportional to usage — every duplicate or repeated computation costs money. Caching eliminates redundant computation and the associated token cost.

The art of AI caching is identifying which parts of the AI workflow are stable and repeatable (cacheable) and which are dynamic and unique to each request (not cacheable). Getting this distinction wrong — caching when you should not — produces stale, incorrect outputs. Getting it right produces significant cost savings with no quality impact.

---

## Prompt Caching

**What it is:** Some model API providers support prompt caching — caching the KV (key-value) activations for a specific prefix of a prompt. If the same prefix is reused in multiple requests, the cached activations are reused, dramatically reducing the input token cost for the repeated prefix.

**Where it applies:** The system prompt and any static context that is the same across many requests.

**Typical savings:** 80–95% cost reduction on the cached prefix. Providers that support prompt caching (as of knowledge cutoff) typically charge ~10% of standard input token price for cache hits.

**Application in the portfolio:**
- Pre-visit brief: the system prompt (instructions, output schema definition, citation rules) is the same for every brief generation. Cache the system prompt.
- PRD generation: the system prompt and PRD template structure are fixed across all runs. Cache both.

**Requirement:** The cached prefix must appear at the start of the prompt, and must be identical across requests. Any variation in the prefix invalidates the cache.

**Implementing prompt caching:**
1. Structure your prompts with the fixed content first (system prompt, static instructions, output schema)
2. Place the variable content last (retrieved context, user query)
3. Use the provider's caching API (varies by provider; check documentation for exact implementation)
4. Monitor cache hit rate in your usage logs

---

## Semantic Caching

**What it is:** Caching the AI-generated response for a given input, and returning the cached response for future requests that are semantically similar (even if not textually identical) to the original.

**How it works:**
1. Embed the user's query
2. Search the semantic cache (a vector store of (query_embedding, response) pairs)
3. If a semantically similar query exists in the cache with similarity above a threshold, return the cached response
4. If no similar query exists, run the full AI pipeline, cache the response

**Application:**
- In-visit Q&A: if an FC at Location A asks "What are the open support issues?" and an FC at Location B (same franchise brand, similar question) asks the same question 30 minutes later, the semantic cache can return the prior response — but this is wrong because the responses should be location-specific.
- Semantic caching is appropriate for queries that are not entity-specific: general product FAQ questions, how-to queries, policy interpretation.

**Critical limitation:** Semantic caching is NOT appropriate for queries that are account-specific, location-specific, or time-sensitive. Returning a cached response from a different account is a data isolation violation. Returning a cached response that was accurate yesterday but not today produces stale information.

**When to use semantic caching:**
- Generic Q&A (no entity-specific data in the response)
- Policy interpretation queries (the policy is stable)
- Template or how-to queries (the answer is the same for all users)

**When NOT to use semantic caching:**
- Any query that references a specific account, customer, location, or entity
- Any query where the answer changes over time (current health scores, ticket status, usage data)
- Any query in a multi-tenant system where the correct answer varies by customer

---

## Response Caching

**What it is:** Caching the complete AI-generated output for a specific task type and input combination, at the level of the final delivered artifact.

**Application:**
- Pre-visit brief: the brief for Location A on June 15 is generated once, delivered to all FCs visiting that location that day. If two FCs share a location on the same day, they receive the same brief (appropriate — the underlying data is the same).
- Weekly digest: generated once for the CS team portfolio, cached for the delivery window.
- QBR section: generated once per account per QBR cycle, cached across all reviewers.

**Cache invalidation trigger:** When the underlying data changes materially (a new support ticket is submitted, the health score crosses a threshold), the cached artifact is invalidated and the next request triggers a fresh generation.

**Implementation:** Use a cache key that includes: task_type + entity_id + data_freshness_hash. The data_freshness_hash is a hash of the timestamps of the most recent source update for each signal type. When any source updates, the hash changes, the key misses, and fresh generation is triggered.

---

## KV Cache (Model-Level)

**What it is:** The key-value cache maintained by the transformer model during inference. Within a single generation request, the model caches the attention keys and values for the context that has already been processed, enabling efficient sequential token generation.

**What the PM needs to know:** KV cache is automatic and handled by the model inference layer. PMs do not configure it directly. However, it has implications for system design:

- **Streaming:** KV cache enables streaming responses. The model generates one token at a time, using the cached context. This allows the UI to start displaying the response before the full output is complete — important for interactive use cases.
- **Long contexts:** KV cache memory usage grows with context length. Very long contexts (hundreds of thousands of tokens) require substantial GPU memory. This is a cost and availability consideration for deployments with very large context windows.

---

## Caching Anti-Patterns

**Anti-pattern 1: Caching entity-specific responses and serving them cross-account**
The most dangerous caching mistake. A response about Acme Corp's health score is cached and served to a query about Ajax Corp. This is both a data accuracy failure and a data privacy violation.

**Prevention:** Cache keys must always include a non-forgeable entity identifier (account_id, location_id). Cross-entity cache hits are architecturally impossible if the key space is correctly scoped.

---

**Anti-pattern 2: Not invalidating caches when data changes**
A cached pre-visit brief becomes stale when a new support ticket is submitted or the health score changes. Serving a stale brief to a field consultant can cause them to miss important information.

**Prevention:** Define explicit cache invalidation triggers. Every data ingestion event that could materially change a cached artifact should trigger cache invalidation for the affected entities.

---

**Anti-pattern 3: Caching low-hit-rate content**
If the same query is rarely repeated, semantic caching or response caching creates overhead (embedding and storing the response) without saving anything (the cache is never hit).

**Prevention:** Monitor cache hit rates. If hit rate is below 20%, the caching layer is adding cost without saving cost. Remove or narrow the cache scope.

---

**Anti-pattern 4: Caching with too-aggressive similarity thresholds**
Setting the semantic cache similarity threshold too low (too permissive) causes the cache to serve responses for queries that are only superficially similar but substantively different.

**Prevention:** Calibrate the similarity threshold on a golden set of query pairs. Define the threshold as: "the minimum similarity score where returning the cached response is clearly correct." Test with both similar-but-different queries (should miss) and paraphrase queries (should hit).

---

## Caching Cost Impact Model (Illustrative)

| Caching Type | Estimated Hit Rate | Cost Reduction | Notes |
|---|---|---|---|
| Prompt caching (system prompt) | 95–100% | 20–30% of total input cost | High hit rate because system prompt is always reused |
| Response caching (pre-visit brief) | 10–20% | Variable | Low hit rate because each location is unique |
| Semantic caching (generic Q&A) | 40–60% | 40–60% of Q&A cost | Only applies to non-entity-specific queries |
| Combined (all above) | — | ~25–35% total AI cost | Illustrative; depends heavily on usage patterns |

*All numbers are illustrative estimates. Actual savings depend on your specific usage patterns, query distribution, and cache configuration.*

---

*See also: [AI Unit Economics](ai-unit-economics.md) · [Model Routing Strategy](model-routing-strategy.md) · [Token Cost Model Template](token-cost-model-template.md)*
