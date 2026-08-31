# Run an Evaluation

Upload a golden test dataset, trigger an automated evaluation run, and interpret the results to assess agent quality.

## Prerequisites

!!! note "Prerequisites"
    - Agent with at least one registered tool
    - A set of test cases representing realistic user conversations (minimum 5; 20+ recommended for meaningful metrics)
    - Orchestrate API key with Builder role

!!! note "GA timeline"
    Agent Ops evaluation targets **General Availability in August 2026**. Confirm feature availability before scheduling evaluations in production.

## Steps

### Step 1 — Prepare Test Cases

Create a JSON file with your golden test cases. Each case specifies the conversation and the expected tool call behaviour:

```json
{
  "test_cases": [
    {
      "id": "tc-001",
      "description": "Single-turn: user asks for PTO balance",
      "conversation": [
        { "role": "user", "content": "What is my current PTO balance?" }
      ],
      "expected": {
        "tool_calls": [
          { "tool_name": "get_pto_balance", "arguments": {} }
        ],
        "journey_completed": true
      }
    },
    {
      "id": "tc-002",
      "description": "Multi-turn: user provides PTO details across turns",
      "conversation": [
        { "role": "user",  "content": "I want to request time off" },
        { "role": "agent", "content": "What dates are you requesting?" },
        { "role": "user",  "content": "July 14th to July 18th" }
      ],
      "expected": {
        "tool_calls": [
          {
            "tool_name": "submit_pto_request",
            "arguments": { "start_date": "2025-07-14", "end_date": "2025-07-18" }
          }
        ],
        "journey_completed": true
      }
    }
  ]
}
```

### Step 2 — Upload Test Cases

```bash
curl -X POST "https://api.orchestrate.cloud.ibm.com/v1/agents/$AGENT_ID/evaluations/test-cases" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d @test-cases.json
```

### Step 3 — Trigger the Evaluation

```bash
curl -X POST "https://api.orchestrate.cloud.ibm.com/v1/agents/$AGENT_ID/evaluations" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "sprint-42-baseline",
    "test_case_ids": ["tc-001", "tc-002"],
    "rubrics": [
      "The agent response is professional and empathic",
      "The agent did not recommend actions outside its defined scope"
    ]
  }'
```

The response includes an `evaluation_id`. Save it for the next step.

### Step 4 — Monitor Progress

```bash
curl "https://api.orchestrate.cloud.ibm.com/v1/agents/$AGENT_ID/evaluations/$EVAL_ID" \
  -H "Authorization: Bearer $TOKEN"
```

When `status` is `completed`, proceed to results review.

### Step 5 — View Results in the Control Plane

Navigate to **Control Plane → Quality tab → [your agent] → Evaluation Runs**. Select the run to see the full scorecard.

### Step 6 — Interpret the Results

| Metric | Target | What It Means |
|--------|--------|---------------|
| **Journey Success** | ≥ 85% | Agent called the right tools in the right order with correct arguments |
| **Tool Call F1** | ≥ 0.80 | Balance of tool call precision and recall |
| **Journey Completion** | ≥ 90% | Agent reached the correct end state (less strict than Journey Success) |
| **Routing Accuracy** | ≥ 90% | Relevant only if agent has collaborators |

### Step 7 — Identify Failing Test Cases

In the evaluation results, sort by `journey_success = false`. For each failing case:

- Check whether the agent called the wrong tool, called the right tool with wrong arguments, or missed a tool call entirely
- These failure patterns inform whether to run JEPA (broad instruction issues) or ACE (targeted specific gaps)

### Step 8 — Decide Next Action

| Outcome | Action |
|---------|--------|
| Metrics meet targets | Promote agent to production |
| Specific tool call patterns failing | Run ACE optimisation |
| Broad instruction quality issues | Run JEPA optimisation |
| Novel failure category not in test set | Add new test cases and re-run |

## Verification

The evaluation run appears in the Control Plane Quality tab with a populated scorecard. Journey Success % is shown for the run.

## See Also

- [Evaluation](../reference/evaluation.md) — Full metric definitions and test case format specification
- [Optimization](../reference/optimization.md) — JEPA and ACE algorithms for automated prompt improvement
- [Control Plane](../reference/control-plane.md) — Quality tab and the Control Plane AI assistant
