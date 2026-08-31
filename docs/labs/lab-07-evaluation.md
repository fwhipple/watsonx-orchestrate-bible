# Lab 7: Evaluation & Optimization

**Objective:** Upload a golden test dataset for the Acme Employee Assistant, run an automated evaluation to measure Journey Success and Tool Call F1, and run the ACE optimization algorithm to improve a failing test case.

**Estimated Time:** 60 minutes

**Prerequisites:**

!!! note "Before you start"
    - Completed [Lab 6](lab-06-multi-agent.md)
    - Python 3.11+ with `requests` library: `pip install requests`
    - Your IBM Cloud API key and Agent ID from Lab 1

!!! note "GA timeline"
    Agent Ops evaluation targets General Availability in August 2026. If this feature is not yet available in your tenant, complete the conceptual walkthrough in Steps 1–3 and return to run the live evaluation when available.

---

## Part 1 — Prepare the Golden Test Dataset

### Step 1.1 — Create the Test Cases

Save this as `test-cases.json`:

```json
{
  "test_cases": [
    {
      "id": "tc-pto-001",
      "description": "Single-turn: user requests PTO days calculation",
      "conversation": [
        { "role": "user", "content": "How many working days is it from July 14th to July 18th 2025?" }
      ],
      "expected": {
        "tool_calls": [{ "tool_name": "pto_calculator", "arguments": { "start_date": "2025-07-14", "end_date": "2025-07-18" } }],
        "journey_completed": true
      }
    },
    {
      "id": "tc-routing-hr-001",
      "description": "Routing: PTO request should route to HR Specialist",
      "conversation": [
        { "role": "user", "content": "I want to take two weeks off in August" }
      ],
      "expected": {
        "tool_calls": [{ "tool_name": "PTO Approval Workflow", "arguments": {} }],
        "journey_completed": true
      }
    },
    {
      "id": "tc-routing-it-001",
      "description": "Routing: IT ticket check should route to IT Helpdesk",
      "conversation": [
        { "role": "user", "content": "Can you check the status of my IT ticket IT-12345?" }
      ],
      "expected": {
        "tool_calls": [{ "tool_name": "get_it_ticket_status", "arguments": { "ticket_id": "IT-12345" } }],
        "journey_completed": true
      }
    },
    {
      "id": "tc-multi-turn-001",
      "description": "Multi-turn: user provides PTO dates across multiple messages",
      "conversation": [
        { "role": "user",  "content": "I need to request some time off" },
        { "role": "agent", "content": "Of course! What dates are you thinking?" },
        { "role": "user",  "content": "Starting August 11th and coming back August 15th, for employee EMP-001" }
      ],
      "expected": {
        "tool_calls": [
          {
            "tool_name": "PTO Approval Workflow",
            "arguments": { "employee_id": "EMP-001", "start_date": "2025-08-11", "end_date": "2025-08-15" }
          }
        ],
        "journey_completed": true
      }
    },
    {
      "id": "tc-weather-001",
      "description": "Direct: weather query should be handled by orchestrator directly",
      "conversation": [
        { "role": "user", "content": "What's the weather forecast for London this week?" }
      ],
      "expected": {
        "tool_calls": [{ "tool_name": "get_weather_forecast", "arguments": {} }],
        "journey_completed": true
      }
    }
  ]
}
```

### Step 1.2 — Upload the Test Cases

```bash
export AGENT_ID="<your-agent-id>"
export TOKEN="<your-iam-token>"  # Get from: ibmcloud iam oauth-tokens

curl -X POST \
  "https://api.orchestrate.cloud.ibm.com/v1/agents/$AGENT_ID/evaluations/test-cases" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d @test-cases.json
```

---

## Part 2 — Run the Evaluation

### Step 2.1 — Trigger the Evaluation

```bash
curl -X POST \
  "https://api.orchestrate.cloud.ibm.com/v1/agents/$AGENT_ID/evaluations" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "lab-7-baseline",
    "test_case_ids": ["tc-pto-001", "tc-routing-hr-001", "tc-routing-it-001", "tc-multi-turn-001", "tc-weather-001"],
    "rubrics": [
      "The agent response is professional and helpful",
      "The agent correctly identifies when to delegate vs handle directly"
    ]
  }'
```

Note the `evaluation_id` in the response.

### Step 2.2 — Monitor Progress

```bash
curl \
  "https://api.orchestrate.cloud.ibm.com/v1/agents/$AGENT_ID/evaluations/$EVAL_ID" \
  -H "Authorization: Bearer $TOKEN"
```

Wait for `"status": "completed"`.

---

## Part 3 — Interpret the Results

### Step 3.1 — View in Control Plane

Navigate to **Control Plane → Quality tab → Acme Employee Assistant → Evaluation Runs → lab-7-baseline**.

Review:

| Metric | Your Score | Target |
|--------|-----------|--------|
| Journey Success | ___ % | ≥ 85% |
| Tool Call F1 | ___ | ≥ 0.80 |
| Journey Completion | ___ % | ≥ 90% |
| Routing Accuracy | ___ % | ≥ 90% |

### Step 3.2 — Identify Failing Cases

Sort the test case results by `journey_success = false`. Likely failure:
- `tc-routing-hr-001` — the orchestrator may have tried to handle the PTO request directly instead of routing to the HR Specialist

---

## Part 4 — Run ACE Optimization

### Step 4.1 — Identify the Failure Pattern

The routing failure pattern: the orchestrator is not consistently delegating to the HR Specialist for PTO requests.

ACE is the right algorithm here — it adds a targeted guideline rather than rewriting the entire system prompt.

### Step 4.2 — Run ACE

```bash
curl -X POST \
  "https://api.orchestrate.cloud.ibm.com/v1/agents/$AGENT_ID/optimizations" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "algorithm": "ace",
    "evaluation_id": "'"$EVAL_ID"'",
    "target_metrics": { "journey_success": 0.90 }
  }'
```

### Step 4.3 — Review the Proposed Guideline

When complete, review the proposed addition. ACE should have added something like:

```
When a user mentions PTO, vacation, time off, or leave requests, always delegate to the HR Specialist. Do not attempt to handle PTO requests directly.
```

### Step 4.4 — Accept and Re-Evaluate

Accept the proposed guideline. The platform adds it to the agent's instructions. Re-run the evaluation:

```bash
curl -X POST \
  "https://api.orchestrate.cloud.ibm.com/v1/agents/$AGENT_ID/evaluations" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "lab-7-post-ace",
    "test_case_ids": ["tc-routing-hr-001", "tc-multi-turn-001"],
    "rubrics": []
  }'
```

Journey Success for the two routing test cases should improve.

---

## Validation Checkpoints

- [ ] Test cases uploaded successfully (5 test cases)
- [ ] Evaluation completed with a scorecard (all 4 core metrics populated)
- [ ] At least one failing test case identified
- [ ] ACE optimization run triggered and proposed guideline reviewed
- [ ] Post-ACE evaluation shows improvement in the routing test cases

---

## What You Learned

- How to design a golden test dataset with single-turn, multi-turn, and routing test cases
- How to interpret Journey Success, Tool Call F1, Journey Completion, and Routing Accuracy
- The difference between JEPA (holistic rewrite) and ACE (additive guideline) optimization
- How the optimization → re-evaluate loop works in practice

---

**Next Lab:** [Lab 8 — Observability](lab-08-observability.md)
