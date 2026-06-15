# Agent Task Success Evals

**How to evaluate agent task completion: task decomposition quality, tool use accuracy, plan adherence, and loop completion**

---

## Why Agent Evals Are Different

Evaluating a single-turn RAG response is relatively straightforward: did the response answer the question, was it grounded, was the citation correct?

Evaluating an agent is harder because:
1. The agent takes multiple actions over multiple steps — quality is a function of the full trajectory, not just the final output
2. The same goal can be achieved through different valid action sequences — there is no single "right" trajectory
3. Failure can occur at any step and cascade through subsequent steps in non-obvious ways
4. The agent may produce a correct final output despite making errors mid-trajectory (lucky success) or a wrong final output despite correct reasoning (bad luck, bad tools)

Agent evals must measure both the trajectory (how did it get there?) and the outcome (did it get there?).

---

## The Three Levels of Agent Evaluation

### Level 1: Step-Level Evaluation

Evaluate the quality of individual agent actions (tool calls, decisions) without regard to the overall task.

**What to measure:**
- **Tool call validity:** Did each tool call use valid parameters and match the schema?
- **Tool selection appropriateness:** Was the chosen tool the right tool for the current step's goal?
- **Parameter accuracy:** Were the parameters passed to the tool correct given the agent's current context?

**Why it matters:** Step-level failures compound. An incorrect parameter in step 2 may not produce an error but will produce incorrect downstream results that cascade through all subsequent steps.

**Measurement:** Build a step validator that checks each tool call against: (a) schema validity, (b) expected tool for this step type, (c) parameter sanity checks (e.g., date ranges that are not negative, entity IDs that exist).

---

### Level 2: Trajectory-Level Evaluation

Evaluate the quality of the agent's action sequence relative to a reference trajectory or a defined task decomposition.

**What to measure:**
- **Task decomposition quality:** Did the agent correctly identify the subtasks needed to complete the goal?
- **Step efficiency:** Did the agent complete the task in a reasonable number of steps, or did it take unnecessary detours?
- **Plan adherence:** If the agent generated an explicit plan (as in a planner-executor pattern), did the executor follow the plan, and did the planner update the plan correctly when intermediate results changed?
- **Hallucinated tool calls:** Did the agent attempt to call tools that do not exist?

**Reference trajectory:** For a golden evaluation set, define a reference trajectory (the "correct" or "acceptable" sequence of steps) for each task. Calculate the edit distance between the agent's actual trajectory and the reference. High edit distance indicates the agent is solving the problem differently — which may or may not be a quality issue.

**Note on trajectory comparison:** Unlike code, agent trajectories may have multiple valid paths. The reference trajectory is a quality signal, not a ground truth. A trajectory that differs significantly from the reference should be reviewed, not automatically failed.

---

### Level 3: Outcome-Level Evaluation

Evaluate whether the agent achieved the intended goal, regardless of how it got there.

**What to measure:**
- **Task completion rate:** What fraction of tasks are completed successfully?
- **Output quality:** Does the final output meet the quality standard? (For RAG outputs in an agent: apply the RAG quality rubrics)
- **Goal achievement:** Did the agent accomplish what the user intended, even if the final output technically matches the goal specification?

**Task completion rate** is the primary agent success metric. Measure it on a task-type basis — different task types have different expected completion rates.

**Goal vs. specification:** An agent may produce an output that technically satisfies the task specification but does not accomplish the user's intent. Example: a task specified "draft a follow-up email for the Acme Corp meeting" — the agent drafts an email but addresses the wrong person. The specification is met; the goal is not. Measuring both requires human review of the final output, not just automated schema validation.

---

## Evaluation Dimensions by Agent Type

### Single-Agent Tasks

| Dimension | Measurement | Target |
|---|---|---|
| Task completion rate | % of tasks completed without error or escalation | >85% |
| Step efficiency | Actual steps / expected minimum steps (ratio) | <2.0 |
| Tool error rate | % of tool calls that return an error | <5% |
| Final output quality | LLM-as-judge on final output | >4.0/5.0 |
| HITL escalation rate | % of tasks escalated to human | Baseline in month 1; target trending down |

### Planner-Executor Tasks

| Dimension | Measurement | Target |
|---|---|---|
| Plan quality | Human reviewer rating of generated plan appropriateness (sampled) | >4.0/5.0 |
| Step success rate | % of individual plan steps completed successfully | >90% |
| Replan frequency | % of tasks requiring a mid-execution replan | Tracked; high rate signals poor initial plan quality |
| Plan-to-execution alignment | % of executed steps that match the generated plan | >85% |
| Overall completion rate | % of plans where all required steps complete | >80% |

### Multi-Agent Tasks

| Dimension | Measurement | Target |
|---|---|---|
| Coordinator decomposition quality | % of multi-agent tasks where the coordinator correctly identifies all required specialists | >85% |
| Specialist success rate | Per-specialist completion rate (identifies reliability issues by domain) | >90% per specialist |
| Conflict detection rate | % of multi-agent tasks where genuine conflicts are detected and surfaced | Tracks sensitivity |
| Synthesis quality | LLM-as-judge or human review of final synthesised output | >4.0/5.0 |
| Parallelism efficiency | Actual latency / theoretical sequential latency | <1.5 (25% efficiency gain minimum) |

---

## Building a Task Evaluation Suite

### Step 1: Define Task Types

List all task types the agent system handles. For each type:
- Define the success criterion (what does a successful completion look like?)
- Define the minimum and maximum acceptable steps
- Define the expected tool sequence (reference trajectory)
- Identify failure modes specific to this task type

### Step 2: Create Task Evaluation Scenarios

For each task type, create 5–10 evaluation scenarios:
- 3–5 standard cases (normal inputs, expected flow)
- 2–3 edge cases (unusual but valid inputs, error recovery scenarios)
- 1–2 adversarial cases (inputs designed to trigger known failure modes)

### Step 3: Define Automated Checks

For each scenario, define what can be automatically checked:
- Tool call sequence matches reference (with allowed deviations)
- Tool call parameters are valid
- Final output schema is valid
- Task terminal condition is met

### Step 4: Define Human Review Criteria

For each scenario, define what requires human review:
- Output quality (does it actually answer the question well?)
- Goal achievement (did the agent accomplish the user's intent?)
- Ambiguous trajectory deviations (is this deviation a problem or a valid alternative?)

---

## Common Agent Failure Patterns

| Failure Pattern | Description | Detection |
|---|---|---|
| Infinite loop | Agent calls the same tools repeatedly without progress | Step count limit exceeded; repeated tool call pattern detection |
| Tool hallucination | Agent attempts to call a tool that does not exist | Tool call validation against manifest |
| Context loss | Agent loses track of the original goal mid-trajectory | Goal reference tracking; final output relevance eval |
| Premature termination | Agent declares the task complete before it is done | Terminal condition validation |
| Over-cautious escalation | Agent escalates to human too frequently for simple tasks | HITL escalation rate monitoring |
| Silent failure | A tool fails silently and the agent continues as if it succeeded | Tool call return value monitoring; error state propagation |
| Plan abandonment | Agent generates a plan but deviates from it without replanning | Plan-to-execution alignment metric |

---

*See also: [Single-Agent Pattern](/agent-workflow-blueprints/single-agent-pattern.md) · [Multi-Agent Operating Loop](/agent-workflow-blueprints/multi-agent-operating-loop.md) · [Online AI Product Metrics](online-ai-product-metrics.md)*
