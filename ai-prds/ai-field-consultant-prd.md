# PRD: AI Field Consultant for Franchise Operations

**Version:** 1.0  
**Status:** Draft for Review  
**Author:** Siddharth Jaiswal  
**Date:** 2026-06  
*Note: All data, personas, and scenario examples are synthetic. This is a portfolio artifact demonstrating AI product requirements methodology.*

---

## Problem

Franchise field consultants arrive at location visits without a synthesised view of the operational signals that matter most to that location. Pre-visit preparation is manual, time-consuming, and inconsistent across the consultant team. Post-visit action tracking is not systematic, making it difficult to verify whether visit recommendations produced measurable outcomes.

The result: field visits are less impactful than they should be, operator experience suffers from repetitive information requests, and the franchisor loses the institutional learning that could compound across the network.

---

## Target Users

**Primary:**
- Field consultants (FCs) at franchise networks with 200+ locations. Typically 5–10 years of franchise operations experience. High domain expertise, moderate technical literacy. Mobile-first during visit hours.

**Secondary:**
- Franchise operations directors who manage FC teams and need visibility into visit quality and action completion rates.
- Franchisee operators who receive visit action items and want clear, trackable follow-up.

---

## Jobs to Be Done

1. **FC — Pre-visit:** "Give me a complete picture of this location before I walk in the door, without me having to pull it from four different systems."
2. **FC — During visit:** "Let me ask a quick question about this location's support history without interrupting my conversation with the operator."
3. **FC — Post-visit:** "Help me turn my visit notes into structured action items that the operator can acknowledge and track."
4. **Operations Director:** "Show me which locations are getting insufficient attention and whether the AI brief is improving visit quality across the team."
5. **Franchisee Operator:** "Receive a clear action plan from my field consultant visit with due dates I can acknowledge."

---

## Scope

This PRD covers:
- Pre-visit brief generation (automated, T-24 hours before scheduled visit)
- In-visit natural language Q&A (on-demand, location-scoped)
- Post-visit voice note transcription and action item structuring
- FC review and approval of all action items before operator delivery
- Action item tracking and completion verification
- Operations director reporting dashboard

---

## Non-Goals

This PRD does not cover:
- Real-time video analysis during visits
- Automated operator communication without FC approval (explicitly excluded)
- Performance management or FC evaluation
- Financial reporting or royalty management
- Consumer-facing features
- Integration with non-franchise-operations data sources (e.g., supply chain, marketing platforms) — future scope

---

## User Stories

1. **As a field consultant,** I want to receive a pre-visit brief on my phone 24 hours before each scheduled visit, so that I arrive at every location with a complete, current operational picture without spending an hour on manual research.

2. **As a field consultant,** I want the brief to include citations for every key claim (e.g., "LMS completion rate is 62%, last updated 6 hours ago"), so that I can present findings to operators with confidence and verify data when challenged.

3. **As a field consultant,** I want to ask natural language questions about the location during the visit without switching applications (e.g., "What were the last three support tickets about?"), so that I can maintain conversational flow with the operator.

4. **As a field consultant,** I want to capture my visit notes by voice and have them automatically structured into action items with suggested owner and due date, so that I spend less time on post-visit administration.

5. **As a field consultant,** I want to review and edit all AI-drafted action items before they are sent to the operator, so that I can ensure accuracy and tone before any operator-facing communication goes out.

6. **As a franchise operations director,** I want to see which FCs are opening and acting on pre-visit briefs, so that I can identify coaching opportunities and correlate brief usage with visit quality scores.

7. **As a franchisee operator,** I want to receive a clear, structured action plan after each visit with my name or role as owner for each item and a specific due date, so that I know exactly what to do and when.

8. **As a franchise operations director,** I want to see action item completion rates by location and FC over rolling 90 days, so that I can identify where follow-through is strong and where it needs support.

---

## Functional Requirements

### Pre-Visit Brief Generation
- System pulls scheduled visits from the franchise management platform 24 hours before each visit
- For each visit, system retrieves data from: LMS (training completion), mystery shop (scores + trend), support (ticket volume + top categories, 90-day window), sales (performance vs. network average, last 2 periods), prior visit notes (last 2 visits)
- Brief is generated in under 30 seconds and pushed to the FC's mobile app
- Brief includes: overall location summary (3–5 sentences), signal-by-signal breakdown with citations, top 3 recommended focus areas for the visit, open action items from the last visit
- Audio summary (TTS, 2–3 minutes) generated from the brief text
- Data freshness timestamp displayed per signal source; alert if any source is >48 hours stale

### In-Visit Q&A
- FC activates via voice trigger on mobile app
- Natural language question processed via location-scoped RAG (retrieval limited to the current location's indexed data)
- Response delivered in <2 seconds (target p95)
- Response includes source attribution (e.g., "Based on Zendesk ticket #4521, opened Nov 12")
- Session is logged for quality monitoring (not surfaced to FC in real time)

### Post-Visit Action Structuring
- FC starts post-visit mode (tap or voice trigger) after visit completion
- Voice note captured and transcribed using Whisper-class transcription
- Extracted action items structured into JSON schema: action_title, description, owner_role, due_date_suggested, priority, related_signal
- FC reviews on a per-action-item basis: approve, edit, or delete
- Approved action items staged for operator delivery

### FC Review and Approval
- All action items require FC review and explicit approval before operator delivery
- Approval interface: swipe right to approve, swipe left to edit, long press to delete
- Batch approval available for low-priority items after individual review
- No action item can be delivered to an operator without FC approval (hard system constraint)
- Time from visit end to FC approval tracked (target median: <2 hours)

### Action Item Tracking
- Approved action items sent to operator via in-app notification and email
- Operator required to acknowledge each action item (read receipt tracked)
- System sends automated reminders at: T+7 days, T+14 days, and T-3 days before due date
- Open action items surfaced in the next pre-visit brief for that location
- Completion marked by operator or FC; completion evidence (photo, note) optional but encouraged

### Operations Director Dashboard
- Brief open rate per FC (last 30 days)
- Action item completion rate by location and FC (last 90 days)
- Visit score trends by FC (if scoring system is integrated)
- Locations with zero pre-visit briefs opened in last 30 days (flagged for attention)
- Network-wide signal health summary (average LMS completion, mystery shop trend, support volume trend)

---

## AI Capabilities

| Capability | Method | Model Class |
|---|---|---|
| Signal retrieval | Hybrid RAG (dense + sparse) with metadata filtering | Embedding model (medium) |
| Brief generation | Prompted generation with citation enforcement | Frontier LLM (large) |
| TTS audio summary | Text-to-speech conversion | TTS model |
| In-visit Q&A | Location-scoped RAG + prompted response | Frontier LLM (fast, low latency) |
| Voice transcription | Whisper-class ASR | ASR model |
| Action item extraction | Structured extraction prompt with JSON schema output | Frontier LLM (medium) |

---

## Model Choice Considerations

**Brief generation:** Requires high-quality reasoning to synthesise multiple signal sources into coherent, accurate prose. Use a frontier model (GPT-4 class or equivalent). Cost justified by the high leverage of the pre-visit brief.

**In-visit Q&A:** Latency is the primary constraint (<2 seconds p95). Use a smaller, faster model (e.g., a distilled or fine-tuned model) with tight context window (location-scoped retrieval). Quality slightly lower than brief generation is acceptable because the FC can ask a follow-up question if the first answer is incomplete.

**Action item extraction:** Structured output task. JSON schema output. Medium model sufficient. Consider prompt caching if similar extraction patterns repeat across visits.

**TTS:** Standard TTS API sufficient. Voice quality matters for mobile listening during commute. Use a natural-sounding voice with good performance on operational vocabulary (franchise-specific terms, location names).

---

## Prompting and RAG Design

### Brief Generation Prompt Architecture
- **System prompt:** You are a franchise operations intelligence assistant. Your task is to synthesise operational signal data for a specific franchise location into a field consultant visit brief. Every claim you make must be grounded in the provided context. Do not generate any claim that is not supported by the retrieved data. If a signal category has no data, explicitly note "no data available for this category."
- **Context window:** Retrieved chunks from all signal sources for the location, ordered by recency within category, with source metadata (system, date, URL or ID)
- **Output format:** Structured JSON with sections: summary, signals (array), focus_areas (array), open_actions (array), citations (array)
- **Citation enforcement:** Citation Agent validates that each claim in signals and focus_areas maps to a specific citation object. Claims without citations are rejected and the brief is re-generated.

### In-Visit Q&A Architecture
- **Context scoping:** Retrieval filtered to location_id + last_90_days metadata filter
- **Response format:** 1–3 sentences + source attribution. No verbose explanations.
- **Latency optimisation:** Short context window (top-5 retrieved chunks), small-fast model, streaming response

### Action Item Extraction Prompt
- **Input:** Raw voice transcript
- **Output schema:**
```json
{
  "action_items": [
    {
      "action_title": "string",
      "description": "string",
      "owner_role": "operator | fc | both",
      "due_date_suggested": "ISO 8601 date",
      "priority": "high | medium | low",
      "related_signal": "string (e.g., 'LMS completion below 70%')"
    }
  ]
}
```
- **Extraction instructions:** Extract only explicit commitments made during the visit. Do not infer actions that were not mentioned. Mark any action where the timeline was not mentioned as "due_date_suggested: null."

---

## Eval Plan

### Offline Evals

**Brief quality eval (monthly, human-reviewed sample of 30 briefs):**
- Citation accuracy: % of cited claims that correctly reference the stated source — target >95%
- Completeness: % of briefs that include all five required signal categories — target 100%
- Claim accuracy: % of factual claims in the brief verified against the source data — target >92%
- Hallucination rate: % of claims with no corresponding source — target <2%

**In-visit Q&A eval (monthly, golden dataset of 50 Q&A pairs):**
- Answer correctness: % of answers that correctly answer the question based on available data — target >88%
- Source accuracy: % of cited sources that correctly correspond to the answer — target >95%
- Appropriate "no data" response: % of queries with no relevant data that correctly return "I don't have data on this" — target >90%

**Action item extraction eval (monthly, 50 transcript samples with human-labeled ground truth):**
- Extraction recall: % of human-labeled action items that were extracted — target >85%
- Extraction precision: % of extracted action items that correspond to real commitments — target >90%
- Structured output validity: % of extraction outputs that are valid JSON matching the schema — target 100%

### Online Metrics

| Metric | Description | Target |
|---|---|---|
| Brief generation latency | p95 time from trigger to push notification | <30 seconds |
| Brief open rate | % of FCs who open the brief before the visit | >80% |
| In-visit Q&A latency | p95 time from voice trigger to response | <2 seconds |
| Q&A session rate | % of visits with at least one in-visit Q&A | >40% (3 months post-launch) |
| Action item extraction rate | % of post-visit sessions that produce ≥1 action item | >85% |
| FC override rate | % of AI-extracted action items that FC edits substantially | Baseline in month 1, trend down over 6 months |
| Operator acknowledgment rate | % of delivered action items acknowledged within 48 hours | >75% |

---

## Human-in-the-Loop Design

| Action | Autonomy Level | Rationale |
|---|---|---|
| Brief generation | Automatic (no approval) | Brief is for FC consumption only; FC is responsible for how they use it |
| In-visit Q&A response | Automatic (no approval) | Real-time response; FC interprets and applies judgment |
| Action item draft | Automatic generation, FC approval required | Operator-facing; FC owns the relationship |
| Action item delivery to operator | Requires FC approval | No customer-facing content without FC sign-off |
| Automated reminders to operator | Automatic (templated, not AI-generated) | Templated reminder; not custom content |
| Action item completion marking | Operator or FC initiated | Both parties can confirm completion |

---

## Guardrails

- **No hallucinated claims in briefs:** Citation enforcement is a system requirement, not a suggestion. Any brief generation that fails the citation validation check is not delivered; it is flagged for human review.
- **Location data scoping:** In-visit Q&A retrieval is hard-scoped to the current location. Cross-location retrieval is not permitted.
- **No autonomous operator communication:** The system cannot send any message to a franchisee operator without FC approval. This is enforced at the delivery layer, not just the UI layer.
- **Data freshness transparency:** Stale data (>48 hours) is flagged visibly in the brief. FCs are trained to disclose data age to operators when presenting stale signals.
- **Voice data retention:** Voice notes are retained for 30 days post-processing for quality review. Raw audio is deleted after transcription is confirmed; the transcript is retained per the data retention policy.

---

## Privacy and Security

- All signal data (LMS, mystery shop, support, sales) is ingested and stored within the franchise platform's existing data boundary — no external API calls to third-party services for signal data.
- Voice transcription: audio is processed on-device where technically feasible; if cloud-processed, audio is encrypted in transit and at rest, and is deleted after transcription confirmation.
- Action item content is visible only to the FC who created it and the operator who received it. Operations director sees aggregate completion rates, not individual action item content.
- In-visit Q&A session logs are accessible to the FC and operations director for quality review purposes; not visible to operators.
- GDPR/CCPA: Franchisee operator personal data included in briefs or action items is subject to the platform's data processing agreement. No operator PII is retained in a context accessible to the AI model training pipeline without explicit consent.

---

## Rollout Plan

**Phase 1 — Private Beta (Weeks 1–6):**
- 10 FCs at 2 franchise networks
- Pre-visit brief only (no in-visit Q&A, no post-visit action structuring)
- Manual citation review by product team
- Weekly FC feedback sessions

**Phase 2 — Limited GA (Weeks 7–12):**
- 50 FCs across 5 networks
- Pre-visit brief + post-visit action structuring
- Automated citation validation
- Brief open rate and action item completion rate tracking live

**Phase 3 — Full GA (Weeks 13–20):**
- All FCs on supported platforms
- In-visit Q&A added
- Operations director dashboard live
- Begin measuring downstream visit quality correlations

**Success gate for Phase 2:** Brief open rate >70%, citation accuracy >90% in offline eval, no hallucinated claims escalated from beta FCs.

---

## Success Metrics

**6-month targets (post full GA):**
- Brief open rate: >80% of scheduled visits
- Action item completion rate: 25% improvement over baseline
- FC time on pre-visit research: 80% reduction (self-reported)
- Operator NPS for field visits: +5 points vs. baseline (synthetic target for illustration)
- Zero incidents of AI-generated claims leading to incorrect data presented to operators (measured via FC escalation log)

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| FC scepticism of AI-generated content | High | Medium | Citation visibility, beta with high-trust FCs first |
| Data freshness causing stale briefs | Medium | High | Freshness indicators, per-source update timestamps |
| Voice transcription errors in noisy environments | Medium | Medium | FC review of all extracted action items before send |
| Operator discomfort with data synthesis | Low | High | Operator disclosure in visit notice, clear framing |
| LLM hallucination in citation content | Low | High | Citation enforcement, monthly human eval |

---

## Open Questions

1. Should the brief include a recommended visit agenda, or just the signal synthesis? (Risk: a suggested agenda might reduce FC judgment; Benefit: saves 15 minutes of visit planning)
2. What is the minimum data coverage needed to generate a valid brief? (Currently: all five signal sources required. Should we generate a partial brief with available sources and flag missing ones?)
3. Should in-visit Q&A sessions be optionally shareable with the operator in real time, as a "live data" demonstration? (Operator transparency vs. FC workflow disruption)
4. How do we handle FCs who are visiting a location for the first time with no prior visit history? (Brief will be thinner; how do we communicate this?)
5. Should the post-visit action structuring include a "suggested follow-up call date" or is that too prescriptive?

---

*See also: [AI Field Consultant Case Study](/case-studies/ai-field-consultant-franchise-ops.md) · [AI Field Consultant Agent Blueprint](/agent-workflow-blueprints/ai-field-consultant-agent.md)*
