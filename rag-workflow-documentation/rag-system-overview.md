# RAG System Overview: What It Is, Why It Matters, and the PM's Role

---

## What RAG Is

Retrieval-Augmented Generation (RAG) is an architecture pattern that combines a retrieval system with a language model to produce answers that are grounded in a specific, curated corpus of documents — rather than relying solely on the model's training data.

The basic flow: when a user asks a question (or when an agent needs to generate a response), the system first retrieves the most relevant documents or text chunks from an indexed corpus, then provides those documents to the language model as context, and the model generates a response grounded in the retrieved content.

The critical distinction from pure LLM generation: with RAG, the model is constrained to what was retrieved. It cannot (with appropriate prompting and guardrails) make up facts about your company's customers, your product's capabilities, or your internal data — because those facts must be grounded in retrieved documents that you control.

This makes RAG the foundation for enterprise AI products where factual accuracy, auditability, and source attribution matter. Which is to say: most enterprise B2B SaaS use cases.

---

## Why RAG Matters for Enterprise AI Products

**The core problem RAG solves:** Foundation models know a lot, but they do not know your company's specific data — your customer records, your product documentation, your internal processes, your historical decisions. And they cannot be updated in real time as your data changes.

RAG solves this by making your data retrievable at inference time. Instead of asking "what does the model know about Customer X?", you retrieve the relevant records about Customer X and ask the model to reason about what it finds.

**Why this matters for enterprise trust:** Enterprise customers are rightfully sceptical of AI outputs that cannot be traced. When an AI system tells a CSM that a customer's health score is at risk, the CSM needs to know why — and they need to be able to verify it. RAG with citation enforcement makes this possible: every claim in the output can be traced to a specific document in the corpus.

**Why this matters for enterprise data governance:** With RAG, you control what the model sees. You can scope retrieval to authorised data for the requesting user. You can exclude sensitive documents from the index. You can apply the same access controls that govern your data systems to the retrieval layer. You cannot do this with pure model generation.

**Why this matters for freshness:** Your data changes daily. Customer health signals change. Support ticket volumes change. Product usage changes. A RAG system with daily index updates delivers current information; a model fine-tuned on a historical snapshot does not.

---

## The Full RAG Pipeline

A production RAG system has six major stages:

```
1. DATA SOURCES
   Raw documents, records, transcripts, tickets from operational systems

2. INGESTION PIPELINE
   Parse → Clean → Chunk → Embed → Index

3. RETRIEVAL
   Query → Embed query → Search index → Rerank → Return top-N chunks

4. CONTEXT ASSEMBLY
   Select chunks → Order and format → Check token budget → Build context window

5. GENERATION
   System prompt + context + user query → LLM → Raw response

6. POST-PROCESSING
   Citation injection → Schema validation → Guardrail checks → Delivery
```

Each stage is a product decision, not just an engineering decision. The choices made at each stage — how to chunk, which embedding model to use, how many chunks to retrieve, how to format the context, what the system prompt says — directly determine the quality, reliability, and cost of the AI product.

---

## The PM's Role in RAG Design Decisions

Most product managers treat RAG as an engineering concern and stay out of the detailed design decisions. This is a mistake. The quality of a RAG-based AI product is determined by product decisions at every stage of the pipeline.

**Decision 1: What goes in the corpus?**
This is a product decision. Not everything should be indexed. Indexing too broadly increases noise and degrades retrieval quality. Indexing too narrowly misses important signals. The PM should define the corpus scope: which systems, which document types, which time ranges, which access tiers.

**Decision 2: How is content chunked?**
Chunking determines what unit of information the retrieval system returns. A chunk that is too large buries the relevant passage in irrelevant content. A chunk that is too small loses context. The PM should understand the trade-offs between fixed-size, semantic, and hierarchical chunking and push for the right approach for the use case.

**Decision 3: How many chunks are retrieved?**
Retrieving more chunks increases recall (less likely to miss relevant information) but also increases noise and cost. The PM should define the quality vs. cost target for each use case and ensure the retrieval configuration is calibrated accordingly.

**Decision 4: What is the citation policy?**
Is every claim required to cite a source? Are citations displayed to the user? What happens when the model cannot find a grounded source for a claim? These are product decisions that directly affect user trust and the audit trail.

**Decision 5: What is the freshness requirement?**
How current does the indexed data need to be? Real-time for high-stakes alerts; daily for most intelligence workflows; weekly for historical synthesis. The freshness requirement drives ingestion architecture complexity and cost.

**Decision 6: Who can retrieve what?**
Access control at the retrieval layer is a product requirement. The PM should define the permission model for the retrieval system: which users can retrieve which documents, and how that is enforced.

---

## RAG vs. Fine-Tuning: When to Use What

A common product question: should we RAG or fine-tune?

| Criterion | RAG | Fine-Tuning |
|---|---|---|
| Data freshness | Excellent — updates with index | Poor — requires re-training |
| Source attribution | Excellent — built-in | Poor — model cannot cite sources |
| Domain adaptation (style, format) | Moderate | Excellent |
| Factual accuracy on private data | Excellent (when retrieval works) | Good (but can hallucinate interpolation) |
| Inference cost | Higher (retrieval + generation) | Lower (no retrieval step) |
| Setup complexity | Moderate | High |
| Interpretability | High (you can see what was retrieved) | Low (opaque model internals) |

**The answer for most enterprise B2B use cases:** RAG first. Fine-tune only if you have a specific style or format adaptation need that RAG cannot satisfy, or if inference cost at scale is prohibitive and the data does not change frequently.

Fine-tuning and RAG are not mutually exclusive. A fine-tuned model that generates in your brand's style, grounded by a RAG retrieval system, is a reasonable architecture for a mature product.

---

## Common RAG Failure Modes (Brief Overview)

Full analysis in [RAG Failure Modes](/rag-workflow-documentation/rag-failure-modes.md). The most important ones to understand:

1. **Missing chunk:** The relevant information exists in the corpus but is not retrieved (recall failure)
2. **Wrong chunk:** An irrelevant chunk is retrieved and used as context (precision failure)
3. **Hallucination despite retrieval:** The model ignores the retrieved context and generates from training data
4. **Citation fabrication:** The model cites a source that does not support the claim
5. **Stale index:** The corpus has not been updated; answers reflect outdated information

---

## RAG and Org Memory

The most powerful long-term property of a RAG system is that it creates an organisational memory that compounds. Every document added to the index is permanently retrievable. Every interaction that produces a high-quality answer can be added to the corpus as a verified example. Every override that a human makes to an AI-generated answer is a signal that can be used to improve future answers.

Over time, a RAG system that is actively maintained becomes harder to replace — not because of vendor lock-in, but because the corpus itself is the accumulated intelligence of the organisation. A competitor can copy the architecture; they cannot copy the two years of indexed organisational knowledge.

This is the strategic case for investing in RAG infrastructure early, maintaining corpus quality, and treating the index as a first-class product asset.

---

*See also: [Enterprise RAG Architecture](/rag-workflow-documentation/enterprise-rag-architecture.md) · [Chunking, Embeddings, and Retrieval](/rag-workflow-documentation/chunking-embeddings-retrieval.md) · [RAG Failure Modes](/rag-workflow-documentation/rag-failure-modes.md) · [RAG Evaluation Plan](/rag-workflow-documentation/rag-eval-plan.md)*
