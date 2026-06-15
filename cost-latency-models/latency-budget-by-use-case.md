# Latency Budget by Use Case

**Acceptable latency by workflow type, P50/P95/P99 targets, streaming, and user perception of latency**

---

## Why Latency Budgets Must Be Set Before Architecture Decisions

Latency is not a property of the AI model alone. It is determined by the combination of: retrieval latency, reranking latency, generation latency (which depends on model size, context length, and output length), and post-processing latency. Every architectural choice — model size, number of retrieval queries, reranker candidate count, context compression — affects latency.

If the PM does not specify latency requirements before the engineering team designs the pipeline, the team will optimise for quality (which generally means using larger models, retrieving more candidates, and running more post-processing steps) — and the resulting system may be technically impressive but too slow for interactive use.

The latency budget must be set at the product level, before architecture decisions are made, not after.

---

## User Perception of Latency

Before setting targets, understand how users perceive latency:

| Latency Range | User Experience | Context |
|---|---|---|
| <100ms | Instantaneous | Feels like local computation |
| 100–300ms | Nearly instantaneous | Barely perceptible |
| 300ms–1s | Noticeable but fast | Users accept for AI; comparable to a Google search |
| 1–3s | Acceptable for AI | Users start to wonder; add a loading indicator |
| 3–10s | Borderline | Users may feel impatient; progress feedback essential |
| 10–30s | Tolerable if expected | Must communicate that AI is "thinking"; use progress steps |
| >30s | Significant wait | Requires clear expectation setting; only acceptable for batch/async |

**The key insight:** AI users generally have higher latency tolerance than web users — because they understand that AI inference takes time. But this tolerance is not unlimited and depends heavily on context. Real-time conversational contexts require <2 seconds. Batch processing tasks can tolerate minutes.

---

## Workflow Types and Latency Budgets

### Type 1: Interactive Real-Time (< 2 seconds)

**Characteristics:** User is waiting actively, in-product, and expects an immediate response. Any lag breaks the conversational flow.

**Examples from the portfolio:**
- In-visit Q&A (field consultant asks a location-specific question during a visit)
- Customer health score dashboard loading
- Real-time signal anomaly alerts

**Latency targets:**
- P50: <800ms
- P95: <2,000ms (2 seconds)
- P99: <3,500ms

**Architecture implications:**
- Small-to-mid-tier model only (large model latency will breach the P95 target)
- Retrieval limited to top-5 chunks (more chunks increase retrieval latency)
- No reranking (adds 50–200ms per query)
- Streaming responses (first token visible within 300ms even if full response takes longer)
- Response caching for common queries

---

### Type 2: On-Demand Async (< 30 seconds)

**Characteristics:** User explicitly requests something and waits while the system processes. A progress indicator is visible. The user may do something else while waiting.

**Examples:**
- Pre-visit brief generation (triggered by a notification; user opens it on mobile)
- On-demand account intelligence card
- QBR section generation (one section at a time)
- PRD opportunity brief generation

**Latency targets:**
- P50: <8 seconds
- P95: <20 seconds
- P99: <30 seconds

**Architecture implications:**
- Frontier model acceptable for brief generation (quality > speed)
- 10–20 retrieved chunks acceptable (more retrieval for better context)
- Reranking included (adds quality, acceptable at this latency budget)
- Streaming response (user sees content appearing, not a spinner)
- Show progress steps: "Retrieving signals... Synthesising brief... Validating citations..."

---

### Type 3: Background/Scheduled (< 5 minutes per batch)

**Characteristics:** Process runs in the background on a schedule without any user waiting. User receives the output when it is ready.

**Examples:**
- Nightly pre-visit brief batch generation (all FCs' scheduled visits for tomorrow)
- Weekly portfolio health digest generation
- Daily health score recalculation for all accounts
- Nightly signal ingestion and indexing

**Latency targets:**
- Per-item processing: <60 seconds
- Full batch completion: defined by business requirement (e.g., all Monday digests ready by 7am Monday)
- Not a P-percentile target — a completion window target

**Architecture implications:**
- Frontier model acceptable (latency per item is not user-facing)
- Parallelise across accounts/locations
- Process cheapest-to-fail checks first (fail fast: if data is missing, skip and log, do not spend generation tokens)
- Retry with backoff on transient failures

---

### Type 4: Large Batch Synthesis (< 10 minutes)

**Characteristics:** User initiates a large synthesis task and is told to "come back when it's ready." Not interactive.

**Examples:**
- Full QBR deck generation (all sections for one account)
- Full PRD draft generation (all sections from multi-source signals)
- Large-scale retrospective analysis

**Latency targets:**
- Total completion: < 10 minutes
- First section available for preview: < 60 seconds (streaming delivery by section)

**Architecture implications:**
- Process sections in parallel where possible (section 1 and section 2 can be generated simultaneously)
- Deliver results progressively (first section delivered as soon as ready; remaining sections follow)
- Set explicit user expectations: "Your QBR will be ready in approximately 4–5 minutes. We'll notify you."

---

## Streaming Architecture

Streaming is the most impactful single optimisation for user-perceived latency. Instead of waiting for the full generated response before displaying anything, the model generates tokens one at a time and the UI displays them as they arrive.

**How it works:**
- Model API returns a streaming response (Server-Sent Events or equivalent)
- UI renders tokens as they arrive: the response "types itself" in front of the user
- User sees the first tokens within 200–500ms, even if the full response takes 8–15 seconds

**User perception improvement:** A 10-second response with streaming feels significantly faster than a 4-second response without streaming, because the user sees progress from the first token.

**Implementation requirements:**
- The AI API must support streaming (most frontier model APIs do)
- The frontend must render streaming tokens (requires state management for partial content)
- Structured JSON outputs require special handling: stream the raw text, parse to JSON only when the stream is complete

**When NOT to stream:**
- When the output must be fully validated before display (citation validation cannot start until the full output is available)
- When the output is not human-readable in partial form (binary outputs, highly structured JSON that is meaningless without all fields)
- When the UI cannot support streaming rendering (some server-rendered contexts)

---

## Latency Measurement and Alerting

**What to measure:**
- Time-to-first-token (TTFT): time from user request to first token delivered to the UI
- Time-to-complete (TTC): time from user request to full output delivered
- Component latencies: retrieval time, reranking time, generation time, post-processing time

**Alerting thresholds:**
- If P95 TTFT exceeds 1.5 seconds for interactive workflows: alert
- If P95 TTC exceeds 150% of the target: alert
- If P99 TTC exceeds 200% of the target: investigate for outliers

**Root cause analysis for latency spikes:**
1. Check model provider status page (provider incidents cause latency spikes)
2. Check context window size for affected requests (long contexts cause long generation times)
3. Check retrieval latency (vector store performance can degrade under load)
4. Check whether a specific user or account triggers disproportionate latency (large corpus, complex query)

---

*See also: [Model Routing Strategy](model-routing-strategy.md) · [AI Unit Economics](ai-unit-economics.md) · [Caching and Cost Optimisation](caching-and-cost-optimization.md)*
