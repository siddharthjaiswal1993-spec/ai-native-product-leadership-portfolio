# PRD: AI Integration Assistant (Synthetic — Inspired by Aquera-Style Identity Integration)

**Version:** 1.0  
**Status:** Draft for Review  
**Author:** Siddharth Jaiswal  
**Date:** 2026-06  
*Note: This PRD is entirely synthetic. It is a portfolio artifact demonstrating AI product requirements methodology applied to the identity and access management integration domain. It does not describe any specific internal system at Aquera Labs or any other company.*

---

## Problem

Building identity provisioning connectors between enterprise identity providers (IdPs — Okta, Azure AD, Entra ID) and SaaS applications is a manual, expert-intensive process. A solutions engineer must read API documentation, analyse the customer's IdP configuration, manually write a connector configuration (JSON mapping), build a test plan, and execute and debug it. The process takes 8–28 hours per integration depending on complexity.

This creates a meaningful time-to-value delay for enterprise customers, a recurring professional services cost for each integration, and a persistent risk of configuration errors (misconfigured attribute mappings that cause provisioning failures or incorrect permission assignment).

---

## Target Users

**Primary:**
- Solutions Engineers (SEs) at identity platform companies responsible for deploying identity integrations for enterprise customers. Deep protocol knowledge (SCIM, LDAP, SAML, OAuth). Spend 40–60% of time on predictable configuration work.

**Secondary:**
- IT/IAM Administrators (customer side): Responsible for identity governance. Need correct connectors. Prioritise accuracy over speed.
- Customer Success Managers: Own the customer relationship during deployment. Track time-to-value as a proxy for customer health.

---

## Jobs to Be Done

1. **SE — Configuration drafting:** "Given the application's API docs and the customer's IdP config, generate a connector configuration that I can review, not write from scratch."
2. **SE — Test planning:** "Give me the test scenarios I need to validate this connector, including the edge cases I might miss."
3. **SE — Documentation:** "After the integration is built and validated, generate the documentation so I do not have to write it manually."
4. **IT/IAM Admin:** "Review and approve the connector configuration before production deployment."
5. **Operations Leader:** "See how long integrations are taking vs. baseline and understand where time is being spent."

---

## Scope

This PRD covers:
- SaaS application API documentation ingestion and schema extraction
- Customer IdP configuration ingestion and attribute schema extraction
- Integration catalog retrieval (similar prior integrations)
- AI-generated connector configuration draft with per-mapping confidence scores
- SE review interface with per-mapping approve / edit / override
- Synthetic test plan generation for sandbox execution
- Sandbox deployment and automated test execution
- Post-validation documentation generation
- Integration catalog storage for future reference

---

## Non-Goals

This PRD does not cover:
- Production deployment without SE and customer IT approval (explicitly excluded)
- Monitoring of running connectors in production (operational monitoring — separate product area)
- SAML or OAuth federation configuration (scoped to SCIM and proprietary provisioning APIs for V1)
- Custom connector development for entirely novel protocols
- Multi-IdP federation (single IdP per integration for V1)
- Identity governance reporting or access certification
- Customer-facing configuration interface (all interactions through SE in V1)

---

## User Stories

1. **As a solutions engineer,** I want to paste or upload a SaaS application's SCIM documentation and receive an extracted schema (supported attributes, required fields, authentication method, known quirks) in under 2 minutes, so that I do not have to read 40 pages of documentation to start building.

2. **As a solutions engineer,** I want to upload the customer's IdP attribute export and receive a draft connector configuration with attribute mappings and a confidence score per mapping, so that I can focus my review on the low-confidence or flagged mappings rather than the entire configuration.

3. **As a solutions engineer,** I want every low-confidence or ambiguous mapping to be explicitly flagged with the available options and the model's reasoning, so that I can make an informed decision rather than accepting a silent default.

4. **As a solutions engineer,** I want the system to generate a synthetic test plan with 8–12 scenarios (provisioning, deprovisioning, group membership change, attribute update, edge cases), so that I do not have to write test cases manually and can focus on reviewing coverage.

5. **As a solutions engineer,** I want to deploy the connector configuration to a sandbox environment and see automated test execution results (pass/fail per scenario + raw API logs) before I approve production deployment, so that I catch errors before they affect the customer's live environment.

6. **As a solutions engineer,** I want the system to search the integration catalog for similar prior integrations when generating a new configuration, and surface relevant precedents with notes on what worked and what was customised, so that I can build on prior work rather than starting from scratch every time.

7. **As an IT/IAM administrator,** I want to review and approve the connector configuration before production deployment, including seeing the attribute mapping table in plain English (not just JSON), so that I can confirm the configuration aligns with our identity governance requirements.

8. **As an operations leader,** I want to see average integration setup time by application type and SE, baseline vs. post-launch, so that I can measure the efficiency improvement and identify where the AI assistant is and is not adding value.

---

## Functional Requirements

### Documentation Ingestion and Schema Extraction
- Input: URL to documentation, file upload (PDF/HTML/Markdown), or direct text paste
- Extraction output: structured JSON containing: scim_attributes (array), custom_extensions (array), required_fields (array), auth_method (string), rate_limits (object), known_quirks (array of strings), documentation_url, extraction_timestamp
- Extraction runtime: <2 minutes for standard documentation
- Extraction confidence score per field: high / medium / low based on how explicitly the field is documented
- Fields not explicitly documented flagged as "not confirmed in documentation — verify before use"

### IdP Configuration Ingestion
- Input: Okta application export (JSON), Azure AD enterprise application export (JSON), or manual attribute schema entry
- Extraction output: user_attributes (array with type and example value), group_model (string: push-based or pull-based), provisioning_scope (string), existing_mappings (array, for accounts with existing partial configuration)
- Validation: verify that all required SCIM base attributes (userName, displayName, emails, active) are present in the IdP attribute schema

### Integration Catalog Retrieval
- Before generating a new connector configuration, retrieve similar prior integrations from the catalog
- Similarity: semantic similarity on application type and API protocol (not just exact match)
- Surface: top-3 most similar prior integrations with: application name (may be anonymised), mapping approach for similar fields, known issues encountered, SE who built it (internal only)
- Catalog grows with each completed and validated integration

### Connector Configuration Generation
- Generate connector configuration JSON based on: extracted application schema + customer IdP attributes + catalog precedents
- Per-mapping output:
  - idp_attribute: source field name
  - app_attribute: target field name
  - mapping_type: direct / transform / constant / complex
  - confidence: high / medium / low
  - reasoning: one-sentence explanation of why this mapping was chosen
  - alternatives: if confidence is medium or low, list alternative mappings with brief rationale
  - flagged: boolean — true if mapping requires SE decision
- Any application attribute with no clear IdP counterpart: flagged as "unmapped — action required"
- Any IdP attribute with no clear application counterpart: listed in "unmapped IdP attributes" section

### SE Review Interface
- Mapping-by-mapping display: green (high confidence, pre-approved), yellow (medium confidence, flagged), red (low confidence, requires SE decision)
- SE can: approve (no change), edit (update the mapping), or override (choose from alternatives or write custom)
- Batch-approve all green (high confidence) mappings; individual review required for yellow and red
- Approving a yellow or red mapping requires SE to select from provided alternatives or enter a custom value — cannot "approve" without making a choice
- Full configuration JSON preview before sandbox deployment
- Customer IT review link: generates a plain-English attribute mapping table (not JSON) for customer IT admin review and approval

### Test Plan Generation
- For each connector configuration, generate a test plan with 8–12 synthetic user records and expected outcomes
- Required scenario coverage: at least one of each type — (1) new user provisioning, (2) user deprovisioning, (3) group membership change, (4) required attribute update, (5) optional attribute with null value, (6) user with special characters in name/email
- Each scenario: input state (JSON user object), expected provisioning outcome, pass/fail criterion, failure detection method

### Sandbox Deployment and Test Execution
- Requires SE approval of full connector configuration before sandbox deployment
- Deploys configuration to sandbox environment via integration platform API
- Executes test plan: sends SCIM requests per scenario, captures API responses
- Reports pass/fail per scenario + raw API request and response logs
- High-priority scenarios (new user provisioning, deprovisioning) must pass before production deployment is enabled
- Medium-priority scenarios can have failures with SE documented exception before production deployment

### Documentation Generation
- Triggered after sandbox test pass and before production deployment approval
- Generates: attribute mapping table (plain English), authentication configuration summary, known limitations and edge cases, deployment runbook (step-by-step production deployment guide), change history (who reviewed, when, which mappings were overridden and why)
- Stored in integration catalog with metadata for future retrieval

### Production Deployment
- Requires: (1) SE explicit approval, (2) Customer IT administrator explicit approval
- System surfaces the plain-English mapping table to the customer IT administrator for review via a shareable review link (no account required, but authenticated with a secure token)
- Customer IT administrator approves or returns with feedback
- No production deployment action is initiated by the system — SE initiates via a "Deploy to Production" button after both approvals are received

---

## AI Capabilities

| Capability | Method | Model Class |
|---|---|---|
| Documentation parsing | Structured extraction from HTML/PDF/Markdown | Medium LLM with structured output |
| Attribute mapping inference | Semantic matching + catalog RAG + generation | Frontier LLM (medium) |
| Confidence scoring | Classification on mapping quality signals | Small LLM or rule-based classifier |
| Test plan generation | Structured generation with scenario templates | Medium LLM with JSON schema output |
| Documentation generation | Structured generation from approved config | Medium LLM |
| Catalog retrieval | Semantic similarity over prior integrations | Embedding model + vector search |

---

## Model Choice Considerations

**Attribute mapping inference:** Semantic understanding of API attribute names and their likely correspondence across systems. Requires a model with good semantic similarity capabilities and ability to reason about ambiguous mappings. Medium-to-large LLM depending on complexity of the integration.

**Documentation parsing:** Primarily an extraction task. Structure-from-unstructured. Can be handled by a medium LLM with a well-defined output schema. Not a reasoning task — prioritise accuracy and consistency over creativity.

**Test plan generation:** Structured output generation from a fixed scenario template. Small-to-medium LLM. JSON schema enforcement. The scenario types are known; the test data is synthetic but must be realistic.

**Catalog retrieval:** Embedding model for semantic similarity over prior integration descriptions. Not an LLM task.

**Latency requirements:** Documentation parsing (<2 min) and configuration generation (<5 min) are acceptable wait times for an SE workflow. These are not real-time interactions. Use higher-quality models at the cost of latency.

---

## Prompting, RAG, and Tool-Use Design

### Attribute Mapping Inference Prompt
```
System: You are an identity integration specialist. Your task is to generate an attribute 
mapping configuration between an identity provider and a SaaS application provisioning API.

Given:
- The SaaS application's supported attribute schema (with required/optional flags and types)
- The customer's IdP user attribute schema
- Similar prior integrations from the catalog (if available)

For each SaaS application attribute, identify the best IdP attribute to map to it. 
Provide a confidence level (high/medium/low) and a one-sentence reasoning for each mapping.
Flag any mapping where you are uncertain or where multiple valid options exist.
If no IdP attribute clearly maps to a required SaaS attribute, flag it as "unmapped — action required."
Do not guess. If you cannot make a justified mapping, say so explicitly.

Output must conform to the provided JSON schema.
```

### Integration Catalog RAG
- Index: all completed and validated connector configurations, stored as structured JSON + natural language description
- Retrieval query: application type + protocol + known quirks (e.g., "SCIM 2.0 application with custom department attribute extension")
- Retrieved context: top-3 similar configurations, including mapping decisions and SE notes
- Citation: every catalog reference in the generated configuration includes the source integration ID

### Test Plan Generation Prompt
- Input: approved connector configuration JSON + application attribute schema
- Output: array of test scenarios, each conforming to the test_scenario JSON schema
- Instruction: "Generate test scenarios that cover all scenario types in the required coverage list. Use realistic but synthetic user data (no real names, emails, or IDs). Ensure at least one scenario tests each of the required coverage types."

---

## Eval Plan

### Offline Evals

**Mapping accuracy evaluation (monthly, 20 integrations reviewed by senior SE):**
- Mapping correctness rate: % of AI-generated mappings that the SE agrees with without modification — target >75%
- High-confidence mapping correctness: % of high-confidence mappings that are correct — target >90%
- Unmapped field detection: % of cases where a required application field has no IdP counterpart, correctly flagged — target >95%

**Test plan coverage evaluation (monthly, 20 test plans reviewed by senior SE):**
- Scenario type coverage: % of test plans with all required scenario types present — target >95%
- Edge case quality: SE rating of edge case scenario quality (1–5 scale) — target avg >3.5

**Documentation quality evaluation (monthly, 10 documents reviewed by SE):**
- Attribute mapping table accuracy: % of table rows that correctly reflect the approved configuration — target >98%
- Runbook completeness: % of deployment runbooks reviewed as "complete" by SE — target >85%

### Online Metrics

| Metric | Target |
|---|---|
| Config generation runtime | <5 minutes p95 |
| SE override rate (all mappings) | Tracked; target declining from baseline |
| SE override rate (high-confidence mappings) | <10% |
| Sandbox first-pass test success rate | >80% of integrations pass all high-priority scenarios on first sandbox run |
| Time to completed integration | 50% reduction vs. baseline (illustrative target) |
| Customer IT review approval rate | >90% first-attempt approval (without requiring re-review) |

---

## Human-in-the-Loop Design

| Action | Autonomy Level | Rationale |
|---|---|---|
| Documentation parsing | Automatic | No customer-facing impact; SE reviews output |
| IdP config parsing | Automatic | No customer-facing impact |
| Catalog retrieval | Automatic | Information retrieval only |
| Configuration generation | Automatic; SE review required | All mappings require SE review before sandbox deployment |
| Sandbox deployment | Requires SE approval of full config | First deployment gate |
| Test execution | Automatic after sandbox deployment | Automated test; SE reviews results |
| Production deployment | Requires SE approval + customer IT approval | Highest-consequence action; two-gate approval |
| Documentation generation | Automatic; SE review before sharing with customer | SE verifies accuracy before customer-facing delivery |

---

## Guardrails

- **No production deployment without two-gate approval:** SE + customer IT administrator. Hard system constraint. Cannot be overridden by any user role.
- **Unmapped required fields block sandbox deployment:** If any required SaaS application attribute is flagged as "unmapped — action required" and the SE has not resolved the mapping, sandbox deployment is blocked. The system surfaces the blocking unmapped fields.
- **Confidence score transparency:** Confidence scores are always visible to the SE. There is no "hide low-confidence" option.
- **Documentation timestamp and version:** Every generated document is timestamped and version-controlled. If the underlying SaaS application updates its API, the system detects documentation staleness (via periodic re-check of the documentation URL) and alerts the SE that the configuration may need re-validation.
- **Integration catalog access control:** Prior integration configurations are accessible only to SEs with appropriate permissions. Customer-specific configurations are scoped to the account team.

---

## Privacy and Security

- No customer user data is used in the integration configuration process. All test user records are synthetically generated.
- Customer IdP configuration exports may contain tenant-specific metadata. These are stored in the customer's dedicated workspace and not cross-referenced with other customers' configurations.
- API tokens for sandbox deployment are stored encrypted, never logged in plaintext, and rotated after integration completion.
- Documentation shared with customer IT administrators is sent via a secure tokenised link with 7-day expiry and single-use access logging.
- Integration catalog entries for completed integrations retain the configuration schema but strip any customer-identifying metadata before indexing for cross-customer retrieval.
- SOC 2 Type II controls required for the integration platform hosting this system.

---

## Rollout Plan

**Phase 1 — Internal Beta (Weeks 1–6):**
- 3 SEs, SCIM 2.0 applications connected to Okta only
- Mapping inference for standard SCIM attributes only (no custom extensions)
- Manual test plan creation (test plan generation not in scope for Phase 1)
- Measure: SE override rate, mapping correctness (SE review)

**Phase 2 — Expanded Beta (Weeks 7–14):**
- 10 SEs, full SCIM + proprietary API support
- Test plan generation added
- Integration catalog retrieval active (seeded with Phase 1 completed integrations)
- Sandbox deployment and automated test execution

**Phase 3 — GA (Weeks 15–24):**
- All SEs on-platform
- Azure AD and Entra ID support added
- Customer IT review workflow
- Documentation generation
- Operations leader dashboard

**Success gate for Phase 3:** SE override rate for high-confidence mappings <10%, sandbox first-pass test success rate >80%, average integration setup time reduced by 40% vs. baseline (illustrative target).

---

## Success Metrics

**6-month targets (post GA):**
- Integration setup time: 50% reduction vs. baseline (illustrative, synthetic target)
- Provisioning error rate (production): 75% reduction vs. baseline (illustrative, synthetic target)
- SE time spent on routine configuration work: 60% reduction (self-reported)
- Integration catalog coverage: >80% of new integrations have a similar prior integration in the catalog
- Zero production incidents caused by a connector configuration deployed without both required approvals

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Incorrect mapping deployed to production | Low (due to two-gate approval) | Critical | Two-gate approval; sandbox test requirement; post-deploy monitoring |
| Hallucinated application capabilities | Medium | High | Grounding on parsed documentation only; flag fields not in parsed schema |
| Documentation staleness | Medium | High | Periodic re-check of documentation URL; freshness alerts |
| SE over-trust reducing review quality | Medium | High | Monitor override rate decay; periodic calibration exercises |
| Integration catalog data leakage | Low | High | Customer-scoped access control; metadata stripping before cross-customer indexing |

---

## Open Questions

1. How should the system handle integrations where the SaaS application does not have a public API documentation URL? (Manual schema entry? Direct API interrogation?)
2. What is the right threshold for flagging a mapping as "medium" vs. "low" confidence? Is this configurable by the SE or fixed at the platform level?
3. Should the system generate a full attribute mapping table in plain English as the default view, with JSON as a secondary view, or vice versa?
4. For the customer IT review step, should the system support async review (email link + digital signature) or synchronous review (in-app with account creation)? Async is simpler but harder to track; synchronous has better audit trail.
5. Should the integration catalog include anonymised failure records (integrations where certain mappings failed) to help future SEs avoid known failure patterns?

---

*See also: [AI Integration Assistant Case Study](/case-studies/ai-integration-assistant-aquera-style.md) · [Confidence Thresholds](/hitl-governance/confidence-thresholds.md) · [Audit Logs and Decision Trails](/hitl-governance/audit-logs-and-decision-trails.md)*
