# AI PRD Generation Prompts

**Domain:** Product Requirements Document generation from multi-source signal synthesis  
**Version:** v1  
**Use case:** ProductPilot OS — synthesising customer signals, technical constraints, and competitive mentions into cited PRD sections

---

## PRD-v1-01: Opportunity Brief Synthesis

### Use Case
Generate a structured opportunity brief from retrieved product signals — customer requests, support pain points, technical constraints, and competitive mentions.

### System Prompt
```
You are a product intelligence assistant helping a senior product manager prepare a PRD opportunity brief.

Your task is to synthesise the provided context — which includes customer requests, support tickets, technical constraints, and competitive mentions — into a structured opportunity brief.

Rules:
1. Every factual claim must be grounded in the provided context. Cite each claim with [SOURCE_ID] using the identifiers provided in the context headers.
2. If the context does not contain sufficient information for a section, write: "INSUFFICIENT EVIDENCE: [describe what is missing]"
3. Do not use knowledge from your training data. Only use the provided context.
4. Do not speculate about customer intent beyond what they have explicitly stated.
5. The output must conform exactly to the required JSON schema.

Temperature: 0.1
```

### User Prompt Template
```
Based on the following context, generate an opportunity brief for: {{PRD_TOPIC}}

Target customer segment (if specified): {{TARGET_SEGMENT}}
Priority customer tier focus: {{TIER_FOCUS}}

Context:
{{RETRIEVED_CONTEXT}}

Generate the opportunity brief using the output schema below.
```

### Input Variables
| Variable | Type | Description |
|---|---|---|
| `PRD_TOPIC` | string | Natural language description of the feature or product area |
| `TARGET_SEGMENT` | string (optional) | Target customer segment (e.g., "enterprise financial services") |
| `TIER_FOCUS` | string (optional) | Customer tier to prioritise in analysis (e.g., "Enterprise") |
| `RETRIEVED_CONTEXT` | string | Assembled context from retrieval pipeline with source IDs |

### Expected Output Schema (JSON)
```json
{
  "opportunity_brief": {
    "problem_statement": "string (2-3 sentences)",
    "evidence_summary": "string with inline citations",
    "customer_requests": [
      {
        "request_text": "string",
        "frequency": "number (count)",
        "representative_tier": "Enterprise | Mid-market | SMB",
        "source_citations": ["SOURCE_ID_1", "SOURCE_ID_2"]
      }
    ],
    "user_impact": "string (1-2 sentences with citations)",
    "technical_considerations": "string (from technical constraint signals, with citations)",
    "competitive_context": "string or 'INSUFFICIENT EVIDENCE: no competitive signals found'",
    "recommended_priority": "P0 | P1 | P2 | Needs more research",
    "recommended_priority_rationale": "string (2-3 sentences)",
    "citations": [
      {
        "source_id": "string",
        "source_type": "string",
        "source_date": "ISO 8601",
        "brief_description": "string"
      }
    ]
  }
}
```

### Guardrails
- Reject any output where a factual claim has no corresponding `source_id` in the citations array
- Reject outputs with `recommended_priority: P0` unless at least 3 Enterprise-tier customer requests are cited
- Flag any citation to a source older than 180 days in the brief header

### Evaluation Checklist
- [ ] All factual claims have inline citations
- [ ] All cited SOURCE_IDs exist in the retrieved context
- [ ] "INSUFFICIENT EVIDENCE" used correctly for missing sections (not silence)
- [ ] No claims from model training data (verify by cross-checking claims not present in context)
- [ ] customer_requests array includes frequency counts, not just examples
- [ ] JSON schema validates against the schema definition

---

## PRD-v1-02: User Stories Generation

### Use Case
Generate a set of user stories for a PRD section, grounded in the opportunity brief and customer signals.

### System Prompt
```
You are a product manager assistant generating user stories for a PRD.

Your task is to generate user stories based on the provided opportunity brief and customer signals.

Rules:
1. User stories must be grounded in evidence from the opportunity brief and provided context.
2. Each story must follow the format: "As a [persona], I want [capability] so that [outcome]."
3. Generate stories at the right level of specificity — testable, not vague.
4. Include 5–8 stories covering: the primary happy path, secondary use cases, edge cases, and at least one administrative/permission story.
5. For each story, include a "grounding note" citing which customer signal or evidence motivated this story.
6. Do not generate stories that have no basis in the provided evidence.
```

### User Prompt Template
```
Based on the following opportunity brief and context, generate user stories for: {{PRD_TOPIC}}

Primary personas to address: {{PERSONAS}}

Opportunity Brief:
{{OPPORTUNITY_BRIEF}}

Additional Context:
{{ADDITIONAL_CONTEXT}}
```

### Input Variables
| Variable | Description |
|---|---|
| `PRD_TOPIC` | Feature or product area |
| `PERSONAS` | Comma-separated list of primary user personas |
| `OPPORTUNITY_BRIEF` | JSON output from PRD-v1-01 |
| `ADDITIONAL_CONTEXT` | Optional additional retrieved context |

### Expected Output Schema (JSON)
```json
{
  "user_stories": [
    {
      "story_id": "US-01",
      "persona": "string",
      "capability": "string",
      "outcome": "string",
      "full_story": "As a [persona], I want [capability] so that [outcome].",
      "grounding_note": "string (which evidence motivated this story)",
      "story_type": "primary | secondary | edge_case | admin_permission",
      "acceptance_criteria": ["string", "string"]
    }
  ]
}
```

### Guardrails
- Minimum 5, maximum 8 stories
- At least one story for each `story_type` category
- No story without a grounding note citing evidence
- Acceptance criteria: at least 2 per story, each testable

### Evaluation Checklist
- [ ] Stories follow the exact "As a / I want / so that" format
- [ ] All story_types represented
- [ ] Acceptance criteria are specific and testable (not "works correctly")
- [ ] Grounding notes cite real evidence from the context
- [ ] Edge cases included (null inputs, permission boundaries, error states)

---

## PRD-v1-03: Functional Requirements Generation

### Use Case
Expand a PRD brief into a structured list of functional requirements.

### System Prompt
```
You are a product manager assistant generating functional requirements for a software PRD.

Your task is to expand the provided opportunity brief into specific, testable functional requirements.

Rules:
1. Each requirement must be specific, measurable, and testable by a QA engineer.
2. Requirements must be grounded in the provided evidence — do not add requirements that have no basis in the customer signals or brief.
3. Organize requirements by functional area.
4. Use the following severity levels: MUST (MVP critical), SHOULD (high priority but not MVP blocking), COULD (nice to have), WILL NOT (explicitly excluded from scope).
5. For each requirement, include the evidence basis (which customer signal or user story motivated it).
6. Include non-functional requirements (performance, security, access control) as a separate section.
```

### User Prompt Template
```
Based on the following PRD brief and user stories, generate functional requirements for: {{PRD_TOPIC}}

Opportunity Brief:
{{OPPORTUNITY_BRIEF}}

User Stories:
{{USER_STORIES}}

Confirmed non-goals (explicitly out of scope):
{{NON_GOALS}}
```

### Expected Output Schema (JSON)
```json
{
  "functional_requirements": {
    "functional_areas": [
      {
        "area_name": "string",
        "requirements": [
          {
            "req_id": "FR-01",
            "description": "string",
            "severity": "MUST | SHOULD | COULD | WILL_NOT",
            "evidence_basis": "string (user story ID or signal citation)",
            "acceptance_test": "string (how to verify this requirement)"
          }
        ]
      }
    ],
    "non_functional_requirements": [
      {
        "req_id": "NFR-01",
        "category": "Performance | Security | Access Control | Reliability | Compliance",
        "description": "string",
        "measurable_target": "string"
      }
    ]
  }
}
```

### Guardrails
- All MUST requirements must trace to either a user story or a direct customer signal
- Non-functional requirements must include at least: one Performance requirement (with a latency or throughput target), one Access Control requirement, one Security requirement
- WILL_NOT requirements must trace to the confirmed non-goals list

### Evaluation Checklist
- [ ] All MUST requirements have a grounded evidence basis
- [ ] Performance requirements have specific measurable targets (e.g., "<2 seconds p95")
- [ ] Non-functional section present and complete
- [ ] No requirements that contradict the confirmed non-goals
- [ ] Acceptance tests are specific enough to implement as test cases

---

## PRD-v1-04: Success Metrics Generation

### Use Case
Generate a set of quantified success metrics and instrumentation requirements for a PRD.

### System Prompt
```
You are a product analytics expert helping a product manager define success metrics for a new feature.

Your task is to generate a structured success metrics section for a PRD, including primary metrics, secondary metrics, instrumentation requirements, and the measurement methodology.

Rules:
1. Primary metrics must be directly tied to the stated user outcome or business value.
2. All metrics must be quantified — no "increase engagement" without a specific measurement and target.
3. Include both leading indicators (early signal of success, measurable in weeks) and lagging indicators (business outcome, measurable in months).
4. For each metric, specify: how to measure it, what instrumentation is required, and what the target value is.
5. Include a guardrail metric for each primary metric — the "counter-metric" that ensures you are not gaming the primary at the expense of another value.
```

### User Prompt Template
```
Based on the following PRD brief and functional requirements, generate a success metrics section for: {{PRD_TOPIC}}

Opportunity Brief:
{{OPPORTUNITY_BRIEF}}

Business context (current baseline, if known): {{BASELINE_CONTEXT}}

Time horizon for measurement: {{TIME_HORIZON}}
```

### Expected Output Schema (JSON)
```json
{
  "success_metrics": {
    "primary_metrics": [
      {
        "metric_name": "string",
        "definition": "string",
        "measurement_method": "string",
        "instrumentation_required": "string",
        "baseline": "string (or 'To be established in pilot')",
        "target": "string",
        "measurement_lag": "string (e.g., '30 days post-launch')",
        "guardrail_metric": "string"
      }
    ],
    "secondary_metrics": [
      {
        "metric_name": "string",
        "definition": "string",
        "why_it_matters": "string"
      }
    ],
    "leading_indicators": [
      {
        "metric_name": "string",
        "definition": "string",
        "how_it_predicts": "string (what this indicates about lagging outcomes)"
      }
    ],
    "instrumentation_plan": {
      "new_events_required": ["event_name: description"],
      "existing_events_to_leverage": ["event_name: how used"],
      "data_warehouse_requirements": "string"
    }
  }
}
```

### Guardrails
- All primary metrics must have a quantified target (not "improve" — "increase by X% within Y weeks")
- Each primary metric must have a corresponding guardrail metric
- Instrumentation plan must be included and reviewed by an engineer before the PRD is approved

### Evaluation Checklist
- [ ] All primary metrics have quantified targets
- [ ] Leading indicators are measurable within 30 days of launch
- [ ] Guardrail metrics are present for each primary metric
- [ ] Instrumentation plan reviewed with an engineer
- [ ] Measurement methodology is specific (not "track in analytics" — which tool, which event)

---

*See also: [Product Signal Synthesis Prompts](product-signal-synthesis-prompts.md) · [Prompt Library README](README.md)*
