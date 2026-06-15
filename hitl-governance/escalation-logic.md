# Escalation Logic

**Designing escalation: trigger conditions, escalation paths, urgency scoring, notification design, and SLA considerations**

---

## What Escalation Is and Is Not

Escalation is the mechanism by which an AI system or an approval queue transfers responsibility from one human to another when the first human has not acted, cannot act, or when the stakes require a higher level of authority.

Escalation is not:
- A failure state (it is a designed and expected control mechanism)
- Punishment for inaction (it is workflow management)
- Evidence that the AI system is unreliable (high escalation rate may indicate appropriate caution)

Escalation is:
- A signal that a decision requires more attention, higher authority, or broader context than the primary reviewer provided
- A mechanism for ensuring that no high-consequence decision is missed due to absence, overload, or inaction
- An audit trail event — every escalation is logged

---

## Escalation Trigger Conditions

### Time-Based Escalation

**Trigger:** The primary reviewer has not acted on an item within a defined percentage of the item's SLA.

**Example:** A CRITICAL risk finding has a 2-hour SLA. If the primary reviewer has not acted within 90 minutes (75% of SLA), the item is escalated to the manager.

**Configuration:**
```
escalation_trigger: sla_percentage_elapsed
threshold: 0.75
escalate_to: reviewer.manager
notify_primary: true
message: "This item has not been reviewed. It has been escalated to {manager_name}."
```

**Graduated time escalation:** For CRITICAL items, escalate early (75% SLA) and frequently (every 30 minutes thereafter until acted on or SLA expires).

---

### Confidence-Based Escalation

**Trigger:** The AI system's confidence in its finding or recommendation is below the threshold for the assigned autonomy level.

**Example:** An account is flagged as a churn risk with confidence 0.62. The threshold for Level 3 autonomy (execute with notification) for churn risk actions is 0.80. The item is escalated to the approval queue at the CSM manager level rather than executing autonomously.

**Configuration:**
```
escalation_trigger: confidence_below_threshold
action_type: churn_risk_intervention
autonomy_level_threshold: 0.80
current_confidence: 0.62
escalate_to: cs_manager_queue
escalation_label: "Low confidence — requires senior review"
```

---

### Severity-Based Escalation

**Trigger:** The finding severity exceeds the authority level of the primary reviewer.

**Example:** A financial anomaly finding is rated CRITICAL. The CSM who owns the account receives the notification, but the item is simultaneously escalated to the Finance Operations queue because CRITICAL financial findings require Finance Ops review.

**Authority mapping (configurable):**

| Severity | Primary Reviewer | Mandatory Co-Reviewer |
|---|---|---|
| LOW | Account CSM | None |
| MEDIUM | Account CSM | None |
| HIGH | Account CSM | CS Manager (notification only) |
| CRITICAL | CS Manager | VP of CS (approval required) |

---

### Exception-Based Escalation

**Trigger:** A specific content exception is detected in the AI output or the finding.

**Examples:**
- Customer mentioned a competitor in a negative context → escalate to competitive intelligence team
- Finding involves a customer who is also a reseller or partner → escalate to partner team
- Finding involves a customer in a regulated industry (healthcare, finance) → escalate to compliance reviewer

**Exception library:** Maintain a configurable list of exception triggers. Each trigger specifies: detection rule (keyword, entity type, account attribute), co-reviewer to add, notification message.

---

### Inactivity-Based Escalation (Absent Reviewer)

**Trigger:** The primary reviewer is marked as inactive (out of office, PTO, offboarded) and has not configured a delegate.

**Handling:**
1. Check if the reviewer has configured a delegate → route to delegate
2. If no delegate configured → route to the reviewer's manager
3. If the reviewer is offboarded → route to the account's new owner or the team queue
4. Notify the team lead that an unhandled escalation occurred due to missing delegation

---

## Escalation Paths

### Linear Escalation

```
Primary Reviewer → Manager → Director → VP
```

Used for time-based and inactivity-based escalations. Each level up is one step.

**Risk:** Can become bureaucratic for fast-moving situations. Set appropriate time limits at each escalation level.

### Parallel Escalation

```
Primary Reviewer + Co-Reviewer (simultaneously)
```

Used for high-severity findings that require two independent reviewers rather than sequential escalation.

**Example:** All customer communications tagged as high-value renewal negotiations require parallel review by the account CSM and the CS manager.

### Broadcast Escalation

```
Primary Reviewer + Entire Team Queue
```

Used for CRITICAL items where the fastest possible response is required and any team member should be able to act.

**Risk:** If multiple team members attempt to act simultaneously, the first to act "claims" the item and the others are notified that it is handled. Must be designed at the queue level to prevent double-action.

---

## Urgency Scoring for Escalation Prioritisation

Not all escalated items are equally urgent. The escalation notification and queue position should reflect urgency:

```
urgency_score = (base_severity_score) 
              × (1 + sla_pressure_multiplier) 
              × (customer_tier_multiplier)

sla_pressure_multiplier = (1 / hours_remaining_on_sla)^0.5
  (escalates exponentially as deadline approaches)

customer_tier_multiplier: Enterprise = 1.5, Mid-market = 1.2, SMB = 1.0
```

**Urgency score → notification channel:**

| Urgency Score | Notification |
|---|---|
| >8.0 | Immediate: all channels (Slack, email, SMS if configured) |
| 5.0–8.0 | Immediate: Slack + email |
| 2.0–5.0 | In-app + email within 15 minutes |
| <2.0 | Email daily digest |

---

## Notification Design for Escalated Items

An escalation notification must be:

**For the new reviewer (escalated-to):**
- Clear about what is being escalated and why: "[ITEM TYPE] for [ACCOUNT] has been escalated to you because [REASON]"
- Clear about what action is required: "Please review and approve or reject by [DEADLINE]"
- A direct link to the item in the approval interface — not to the home page
- Context about the original reviewer: "Originally assigned to [NAME], who has not acted"

**For the original reviewer (if still active):**
- Notification that escalation has occurred: "Your action on [ITEM] was overdue. It has been escalated to [MANAGER NAME]."
- Clear that they can still act if the escalated reviewer has not yet acted
- Not punitive in tone — factual and informative

**Template:**
```
Subject: [ESCALATED] [ITEM TYPE] — [ACCOUNT NAME] — Action required by [DEADLINE]

This [ITEM TYPE] for [ACCOUNT NAME] was escalated to you because [REASON]:
- Original assignee: [NAME] 
- Time elapsed: [N hours]
- SLA deadline: [TIME]

What the AI found:
[FINDING SUMMARY — 2 sentences max]

Recommended action:
[ACTION — 1 sentence]

[REVIEW NOW →] (direct link to approval interface)
```

---

## SLA Design

SLAs for escalation should be defined per action type and severity level. They must be:
- **Documented and communicated** to all reviewers and managers before the system goes live
- **Monitored continuously** with alert thresholds
- **Reported in team reviews** — SLA compliance is a team health metric

**SLA reference table (illustrative):**

| Action Type | Severity | Primary Reviewer SLA | Escalation Threshold | Post-Escalation SLA |
|---|---|---|---|---|
| Churn risk alert | CRITICAL | 2 hours | 75% (90 min) | 30 minutes |
| Churn risk alert | HIGH | 24 hours | 75% (18 hours) | 4 hours |
| Follow-up email draft | Any | 4 hours | 90% (3.6 hours) | 2 hours |
| Expansion brief | Any | 48 hours | 75% (36 hours) | 12 hours |
| QBR section approval | Any | 72 hours | 75% (54 hours) | 24 hours |

---

*See also: [Approval Queue Patterns](approval-queue-patterns.md) · [Human-in-the-Loop Design](human-in-the-loop-design.md) · [Audit Logs and Decision Trails](audit-logs-and-decision-trails.md)*
