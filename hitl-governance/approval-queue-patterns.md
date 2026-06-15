# Approval Queue Patterns

**Design patterns for HITL approval queues in enterprise B2B SaaS: routing, batching, timeout handling, and delegation**

---

## Why Approval Queue Design Is a Product Problem

The approval queue is where HITL either creates trust or creates overhead. A poorly designed approval queue produces:
- Approval fatigue (reviewers rubber-stamp without reading)
- SLA misses (high-priority items buried under low-priority items)
- Incorrect routing (the wrong person receives the approval request)
- Duplicate notifications (reviewer receives the same item via email, Slack, and in-app)

A well-designed approval queue is one of the most impactful product design decisions in an AI product because it determines whether the human oversight layer is meaningful or ceremonial.

---

## Queue Architecture

### Priority Tiers

The approval queue is not a FIFO queue. Items should be prioritised by a combination of:
- **Urgency:** Time-sensitive items first (renewal conversations, critical risk alerts)
- **Severity:** Higher-severity findings above lower-severity findings
- **Customer tier:** Enterprise customer actions above SMB (configurable)
- **SLA proximity:** Items approaching their SLA deadline surfaced prominently

**Priority score formula (illustrative):**
```
priority_score = (severity_weight × severity) + (urgency_weight × urgency) + (tier_weight × customer_tier) + (sla_weight × sla_proximity)
```

Where:
- severity: CRITICAL=5, HIGH=4, MEDIUM=3, LOW=1
- urgency: NOW=5, TODAY=4, THIS_WEEK=2, NEXT_WEEK=1
- customer_tier: Enterprise=3, Mid-market=2, SMB=1
- sla_proximity: Exponential decay as deadline approaches (1.0 at >7 days, 5.0 at <2 hours)

---

## Routing Design

**Account-based routing:** All actions related to an account should default to routing to the owner of that account. If no owner is assigned, route to the account's team lead.

**Role-based routing override:** Some actions should route based on role, not account ownership:
- Financial anomalies → Finance Ops owner
- Security/compliance findings → Security team
- Senior-level customer executive communications → CS manager, not the individual CSM

**Escalation routing:** If the primary reviewer has not acted within X% of the SLA window, automatically add the reviewer's manager as a co-reviewer. Do not remove the primary reviewer — they should still act if available.

**Routing validation:** The queue system should validate that the routed approver:
- Has the appropriate role permission for this action type
- Is currently active (not on PTO with out-of-office configured)
- Has not been deleted or offboarded from the system

---

## Batch Approval

Not all approval items require individual, sequential review. Batch approval is appropriate for:
- Low-urgency items from the same approver
- Items of the same type where the approver has established high trust (low edit rate)
- Routine notifications (e.g., a series of automated follow-up reminders that the CSM has pre-approved as a template)

**Batch approval interface design:**
- Show all batchable items in a list view with a "Review All" and "Approve All" option
- "Approve All" requires a confirmation ("You are approving X items — this will [describe action]. Confirm?")
- One-click expand to review any individual item before approving
- Items flagged as high-severity or high-customer-tier are excluded from batch approval and must be reviewed individually

**Anti-pattern:** Do not default to batch approval for all items. Force individual review for:
- CRITICAL or HIGH severity findings
- Enterprise-tier customer communications
- Any item with a rejection reason in the last 30 days from the same approver (suggests the approver is not reviewing carefully)

---

## Timeout Handling

What happens when an approver does not act before the SLA deadline?

**Option 1: Escalate (recommended for high-severity items)**
- Route to the approver's manager as the new primary
- Notify the original approver that escalation has occurred
- Do not automatically take the action — escalate to a human

**Option 2: Auto-expire with notification (appropriate for low-severity, time-sensitive items)**
- If the item is time-sensitive enough that inaction is itself a bad outcome, notify the approver that the opportunity has passed
- Log the expiry as a SLA miss
- Example: An expansion signal brief that expires because the window passed

**Option 3: Defer to next cycle (appropriate for recurring, low-urgency items)**
- If the item will be regenerated in the next cycle (e.g., a weekly digest), close the current item and regenerate in the next run

**Timeout configuration per action type:**
```
action_type: expansion_brief
sla_hours: 48
timeout_action: escalate_to_manager
escalation_notification: "Expansion opportunity for [ACCOUNT] has not been reviewed for 48 hours."
```

---

## Delegation Patterns

Enterprise users take PTO, go on parental leave, and change roles. The approval queue must handle delegation:

**Temporary delegation:**
- Reviewer configures a delegate for a date range
- All items that would route to the reviewer are routed to the delegate during the delegation period
- Delegate sees items labeled "Delegated from [NAME]"
- Upon return, the reviewer can see what was approved or rejected during their absence

**Permanent reassignment:**
- Account ownership transfer automatically updates routing for all future items
- Items currently in queue for the previous owner are reassigned to the new owner
- Previous owner and new owner both receive notification of the reassignment

**Team-level delegation:**
- For teams where any team member can approve on behalf of the team, route to the team queue
- Team members can claim individual items from the team queue ("I'll handle this one")
- Items not claimed within X hours are automatically assigned to the team lead

---

## Queue Health Metrics

The approval queue is a product surface that should be monitored like any other:

| Metric | Description | Target |
|---|---|---|
| SLA compliance rate | % of items acted on before the deadline | >90% for HIGH items, >99% for CRITICAL |
| Time-in-queue | Median and P95 time from item creation to first reviewer action | Monitor by severity; alert on trends |
| Approval-without-reading rate | % of approvals that occur in under the minimum review time threshold | <5% (indicates rubber-stamping) |
| Batch approval rate | % of items approved via batch vs. individual review | Track by user; high rate for high-severity items is a risk signal |
| Queue depth | Number of items in queue at a point in time | Alert if any reviewer's queue exceeds a defined depth |
| Delegation usage rate | % of items processed by a delegate vs. the primary reviewer | Tracks delegation patterns; unusually high may indicate overloaded reviewers |

---

## Notification Design

An approval item in the queue should trigger notifications that are:
- **Timely:** Sent when the item arrives, not batched to a daily digest (for HIGH/CRITICAL items)
- **Actionable:** Link directly to the approval interface, not to a home page
- **Non-duplicative:** One notification channel per reviewer preference (in-app, email, OR Slack — not all three by default)
- **Summarised for low-priority items:** Low-urgency items batched into a daily digest rather than individual pings

**Notification channels by default (configurable):**
- CRITICAL: In-app + email + Slack immediate
- HIGH: In-app + email immediate
- MEDIUM: In-app immediate + email daily digest
- LOW: Email daily digest only

---

*See also: [Human-in-the-Loop Design](human-in-the-loop-design.md) · [Escalation Logic](escalation-logic.md) · [Audit Logs and Decision Trails](audit-logs-and-decision-trails.md)*
