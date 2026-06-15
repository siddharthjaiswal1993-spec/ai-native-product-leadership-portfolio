# Responsible AI Principles for Enterprise B2B SaaS

**Eight principles applied to enterprise AI product design**

---

## How to Use These Principles

These principles are not decorative. Each one has specific product design implications. For every AI feature, review each principle and document how it is implemented — or document why it does not apply and what the alternative control is.

---

## Principle 1: Explainability

**The principle:** Users of AI-assisted systems should be able to understand, at an appropriate level of detail, why the AI produced a specific output.

**Why it matters in enterprise B2B SaaS:** Enterprise users are professionally accountable for the decisions they make with AI assistance. They cannot be accountable for a decision they cannot understand. Explainability is the prerequisite for meaningful human oversight.

**What this requires in product design:**
- Every AI finding or recommendation must include a plain-language explanation of why it was generated
- Citations must be provided for all factual claims, enabling users to verify the basis for any assertion
- Confidence scores should be accompanied by a plain-language explanation of what drives the confidence level
- When the system cannot explain why it produced an output (e.g., pure black-box model inference without retrieval), the output should not be used in high-consequence decisions without additional human judgment

**Anti-pattern to avoid:** Presenting a risk score or recommendation without explaining what signals drove it. Enterprise users will not trust scores they cannot interrogate.

---

## Principle 2: Fairness

**The principle:** AI systems should not systematically disadvantage specific groups, customer segments, or demographics in ways that are unjustified by legitimate business criteria.

**Why it matters in enterprise B2B SaaS:** Recommendation systems, churn prediction models, and risk scoring systems can embed systematic biases that favour certain customer segments over others. In contexts like credit decisions, employment recommendations, or financial risk assessments, these biases may violate anti-discrimination laws.

**What this requires in product design:**
- Disaggregate performance metrics by customer segment, industry, and account tier during evaluation
- Review for systematic disparities in recommendation rates: if Enterprise-tier accounts are systematically rated as lower churn risk than SMB accounts with similar signals, investigate whether the model has learned a proxy bias
- Do not use demographic information as a model input in ways that create protected-class discrimination
- Document fairness analysis in the feature's security and legal review

---

## Principle 3: Human Oversight

**The principle:** In high-consequence contexts, humans must retain meaningful oversight of AI-assisted decisions — not just formal approval authority, but genuine understanding and the ability to intervene effectively.

**Why it matters in enterprise B2B SaaS:** As AI systems automate more of the decision-making process, there is a risk that human oversight becomes ceremonial — the human "approves" without understanding or genuinely reviewing the AI's work. This creates an illusion of human accountability without the substance.

**What this requires in product design:**
- HITL requirements should be calibrated so that the human reviewer has enough time, context, and cognitive access to make a genuine judgment — not just a rubber stamp
- Monitor approval time per item: unusually fast approvals indicate rubber-stamping
- The reviewer interface must surface the AI's reasoning, evidence, and confidence — not just the recommended action
- Autonomy levels should be graduated and earned through demonstrated performance, not granted by default

---

## Principle 4: Data Minimisation

**The principle:** AI systems should process only the data necessary for their defined purpose. Data should not be collected, retained, or processed beyond what is needed.

**Why it matters in enterprise B2B SaaS:** Each additional data source connected to the AI pipeline increases the privacy risk surface, the attack surface, and the compliance obligation. More data is not always better if the additional data creates risk without improving outputs.

**What this requires in product design:**
- Maintain a documented inventory of all data sources connected to the AI pipeline with justification for each
- Remove or disconnect data sources that are no longer contributing meaningfully to AI output quality
- Apply the minimum necessary access principle at the retrieval layer
- Define and enforce data retention periods — do not retain indexed data indefinitely

---

## Principle 5: Security

**The principle:** AI systems should be designed to be resilient against security threats specific to AI, including prompt injection, model extraction, data poisoning, and privacy attacks.

**Why it matters in enterprise B2B SaaS:** AI systems introduce new attack surfaces that traditional security controls do not cover. Prompt injection, in particular, is a novel attack vector that requires AI-specific mitigations.

**What this requires in product design:**
- Implement the mitigations in [Prompt Injection Risk](prompt-injection-risk.md)
- Include AI pipeline components in the scope of security testing and penetration testing
- Maintain a threat model that explicitly covers AI-specific attack vectors
- Monitor for anomalous model behavior (unexpected output patterns, unusual retrieval access) as a security detection mechanism

---

## Principle 6: Transparency

**The principle:** The use of AI in decision-making processes should be disclosed to the people affected by those decisions.

**Why it matters in enterprise B2B SaaS:** Enterprise customers (and their customers) have a right to know when AI is contributing to decisions that affect them. Transparency builds trust and enables meaningful consent.

**What this requires in product design:**
- Field consultants should disclose to franchisee operators that they are using an AI-generated brief (appropriately framed as a benefit: "The system gives me a comprehensive picture before I arrive so we can spend our time on the important things")
- CSMs should not present AI-generated QBR content as manually researched without disclosure
- Customer-facing documentation should describe how AI is used in the platform
- The AI feature should not be designed to be invisible to end users — users should be able to tell when they are interacting with AI-assisted content

---

## Principle 7: Accountability

**The principle:** Responsibility for AI-assisted decisions must be clearly assigned to specific humans. The AI system is a tool; the humans who configure, deploy, and use it are accountable for its outputs.

**Why it matters in enterprise B2B SaaS:** As AI systems become more autonomous, accountability can become diffuse ("the AI made that decision"). Preventing this requires designing clear accountability structures into the product.

**What this requires in product design:**
- Every approval action in the HITL layer records the identity of the approving human and timestamps the approval
- The audit trail is immutable and includes who took what action and when
- Terms of service clearly state that the customer is responsible for reviewing AI-generated content before acting on it
- When an AI system takes an autonomous action, the human who configured the autonomy level bears responsibility for the outcome

---

## Principle 8: Robustness

**The principle:** AI systems should be designed to behave reliably across a range of conditions, including edge cases, distribution shifts, and adversarial inputs.

**Why it matters in enterprise B2B SaaS:** Enterprise customers depend on AI systems for operationally important decisions. A system that works well on the training distribution but fails on unusual inputs — a new customer segment, a novel type of support issue, an edge case in a provisioning workflow — creates risk.

**What this requires in product design:**
- The evaluation suite must include edge cases and adversarial inputs, not just happy-path scenarios
- Monitor for distribution shift: if the production input distribution drifts significantly from the training/evaluation distribution, performance may degrade in ways not visible from standard metrics
- Implement graceful degradation: when the system encounters inputs it is not confident about, it should return "insufficient information" rather than a low-quality output
- Regular adversarial testing to validate robustness against known attack patterns

---

## Applying the Principles in Practice

For each AI feature, complete the following before launch:

| Principle | Implementation Documented | PM Sign-Off |
|---|---|---|
| Explainability | ☐ | ☐ |
| Fairness | ☐ | ☐ |
| Human Oversight | ☐ | ☐ |
| Data Minimisation | ☐ | ☐ |
| Security | ☐ | ☐ |
| Transparency | ☐ | ☐ |
| Accountability | ☐ | ☐ |
| Robustness | ☐ | ☐ |

For any principle where the standard implementation is not applicable, document the reason and the alternative control.

---

*See also: [AI Risk Register](ai-risk-register.md) · [Security and Legal Review Checklist](security-legal-review-checklist.md) · [HITL Design Framework](/hitl-governance/human-in-the-loop-design.md)*
