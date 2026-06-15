# AI Evaluation Frameworks — Overview

**Why evals matter for AI products, PM vs. ML engineer responsibilities, and the three layers of evaluation**

---

## Why Evals Are a Product Function, Not Just an Engineering Function

In traditional software, quality is verified through unit tests, integration tests, and QA processes that check deterministic behavior. If the code is supposed to return 42, the test checks for 42. Pass or fail is unambiguous.

AI products are not deterministic. The same input can produce different outputs on different runs. Quality is a spectrum, not a binary. A response can be partially correct, technically accurate but useless, factually grounded but incomplete, or excellent in most cases but catastrophically wrong in edge cases.

This requires a fundamentally different quality discipline: **evaluation**. And because AI product quality is ultimately defined by whether the product helps users accomplish their goals — a product question, not a model question — evals are a core product management responsibility.

The PM who says "evals are the ML engineer's job" is the PM whose AI feature ships without a quality baseline, degrades silently after a prompt update, and produces the first customer escalation about a hallucinated claim.

---

## The Three Layers of AI Evaluation

### Layer 1: Offline Evals (Pre-Deployment Quality Gates)
Run before shipping. Tests the system against a golden dataset of known inputs and expected outputs. Catches regressions when models, prompts, or retrieval configurations change.

**Who owns it:** PM defines what to test and the quality thresholds. ML engineer / PM engineer builds and runs the pipeline.

**Key methods:** Golden dataset evaluation, LLM-as-judge, automated schema validation, regression suites.

### Layer 2: Online Metrics (Post-Deployment Quality Monitoring)
Run continuously in production. Measures how the system performs on real user inputs and real user behavior. Catches drift, distribution shift, and failure patterns not covered by the offline eval set.

**Who owns it:** PM defines which metrics matter and sets thresholds for alerts. Analytics / data engineering builds the monitoring pipeline.

**Key methods:** Acceptance rate, override rate, task completion rate, latency, cost per action, user satisfaction signals.

### Layer 3: Human Feedback Loop (Continuous Improvement Signal)
Captures structured feedback from users and reviewers on AI outputs. Feeds back into the eval dataset and (over time) into model improvement. The mechanism that allows the AI product to get better with use.

**Who owns it:** PM designs the feedback UX and the feedback data schema. ML engineer connects feedback to model improvement. Data team manages feedback data quality.

**Key methods:** Thumbs up/down, structured rejection reasons, expert annotation, preference pairs.

---

## PM vs. ML Engineer Responsibilities in Eval

| Responsibility | PM | ML Engineer |
|---|---|---|
| Define what quality means for this use case | ✓ | |
| Set quality thresholds (what is "good enough to ship") | ✓ | |
| Identify which eval dimensions matter most | ✓ | |
| Define the golden dataset schema | ✓ | |
| Contribute domain-expert annotations to the golden dataset | ✓ | |
| Build the eval pipeline | | ✓ |
| Run evals and report results | | ✓ |
| Design the LLM-as-judge rubric | ✓ (with ML input) | |
| Calibrate the judge against human annotation | | ✓ |
| Define online metric schema and alert thresholds | ✓ | |
| Build production monitoring pipeline | | ✓ |
| Design the user feedback UX | ✓ | |
| Build the feedback data collection infrastructure | | ✓ |
| Decide what to do with eval results | ✓ | |

---

## Eval Cadence

| Eval Type | Frequency | Trigger |
|---|---|---|
| Regression suite | On every model/prompt/retrieval change | Automated CI/CD gate |
| Golden dataset eval (retrieval) | Weekly | Automated scheduled run |
| LLM-as-judge (generation quality) | Monthly | Automated scheduled run on production sample |
| Human annotation review | Monthly | Product team reviews sample + eval metrics |
| Adversarial input testing | On major releases | Manual trigger |
| Human feedback review | Weekly | PM reviews feedback data |

---

## Files in This Directory

| File | Description |
|---|---|
| [golden-dataset-design.md](golden-dataset-design.md) | How to build a golden dataset for offline evals |
| [llm-as-judge-rubrics.md](llm-as-judge-rubrics.md) | Designing and calibrating LLM-as-judge rubrics |
| [rag-evaluation.md](rag-evaluation.md) | RAG-specific evals: retrieval and generation quality |
| [agent-task-success-evals.md](agent-task-success-evals.md) | Evaluating agent task completion quality |
| [online-ai-product-metrics.md](online-ai-product-metrics.md) | Online metrics for deployed AI products |
| [human-feedback-loop.md](human-feedback-loop.md) | Designing the human feedback loop for continuous improvement |

---

*See also: [RAG Evaluation Plan](/rag-workflow-documentation/rag-eval-plan.md) · [Eval Generation Prompts](/prompt-library/eval-generation-prompts.md) · [Evals Are the New Product Analytics](/strategy-memos/evals-are-the-new-product-analytics.md)*
