# Human-in-the-Loop Design Framework

**The 4-level autonomy spectrum, when to use each level, and how to design HITL that builds trust rather than friction**

---

## The Core Principle: HITL Is a Product Strategy, Not a Limitation

The default framing of human-in-the-loop in AI product discussions is negative: "we need HITL because the AI is not reliable enough yet." This framing leads to HITL design that minimises friction and tries to reduce human involvement over time.

The better framing is: human-in-the-loop is a design choice that determines who is accountable for an AI-assisted decision, and accountability design is a product strategy that creates trust in regulated and high-stakes enterprise contexts.

In regulated industries (financial services, healthcare, legal) and high-consequence enterprise contexts (customer relationship management, identity provisioning, franchise compliance), users need to know that a human was accountable for every significant decision. HITL provides that accountability. Designing it well means making it fast, informative, and meaningful — not minimal or absent.

---

## The 4-Level Autonomy Spectrum

### Level 1: Recommend

**What the AI does:** Analyses signals, generates a recommendation. Presents the recommendation to the human. The human reads it and decides independently whether to act.

**Human role:** Reviews the recommendation; decides whether to act and how.

**AI action taken if human ignores:** Nothing. No action is taken without explicit human action.

**When to use:**
- High-stakes decisions where the human needs to own the outcome completely
- Early in deployment when AI quality is not yet established
- When the recommendation context is complex enough that the human needs to apply judgment that cannot be captured in a simple approve/reject

**Example:** AI generates a pre-visit brief for a field consultant. The FC reads it. The FC decides how to run the visit based on the brief. No approval required.

**Design principle:** The recommendation layer should be useful enough that the human actually reads it. If the recommendation is consistently ignored, it is not adding value — investigate why.

---

### Level 2: Draft and Wait for Approval

**What the AI does:** Generates a draft action (email, report, configuration, etc.) and stages it for human review. No action is taken until the human explicitly approves.

**Human role:** Reviews the draft. May edit before approving. Explicitly approves or rejects.

**AI action taken if human ignores:** Nothing. The draft sits in the queue.

**When to use:**
- Any customer-facing or external-facing output
- Any write action that modifies a system of record
- Medium-to-high consequence actions where the human should be accountable for the content
- Actions that can be meaningfully reviewed in under 5 minutes

**Example:** AI drafts a follow-up email after a customer meeting. CSM reviews, edits if needed, approves. Email is sent only after approval.

**Design principle:** The approval step must be visible in the UI, not buried. The human should never have to hunt for the "approve" action. The draft should be presented in a way that makes review easy — not as raw text that requires formatting imagination.

---

### Level 3: Execute with Notification

**What the AI does:** Executes an action autonomously and notifies the responsible human immediately after.

**Human role:** Receives notification. Can review what the AI did. Can reverse the action within a defined time window (if reversible).

**AI action taken if human ignores:** The action stands (unless reversed within the window).

**When to use:**
- Actions that are low-consequence and reversible
- Actions where the cost of delay (waiting for approval) exceeds the risk of the action being slightly wrong
- Actions where the human has established trust in the AI's judgment through repeated validated performance
- Non-customer-facing internal actions (e.g., updating an internal CRM tag, creating a reminder)

**Example:** AI detects that an account's health score has dropped below the threshold and automatically creates a CRM task for the CSM with a suggested action. Notifies the CSM. CSM can delete the task if not relevant.

**Design principle:** The notification must be actionable — the human should be able to review and reverse from the notification itself, without navigating to a separate interface.

---

### Level 4: Execute Autonomously

**What the AI does:** Executes an action without requiring approval or providing immediate notification.

**Human role:** Can review in audit logs. Can reverse if the action is reversible.

**AI action taken if human ignores:** The action stands.

**When to use:**
- Only for fully reversible, low-consequence, high-confidence actions
- Actions where the cost of human involvement (time, attention) consistently exceeds the value of the oversight
- Actions where the AI's performance has been validated at high accuracy over a long period (6+ months)
- Background system maintenance (not user-facing)

**Example:** AI automatically refreshes the data freshness index for location signals overnight. No human involvement needed.

**Design principle:** Level 4 autonomy is a privilege, not a default. It is earned through demonstrated performance and limited to specific action types. Creeping autonomy (gradually expanding Level 4 to more action types without explicit re-evaluation) is a governance risk.

---

## Designing HITL That Creates Trust Rather Than Friction

### Principle 1: Make the Review Informative

If the human approval step does not give the reviewer enough context to make a meaningful judgment, they will approve blindly — which defeats the purpose. Every approval interface must include:
- What the AI is recommending (the draft or action)
- Why the AI is recommending it (the evidence and reasoning, with citations)
- What the expected outcome is if the action is taken
- What happens if the reviewer dismisses or edits

A CSM approving an expansion brief should see: what the expansion opportunity is, what signals detected it, what the confidence level is, and what the recommended action is — not just a "Draft expansion brief — Approve / Reject" button.

### Principle 2: Right Reviewer, Right Time

The approval request must go to the person who has the context and authority to make a good decision — not whoever is technically assigned as the approver by default.

- Account-specific actions → the CSM who owns the account
- High-severity risk findings → the CS manager, not just the CSM
- External communications → the person who owns the customer relationship
- Financial or billing anomalies → the appropriate finance or RevOps owner

Routing matters. An approval queue that goes to the wrong person will be rubber-stamped or ignored.

### Principle 3: SLA Transparency

The approval queue should display urgency and time remaining clearly. A high-severity risk finding that expires in 2 hours should look and feel different from a low-priority follow-up suggestion due in 3 days.

Define explicit SLAs for each action type:
- CRITICAL findings: action within 2 hours
- HIGH priority: action within 24 hours
- MEDIUM priority: action within 72 hours
- LOW priority: action within 7 days

Monitor SLA adherence. Escalate to the reviewer's manager when an SLA is missed on a CRITICAL or HIGH item.

### Principle 4: Meaningful Rejection Paths

"Reject" is not the end of the interaction. When a reviewer rejects an AI-generated output, they should be asked why. The rejection reason is the highest-value signal the system can collect for improving future outputs. Design the rejection flow to make reason capture easy (structured categories + optional free text) and fast (<30 seconds).

### Principle 5: Audit Trails Are the Product

Every approval, rejection, and edit should be logged with: the reviewer's identity, the timestamp, the original AI output, the modified output (if edited), and the reason (if provided). This audit trail is not just for compliance — it is the source of the preference data that drives model improvement.

---

## HITL Governance Configuration

The autonomy level for each action type should be configurable by an authorised administrator, not hardcoded. This allows:
- Starting at Level 1 or 2 for all actions, then selectively graduating to Level 3 for proven action types
- Reverting to a lower autonomy level if a quality regression is detected
- Applying different autonomy levels for different user roles (a senior CSM may have Level 3 autonomy for certain actions; a junior CSM may have Level 2)

**Configuration schema (per action type):**
```
action_type: send_follow_up_email
default_autonomy_level: 2
allowed_roles_for_level_3: [senior_csm, cs_manager]
revert_to_level_2_if: acceptance_rate < 0.70 for 14 days
audit_log_required: true
undo_window_seconds: 3600
```

---

*See also: [Approval Queue Patterns](approval-queue-patterns.md) · [Confidence Thresholds](confidence-thresholds.md) · [HITL as Product Strategy](/strategy-memos/hitl-as-product-strategy.md)*
