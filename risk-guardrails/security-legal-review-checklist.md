# Security and Legal Review Checklist for Enterprise AI Features

**A 40-item pre-launch checklist covering security, legal, privacy, compliance, audit, and vendor contracts**

*Complete this checklist before any AI feature that processes customer data is released to enterprise customers.*

---

## Section 1: Data Security (Items 1–10)

- [ ] **1. Authentication and authorisation:** All AI API endpoints require authenticated requests. Unauthenticated access is not possible.

- [ ] **2. Retrieval access control:** Retrieval queries enforce account-level access control at the API layer. A user cannot retrieve data for accounts they are not authorised to access, regardless of query parameters.

- [ ] **3. Encryption in transit:** All data transmitted between the application, the AI model API, the vector store, and the client is encrypted using TLS 1.2 or higher.

- [ ] **4. Encryption at rest:** Indexed vectors, stored embeddings, audit logs, and feedback data are encrypted at rest.

- [ ] **5. API key management:** Model provider API keys and vector store access credentials are stored in a secrets management system (not in environment variables or source code). Keys are rotated on a defined schedule.

- [ ] **6. Sandbox isolation:** The AI pipeline does not have access to the production database or other critical infrastructure systems beyond what is required for its defined operations.

- [ ] **7. Input validation:** All inputs to the AI pipeline (queries, documents for ingestion, tool call parameters) are validated before processing. Inputs that fail validation are rejected with an error, not processed.

- [ ] **8. Output content scanning:** AI-generated outputs are scanned for PII leakage before delivery. Outputs containing PII from an unauthorised source are quarantined.

- [ ] **9. Logging and monitoring:** All AI pipeline operations are logged. Alerts are configured for anomalous patterns (unusual query volume, access to accounts the user does not normally query, large output volumes).

- [ ] **10. Penetration testing:** The AI feature has been included in scope for the most recent penetration test. All HIGH and CRITICAL findings have been remediated before go-live.

---

## Section 2: Privacy and Data Processing (Items 11–20)

- [ ] **11. DPA in place:** A Data Processing Agreement is in place with every enterprise customer before their data is connected to the AI pipeline.

- [ ] **12. PII inventory:** A PII inventory has been created for this feature: which PII categories are processed, from which source systems, for what purpose.

- [ ] **13. PII redaction implemented:** PII redaction is applied at ingestion for the PII categories identified in the inventory. Redaction is tested and verified to be functioning correctly.

- [ ] **14. Lawful basis for processing:** The lawful basis for processing each category of personal data has been identified and documented (legitimate interests, contractual necessity, consent, etc.).

- [ ] **15. Data minimisation:** The AI pipeline ingests only the data necessary for its defined purpose. Data sources that are not actively used in AI outputs are not connected.

- [ ] **16. Data retention policy implemented:** Retention periods for indexed chunks, generated outputs, and feedback data are defined and enforced. Automated deletion at retention period expiry is implemented and tested.

- [ ] **17. Right to erasure capability:** A process exists to identify and delete all indexed data related to a specific individual upon request. This process has been tested end-to-end.

- [ ] **18. Consent for call recordings:** If call transcripts are indexed, confirmation is in place that the customer has obtained appropriate consent from all call participants per applicable regulations.

- [ ] **19. Cross-border transfer controls:** If customer data may be processed outside the customer's home jurisdiction, the applicable data transfer mechanism is in place (Standard Contractual Clauses for EU-to-non-EU transfers, etc.).

- [ ] **20. Privacy notice update:** The product's privacy notice/policy has been updated to reflect the AI feature's data processing activities.

---

## Section 3: Legal and Compliance (Items 21–30)

- [ ] **21. Terms of service update:** The Terms of Service have been updated to describe the AI feature, including any limitations of the AI's outputs and the user's responsibility for reviewing AI-generated content before acting.

- [ ] **22. AI output disclaimer:** A visible disclaimer is displayed to users of the AI feature stating that AI-generated content may contain errors and should be reviewed by a qualified person before being relied upon.

- [ ] **23. Intellectual property review:** Legal has reviewed whether AI training on customer data creates any IP-related obligations or risks.

- [ ] **24. No autonomous legal or financial decisions:** The AI system does not autonomously make decisions that have direct legal or financial consequences (contract execution, payment initiation, regulatory filings). These actions require human approval.

- [ ] **25. Industry-specific compliance review:** If target customers include regulated industries (financial services, healthcare, legal, government), the AI feature has been reviewed by counsel familiar with the applicable regulations (SOX, HIPAA, FINRA, FedRAMP, etc.).

- [ ] **26. AI liability allocation:** The customer agreement clearly allocates liability for AI-generated outputs. The customer bears responsibility for outputs they approve and act on; the vendor bears responsibility for system availability and data security.

- [ ] **27. Competitive intelligence handling:** Any competitive information in AI outputs (competitor mentions from customer transcripts) is labeled as internal only and not surfaced in customer-facing outputs.

- [ ] **28. Non-discrimination review:** Legal and product have reviewed whether the AI feature could be used in employment, credit, housing, or other regulated contexts where AI-based decisions trigger anti-discrimination laws. If yes, additional controls are in place.

- [ ] **29. Export controls:** If the AI system uses models subject to export control regulations, the customer onboarding process includes export control screening.

- [ ] **30. Insurance:** The company's cyber liability and errors and omissions insurance covers AI-assisted decisions and AI-generated content.

---

## Section 4: Audit and Governance (Items 31–36)

- [ ] **31. Audit log implemented and complete:** The audit log captures all required events (as defined in [Audit Logs and Decision Trails](/hitl-governance/audit-logs-and-decision-trails.md)) and the log has been verified to be complete and accurate.

- [ ] **32. Audit log immutability:** The audit log is stored in an immutable or append-only system. Modification and deletion are technically prevented for authorised retention periods.

- [ ] **33. Audit log access control:** Access to audit logs is restricted to authorised personnel. Access to audit logs is itself logged.

- [ ] **34. HITL controls reviewed:** The HITL controls for all customer-facing AI actions have been reviewed and confirmed to be enforced at the API layer (not just the UI layer).

- [ ] **35. Incident response plan:** A specific incident response plan exists for AI-related incidents (hallucination in customer-facing content, PII breach from AI pipeline, prompt injection attack). The plan includes notification timelines and escalation paths.

- [ ] **36. AI ethics review:** The feature has been reviewed against the organisation's responsible AI principles. Any findings from the review have been addressed or documented with rationale for acceptance.

---

## Section 5: Vendor Contracts (Items 37–40)

- [ ] **37. Model provider DPA:** A Data Processing Agreement with the model provider (e.g., OpenAI, Anthropic) specifies that customer data submitted in prompts is not used for model training and is retained only for the minimum period necessary.

- [ ] **38. Vector store provider DPA:** A DPA with the vector store provider specifies data isolation, retention, and deletion requirements.

- [ ] **39. Model provider security review:** The model provider's security posture has been reviewed (SOC 2 Type II report reviewed, or equivalent). The review has been shared with the security team.

- [ ] **40. Sub-processor disclosure:** All sub-processors involved in processing customer data through the AI pipeline (model provider, vector store, cloud infrastructure) are disclosed in the customer DPA.

---

## Sign-Off Requirements

Before the AI feature can be released to enterprise customers:

| Role | Requirement |
|---|---|
| Product Manager | All 40 items checked and confirmed |
| Security Lead | Section 1 (items 1–10) reviewed and approved |
| Legal / Privacy Counsel | Sections 2–3 (items 11–30) reviewed and approved |
| Engineering Lead | Sections 1, 4 (items 1–10, 31–36) reviewed and approved |
| Vendor Contracts | Section 5 (items 37–40) confirmed signed |

---

*See also: [AI Risk Register](ai-risk-register.md) · [PII and Data Privacy](pii-and-data-privacy.md) · [Responsible AI Principles](responsible-ai-principles.md) · [Security, Legal, Governance from Day One](/strategy-memos/security-legal-governance-from-day-one.md)*
