# Prompt Library — Overview and Usage Guide

---

## Purpose

This prompt library documents the key prompts used across the AI-native product portfolio. Each prompt is designed for a specific enterprise B2B SaaS use case, with documented system prompts, user prompt templates, input variable definitions, expected output schemas, guardrails, and evaluation checklists.

---

## How to Use This Library

Each prompt file documents a cluster of related prompts for a use case. For each prompt:

1. **Review the use case and goal** before adapting
2. **Fill in the input variables** from your specific context
3. **Validate the output against the output schema** — if the output is malformed, re-prompt with the schema violation as feedback
4. **Apply the guardrails** — all guardrails listed are non-optional in production use
5. **Run through the evaluation checklist** before deploying in a customer-facing context

---

## Naming Conventions

Files are named by domain: `[domain]-prompts.md`. Prompt IDs within each file follow the convention `[DOMAIN]-[VERSION]-[NUMBER]`. Example: `CS-v1-01` for the first prompt in the Customer Success library, version 1.

When a prompt is updated, increment the version number and retain the prior version in the file for comparison. Do not overwrite without version tracking.

---

## Versioning Approach

All prompts in this library are considered **v1** unless marked otherwise. When a prompt is modified:
- Increment the version in the prompt ID
- Record the change summary and date
- Keep the prior version for reference (marked as `DEPRECATED`)
- Re-run the evaluation checklist for the updated version before production use

---

## Prompt Categories

| File | Domain | Prompt Count |
|---|---|---|
| [ai-prd-generation-prompts.md](ai-prd-generation-prompts.md) | Product PRD generation | 4 prompts |
| [customer-success-intelligence-prompts.md](customer-success-intelligence-prompts.md) | CS intelligence | 5 prompts |
| [product-signal-synthesis-prompts.md](product-signal-synthesis-prompts.md) | Product signal synthesis | 4 prompts |
| [enterprise-risk-diagnosis-prompts.md](enterprise-risk-diagnosis-prompts.md) | Risk diagnosis | 4 prompts |
| [eval-generation-prompts.md](eval-generation-prompts.md) | Eval dataset generation | 4 prompts |
| [gtm-positioning-prompts.md](gtm-positioning-prompts.md) | GTM and positioning | 4 prompts |

---

## General Prompt Engineering Principles (Applied Across All Prompts in This Library)

**1. Role + task + constraints:** Every system prompt defines: who the model is acting as, what the specific task is, and what constraints apply (citation requirements, output format, prohibited behaviors).

**2. Citation enforcement:** For any prompt that generates factual claims, the system prompt explicitly requires inline citations. This is non-negotiable for enterprise use.

**3. Explicit "I don't know" instruction:** Every prompt that depends on retrieval includes explicit instruction to return a "no information" response rather than generating a plausible-sounding guess when the context is insufficient.

**4. Output schema definition:** Every prompt includes an expected output schema (JSON or structured prose). Schema validation is applied post-generation.

**5. No training data reliance:** For factual retrieval-based tasks, the system prompt explicitly prohibits the model from using training knowledge. Only retrieved context is authoritative.

**6. Temperature:** Most prompts in this library use temperature = 0 or low temperature (0.1–0.2) for factual tasks (retrieval, extraction, classification). Higher temperature (0.5–0.7) for creative or generative tasks (positioning, narrative drafting).

---

## Evaluation Before Production Use

For any new or modified prompt, run through this checklist:

- [ ] Manual testing with 5+ representative inputs, including at least 1 edge case
- [ ] Output schema validation on all test outputs
- [ ] Citation accuracy verification on at least 3 factual outputs
- [ ] "Insufficient context" case tested — does the model respond correctly when context is missing?
- [ ] Adversarial input tested — does the model refuse or handle prompt injection attempts appropriately?
- [ ] Guardrails verified — does the model comply with all guardrail instructions?
- [ ] Latency measured — is the generation time acceptable for the intended use case?
- [ ] Cost estimated — is the token count within the acceptable cost range for the use case?

---

*See also: [AI Evaluation Frameworks](/ai-evaluation-frameworks/README.md) · [RAG Evaluation Plan](/rag-workflow-documentation/rag-eval-plan.md)*
