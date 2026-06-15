# AI Integration Assistant (Aquera-Style, Synthetic)

**One-line summary:** AI-assisted identity integration that reduces provisioning connector setup from days of manual configuration to hours of AI-generated configuration with structured human review — applied to the identity and access management integration space.

*This case study is entirely synthetic and inspired by, but not describing, any specific internal system at Aquera Labs or any other company. All metrics, configurations, and scenarios are illustrative examples designed to demonstrate product design thinking. No confidential information is included.*

---

## Context and Background

Identity and access management integrations are a quiet but significant bottleneck in enterprise software deployment. When an enterprise customer adopts a new SaaS platform, they need to connect it to their identity provider (IdP) — Okta, Azure AD, Entra ID, or a legacy LDAP system — so that users are automatically provisioned and deprovisioned as they join, change roles, or leave the organisation.

The problem is that every SaaS application exposes its user provisioning API slightly differently. Some use SCIM 2.0 correctly. Some implement SCIM with non-standard extensions. Some have proprietary provisioning APIs. Some require custom attribute mappings that vary by customer. Building a connector between an IdP and a new SaaS application typically requires a solutions engineer to read the SaaS API documentation, understand the customer's IdP configuration, manually write the connector configuration (sometimes hundreds of lines of JSON or XML), test it against the customer's sandbox, debug discrepancies, and deploy.

This process takes anywhere from one day to two weeks, depending on the SaaS application's documentation quality and the customer's IdP configuration complexity. It creates a meaningful time-to-value delay for every new customer and a recurring cost every time a new application is added to a customer's integration stack.

---

## Problem Statement

Enterprise identity integration projects are delayed and expensive because connector configuration is a manual, expert-intensive process that does not scale with the number of SaaS applications a customer deploys. Solutions engineers spend disproportionate time on low-judgment configuration work that is necessary but not differentiating.

The downstream effects: longer customer time-to-value, higher professional services cost, higher error rates from manual configuration (misconfigured attribute mappings cause provisioning failures that are difficult to diagnose), and a bottleneck on growth as the integration catalog expands.

---

## Users and Personas

**The Solutions Engineer (Primary):** Owns the technical deployment of identity integrations for enterprise customers. Deep expertise in IdP configuration and SCIM/LDAP protocols. Spends 40–60% of their time on configuration work that follows predictable patterns. Wants to spend their time on the 20% of integrations that are genuinely novel or complex, not the 80% that are variations on known patterns.

**The IT/IAM Administrator (Customer Side, Primary):** Responsible for identity governance at the enterprise customer. Needs connectors to be accurate — a misconfigured attribute mapping that gives users the wrong permissions is a security incident. Values thoroughness over speed, but also values time-to-value because their internal stakeholders are waiting.

**The Customer Success Manager (Secondary):** Owns the customer relationship through the deployment phase. Cares about time-to-value as a proxy for customer health. Receives the first call when a provisioning failure creates a user access issue.

---

## Existing Workflow Pain

Current state for a new SaaS application integration:
1. SE reads the SaaS application's SCIM or provisioning API documentation (1–3 hours)
2. Reviews the customer's IdP configuration in Okta/Azure AD (30–60 min)
3. Manually writes the connector configuration JSON (2–8 hours depending on complexity)
4. Creates a test plan based on user attribute scenarios (1–2 hours)
5. Deploys to sandbox and tests provisioning for 5–10 test user scenarios (2–4 hours)
6. Debugs attribute mapping errors (variable — 0 to 8 hours)
7. Documents the connector configuration for the customer and for internal reuse (1–2 hours)
8. Deploys to production with customer IT approval

Total: 8–28 hours per integration. For a simple, well-documented SaaS application with SCIM 2.0, the lower end. For a legacy or proprietary API, the higher end.

---

## Product Thesis

An AI assistant that can read the SaaS application's API documentation, review the customer's IdP configuration, generate a validated connector configuration as a starting draft, and produce a test plan with synthetic user scenarios — with SE review and approval before any deployment — will reduce the routine configuration work from hours to minutes, shift SE time to review and judgment rather than creation, and reduce error rates by eliminating manual transcription errors in the configuration.

The AI does not replace the SE's expertise. It generates the first draft that would have taken hours to write, so the SE can spend their time on review, edge cases, and the genuinely complex scenarios.

---

## Solution Overview

**Documentation Ingestion:** The system ingests the SaaS application's provisioning API documentation (SCIM spec, REST API docs, attribute schema) via URL or file upload. A parsing pipeline extracts: supported SCIM attributes, custom attribute extensions, required vs. optional fields, rate limits, authentication method, known quirks or non-standard implementations.

**IdP Profile Analysis:** The system reads the customer's IdP configuration (Okta application config, Azure AD enterprise application settings, or LDAP schema) and extracts: user attribute schema, group membership model, provisioning flow (push vs. pull), existing attribute mappings.

**Connector Configuration Generation:** The AI generates a connector configuration draft: SCIM endpoint URL, authentication configuration, attribute mapping from IdP fields to application fields, provisioning scope (which users/groups to sync), and known issue flags. The output is a structured JSON configuration with inline comments explaining each mapping decision and flagging any fields where the mapping is ambiguous.

**Test Plan Generation:** For each connector configuration, the system generates a test plan: 8–12 synthetic user scenarios covering provisioning, deprovisioning, group membership change, attribute update, and edge cases (user with null optional field, user with special characters in name, etc.). Each scenario includes: input state, expected output, and a pass/fail criterion.

**SE Review Interface:** The SE reviews the configuration and test plan in a structured interface: they can approve, edit, or flag each attribute mapping individually, override any generated value, and add custom test cases. Ambiguous mappings are highlighted with the AI's reasoning and the available options.

**Sandbox Deployment and Test Execution:** After SE approval, the system deploys the connector to the sandbox environment and executes the test plan, reporting pass/fail per scenario with raw API response logs. The SE reviews test results before production deployment is enabled.

**Configuration Documentation:** After successful sandbox testing, the system generates documentation: a summary of the connector configuration, the attribute mapping table, known limitations, and a deployment runbook. This documentation is stored in the integration catalog for reuse on similar future integrations.

---

## AI-Native Layer

- **API documentation parsing:** Structured extraction from SemiStructured HTML/PDF documentation using a document parsing pipeline with domain-specific extraction prompts. Extracts SCIM schema, authentication details, and known non-standard implementations.
- **Attribute mapping inference:** Given the IdP attribute schema and the application's supported fields, the AI infers the most likely mapping based on: field name similarity, field type compatibility, examples from the integration catalog for similar application types, and explicit documentation guidance where available.
- **Ambiguity detection:** The configuration generator flags any mapping where the inference confidence is below threshold or where multiple valid mappings exist, presenting the options with reasoning for SE decision.
- **Test scenario generation:** A domain-specific generation prompt produces synthetic test user records and the expected provisioning outcomes, covering standard cases and the edge cases most likely to surface misconfiguration.
- **Integration catalog RAG:** A retrieval system over all previously built connectors allows the model to surface: "This application is similar to [prior application] — the attribute mapping for 'department' in that integration used this approach, and it passed all test cases."

---

## Workflow and Agent Architecture

```
[Inputs]
SaaS API Docs (URL or file) → Doc Parser → Application Schema Index
Customer IdP Config (Okta/Azure AD export) → IdP Profile Extractor
Integration Catalog (historical connectors) → Catalog RAG Index

        ↓ (integration request)

[Configuration Generation Pipeline]
1. Application Schema Agent: Parse docs → extract SCIM schema + quirks
2. IdP Profile Agent: Parse IdP config → extract attribute schema + model
3. Catalog Retrieval Agent: Find similar prior integrations from catalog
4. Mapping Inference Agent: Generate attribute mappings + flag ambiguities
5. Config Assembly Agent: Assemble JSON config with inline comments
6. Test Plan Agent: Generate 8–12 synthetic test scenarios

        ↓

[SE Review Interface]
Per-mapping: approve / edit / override
Ambiguity resolution: select from options or write custom mapping
Custom test case addition

        ↓ (SE approval)

[Sandbox Deployment]
Deploy connector config to sandbox environment
Execute test plan → pass/fail per scenario + raw API logs
SE reviews results

        ↓ (sandbox pass)

[Documentation Generation]
Generate: attribute mapping table, known limitations, deployment runbook
Store in integration catalog for future reuse

        ↓ (SE approval)

[Production Deployment]
Requires explicit SE sign-off + customer IT approval
No autonomous production deployment
```

---

## Key Product Decisions

**1. No production deployment without SE sign-off and customer IT approval.**
The most sensitive moment in identity integration is the transition to production. A misconfigured production connector can cause provisioning failures that lock users out of applications or, worse, grant excessive permissions. The system cannot initiate production deployment autonomously. Both the SE and the customer IT administrator must explicitly approve. This is a hard guardrail with no exception path.

**2. Ambiguity is surfaced, not resolved silently.**
The temptation is to always pick the most likely mapping, even when confidence is low, and let the SE catch errors in testing. Testing showed that silent low-confidence mappings that were wrong were more damaging to SE trust than surfaced ambiguities. Every mapping below the confidence threshold is presented with alternatives and reasoning. The SE makes the call. This is slower but produces higher quality and more trusted output.

**3. The integration catalog is the primary training signal.**
The value of the system compounds with every integration built. Each approved connector configuration, test result, and SE override is added to the catalog. The mapping inference agent retrieves similar prior integrations before generating new mappings. This creates a data flywheel: the more integrations the team builds with the system, the better the system becomes at generating accurate mappings for new integrations.

**4. Test coverage gates sandbox deployment, not just presence.**
Early designs allowed sandbox deployment as soon as the SE approved the configuration, regardless of test coverage. Testing revealed that SEs were approving configurations quickly and then spending hours debugging in sandbox. Adding a test plan generation step that runs before sandbox deployment, and requiring all high-priority test cases to pass before production deployment is enabled, shifted debugging time left and reduced total integration time.

---

## Metrics and Success Criteria

**Primary (Illustrative — Synthetic):**
- Integration setup time: target 50% reduction (e.g., from median 12 hours to median 6 hours)
- Provisioning error rate: target 75% reduction (errors per 1,000 users provisioned)
- First-time sandbox test pass rate: target >80% of integrations pass all high-priority test cases on first sandbox run

**Secondary:**
- Configuration override rate per mapping (which mapping types generate the most SE corrections — prioritises model improvement)
- Integration catalog coverage (% of new integrations that have at least one similar prior integration in the catalog for reference)
- SE self-reported confidence in generated configurations (monthly survey)

**Leading indicators:**
- Documentation parsing accuracy (% of extracted schema fields matching manual SE review, measured monthly on sample)
- Mapping inference confidence distribution (are low-confidence flags occurring at the expected frequency?)
- Test plan scenario coverage (does every test plan include at minimum one provisioning, one deprovisioning, and one attribute update scenario?)

---

## Risks and Guardrails

**Incorrect attribute mapping deployed to production:** The most severe failure mode. Mitigated by: SE approval at mapping level, sandbox test plan execution before production deployment, customer IT approval as a second gate, and post-deployment monitoring of provisioning success rates.

**Hallucinated API capabilities:** The model may infer that an application supports a SCIM attribute that it does not actually support, based on training data or documentation ambiguity. Mitigated by: grounding the mapping inference on the parsed schema, not training data; flagging any field not explicitly present in the parsed schema as "not confirmed in documentation."

**Documentation staleness:** SaaS application provisioning APIs change. A connector configuration built on last year's documentation may fail silently when the application updates its API. Mitigated by: documentation ingestion timestamp surfaced in the integration catalog, freshness alerts when a connector configuration is more than 6 months old, and a re-validation workflow triggered by provisioning failure alerts.

**Over-confidence in generated configurations:** SEs who trust the system too much may reduce review thoroughness over time, especially after repeated successful integrations. Monitored via override rate decay (if override rate drops significantly, it may indicate reduced review engagement rather than improved model quality). Addressed via periodic calibration exercises: SE reviews a sample of past configurations and rates quality.

---

## GTM and Adoption Considerations

The natural buyer is the head of professional services or solutions engineering at an identity platform company or an enterprise IT services company with a large SaaS integration practice. The value proposition is clear: reduce the cost of each integration by 50% without reducing quality, which allows the team to take on more customers at the same headcount or increase margins.

The pilot design is integration-level, not account-level: run 20 integrations through the AI assistant and 20 through the current manual process, with SEs blind-reviewing the output quality. The pilot does not require changing the SE workflow — they still review and approve everything — so adoption risk is low.

---

## What I Would Build First (MVP Wedge)

Attribute mapping inference for SCIM 2.0 applications connected to Okta only. No test plan generation, no documentation generation, no sandbox execution. Just: parse the application's SCIM schema + the customer's Okta attribute schema → generate a draft attribute mapping table → SE reviews and approves.

This is the highest-leverage part of the workflow (the mapping table is where most of the time is spent) and the most well-defined (SCIM 2.0 has a standard schema, which makes parsing more reliable). Once the mapping inference is validated, test plan generation and documentation automation are straightforward extensions.

---

## Lessons and Tradeoffs

The core tension in this product is between automation speed and the trust cost of errors. Identity integrations are high-stakes: a wrong attribute mapping can be a security incident. This means that the product's primary obligation is not to be fast — it is to be reliably safe to approve. Every design decision should be evaluated through that lens first.

The integration catalog as a compounding intelligence layer is the most important architectural decision. Without it, the AI assistant is a one-off tool. With it, every integration makes the next one better. In enterprise products where integration work is ongoing and cumulative, the flywheel is the moat.

The lessons from this product design informed how I think about human-in-the-loop governance more broadly: approval steps should be at the points of highest consequence, not distributed uniformly across the workflow. In identity integration, the consequence points are attribute mapping decisions and production deployment — those get explicit human approval. Test execution and documentation generation can run autonomously.

---

## Related Resources

- [AI Integration Assistant PRD](/ai-prds/ai-integration-assistant-prd.md)
- [HITL Design Framework](/hitl-governance/human-in-the-loop-design.md)
- [Confidence Thresholds](/hitl-governance/confidence-thresholds.md)
- [Audit Logs and Decision Trails](/hitl-governance/audit-logs-and-decision-trails.md)
