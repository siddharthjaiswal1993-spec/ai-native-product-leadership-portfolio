# PII and Data Privacy in Enterprise AI

**Identification, classification, redaction, access control, retention, cross-border considerations, and customer data agreements**

---

## The Privacy Landscape for Enterprise AI

Enterprise AI products face a more complex privacy landscape than traditional SaaS for one fundamental reason: the AI system processes data that was not originally collected for AI processing.

When a company's CRM notes, customer call recordings, support tickets, and product usage data are ingested into an AI system's retrieval corpus, that data — which was collected for operational purposes — is now being processed by an AI model to generate outputs. This creates new privacy obligations that most enterprise customers have not yet fully addressed in their data governance frameworks.

As a PM, your role is to design the product with privacy as a first-class constraint, not to retrofit privacy controls after engineering complains about them. The specific obligations depend on the customer's jurisdiction, industry, and existing data processing agreements — but the design principles below apply broadly.

---

## PII Identification and Classification

### What Counts as PII

For enterprise AI products, the relevant PII categories typically include:

**Direct identifiers:**
- Names (customer contacts, employees, individuals referenced in support tickets)
- Email addresses
- Phone numbers
- User IDs that are not anonymised

**Indirect identifiers (PII when combined):**
- Job titles + company name + department (can identify an individual in a small company)
- Location + role (can narrow to a small group)

**Sensitive data categories (higher risk):**
- Financial data (salary, revenue, pricing information)
- Health data (if ingesting data from healthcare customers)
- Authentication credentials (never index these; treat as critical if encountered)
- Legal matters (litigation references, compliance findings)

### Classification Approach

Classify data by sensitivity level at the source level:

| Source Type | Default Classification | Notes |
|---|---|---|
| Support ticket body | Low-to-medium | May contain customer employee names and email addresses |
| CRM account notes | Medium | May contain contact-level details and sensitive business information |
| Call transcripts (Gong/Chorus) | High | Contains names, email addresses, and potentially sensitive business conversations |
| Product usage data | Low | Typically anonymised at user level; contact names not usually present |
| Financial/billing data | High | Revenue figures, payment terms, contract amounts |

---

## PII Redaction at Ingestion

The principle: redact or tokenise PII at the point of ingestion, before the data enters the AI processing pipeline. It is much easier to control PII at ingestion than to retroactively find and redact it from an embedding vector.

### What to Redact

**By default:**
- Contact email addresses in all text content → replace with [EMAIL]
- Contact phone numbers → replace with [PHONE]
- SSNs, credit card numbers (if encountered) → replace with [REDACTED_SENSITIVE]

**Configurable (off by default, on for specific source types):**
- Contact personal names in transcripts → replace with [PERSON] or with a consistent pseudonym per person across the document (to preserve conversational structure)
- Specific customer executive names in CRM notes → customer configures which names are redacted

**Not redacted by default (PM should confirm with customers):**
- Company names (typically not PII for organisations, though relevant to data isolation)
- Job titles and roles
- General business information

### Redaction Implementation

**Named Entity Recognition (NER):** Use a NER model at ingestion to identify person names, email addresses, phone numbers, and other PII categories. Apply redaction rules per category.

**Regex-based detection:** For structured PII (email addresses, phone numbers, SSNs, credit card numbers), regex patterns are more reliable than NER.

**Context-sensitive redaction:** In some contexts, removing a name changes the meaning of the text in ways that reduce retrieval quality. Consider pseudonymisation (replacing [Person A] with a consistent fake name like "Alex" throughout a single document) rather than full redaction.

---

## Access Control

### Principle of Minimum Necessary Access

Users should only be able to retrieve data they have a legitimate need to access. This is enforced at the retrieval layer, not just the UI layer.

**Account-level access control:**
- CSMs can retrieve data for accounts they own
- CS managers can retrieve data for all accounts in their team
- Finance team can access billing data for all accounts
- No user can retrieve data for accounts they are not authorised to see

**Implementation:** Every retrieval query includes a mandatory `authorised_account_ids` filter. This filter is populated from the user's access rights at the IAM layer, not from user input. A user cannot bypass this filter by passing a different account ID in the query — the filter is applied server-side before results are returned.

**Source-level access control:**
- Call recordings may require additional consent from call participants before being indexed and used in retrieval
- Financial data may require a "finance team" role attribute to be included in the retrieval scope

---

## Retention Policies

PII in AI systems has a lifecycle that must be managed:

**Ingested PII:** Define the retention period for indexed chunks that contain PII. Chunks should not be retained longer than the original source document's retention period.

**Generated outputs:** AI-generated outputs that contain PII (e.g., a pre-visit brief that mentions a customer contact's name) should be subject to the same retention policy as the customer data they derive from.

**Audit logs:** Audit log entries that reference PII (in content snapshots or entity names) must comply with the relevant data retention requirements.

**Right to erasure:** Enterprise customers subject to GDPR may exercise the right to erasure on behalf of individuals whose data is in the system. The product must support:
- Identification of all indexed chunks that contain data related to a specific individual (by name, email, or customer-configured identifier)
- Deletion or redaction of those chunks from the index
- Confirmation that the erasure is complete

---

## Cross-Border Data Considerations

**Data residency:** Some enterprise customers (particularly in the EU, healthcare, financial services) require that their data be processed and stored within a specific geographic region. The product must support:
- Customer selection of their data residency region at onboarding
- Hosting architecture that isolates customer data by region
- Confirmation that all AI inference is performed within the agreed region

**Model provider data residency:** If using a cloud model API (GPT-4, Claude API, etc.), the data sent to the API may cross borders. Review the model provider's data processing agreement for data residency commitments. Some enterprise providers offer data residency-compliant API endpoints.

**Sub-processor disclosure:** GDPR requires disclosure of sub-processors (companies that process data on your behalf). Your model API provider, vector store provider, and cloud infrastructure provider are all sub-processors. Disclose them in your DPA with customers.

---

## Customer Data Agreements

Before connecting any customer data source to the AI system, the customer must sign:

1. **Data Processing Agreement (DPA):** Defines the legal basis for processing customer data, data retention periods, data subject rights procedures, and sub-processor disclosure.

2. **Consent requirements for call recordings:** Many jurisdictions require two-party consent for recording calls. Before indexing call transcripts from Gong or Chorus, confirm that the customer has obtained appropriate consent from all call participants. Include a representation and warranty in the DPA.

3. **Use limitation clause:** The DPA should include a use limitation provision: customer data is used only to provide the contracted service to that customer. It is not used for training models used by other customers without explicit consent.

4. **Data breach notification:** Specify the notification timeline (GDPR requires 72 hours to authorities; most enterprise agreements specify immediate vendor notification + 72-hour customer notification).

---

*See also: [AI Risk Register](ai-risk-register.md) · [Security and Legal Review Checklist](security-legal-review-checklist.md) · [Audit Logs and Decision Trails](/hitl-governance/audit-logs-and-decision-trails.md)*
