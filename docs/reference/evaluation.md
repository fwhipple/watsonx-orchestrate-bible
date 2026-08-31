# Evaluation

The watsonx Orchestrate Agent Ops evaluation framework provides systematic, quantitative assessment of agent quality. It replaces ad-hoc manual testing with reproducible, automated benchmarking — measuring how accurately agents invoke tools, complete user journeys, and route to the correct collaborator.

!!! note "GA timeline"
    Agent Ops evaluation is targeting **General Availability in August 2026**. Features described here reflect the capabilities at that GA milestone.

---

## Core Metrics

### Journey Success

**Definition:** The percentage of test cases in which the agent invoked the correct tools in the correct order with the expected arguments.

This is the primary accuracy metric. A journey is considered successful only if:
1. The right tools were called (no extra, no missing)
2. In the right sequence
3. With the correct argument values (within configured tolerance)

A journey that calls the right tools but in the wrong order, or with incorrect argument values, is scored as a failure.

### Tool Call F1 Score

**Definition:** The harmonic mean of tool call precision and recall across a test set.

- **Precision:** Of all tool calls the agent made, what fraction were correct?
- **Recall:** Of all tool calls the agent *should have* made, what fraction did it actually make?
- **F1:** Balances precision and recall into a single score (0–1 scale)

Tool Call F1 is particularly useful for detecting two failure modes:
- **Over-calling:** Agent invokes tools it shouldn't (high false positive rate → low precision)
- **Under-calling:** Agent misses required tool invocations (low recall)

### Journey Completion

**Definition:** The percentage of test cases in which the agent successfully reached the intended end state — regardless of the exact tool sequence used.

Journey Completion is a more lenient metric than Journey Success. It measures *whether the goal was achieved* rather than *whether the exact path was followed*. Useful for agents that have some flexibility in how they accomplish a task.

### Orchestrate Routing Accuracy

**Definition:** For multi-agent architectures, the percentage of test cases in which the primary orchestrator agent correctly delegated to the intended collaborator sub-agent.

This metric is only relevant for agents configured with collaborators. It validates that routing instructions are working correctly and that the supervisor agent selects the right specialist for each intent.

---

## Metric Summary Table

| Metric | What It Measures | Range | Use When |
|--------|-----------------|-------|----------|
| **Journey Success** | Correct tools, correct order, correct args | 0–100% | Primary quality gate; strict path compliance required |
| **Tool Call F1** | Precision + recall of tool invocations | 0–1.0 | Diagnosing over-calling or under-calling patterns |
| **Journey Completion** | Goal achievement regardless of path | 0–100% | Flexible agents where multiple paths to success are valid |
| **Routing Accuracy** | Correct collaborator selection | 0–100% | Multi-agent architectures with collaborator delegation |

---

## LLM-as-Judge Evaluators

Beyond the four core metrics, the framework supports custom **rubric evaluations** — criteria evaluated by an LLM judge rather than by exact-match comparison.

Examples of rubric criteria:

- "Was the agent's response empathic and professional?"
- "Did the agent avoid recommending actions outside its defined scope?"
- "Did the agent correctly handle the case where required information was missing?"

Rubric evaluations are configured as natural language criteria. The LLM judge scores each test case against each criterion on a 1–5 scale, producing a per-criterion quality distribution across the test set.

---

## Test Case Upload API

Test cases are uploaded via the REST API and stored as the agent's golden dataset.

### Test Case Format

```json
{
  "test_cases": [
    {
      "id": "tc-001",
      "description": "User requests flight status by flight number",
      "conversation": [
        {
          "role": "user",
          "content": "What's the status of flight BA295?"
        }
      ],
      "expected": {
        "tool_calls": [
          {
            "tool_name": "get_flight_status",
            "arguments": {
              "flight_number": "BA295"
            }
          }
        ],
        "journey_completed": true
      }
    }
  ]
}
```

### Upload Endpoint

```bash
POST /v1/agents/{agent_id}/evaluations/test-cases
Content-Type: application/json
Authorization: Bearer <api-key>

{
  "test_cases": [ ... ]
}
```

---

## Multi-Turn Conversation Support

Test cases can include multi-turn conversations — sequences of user and agent turns that simulate a realistic dialogue before the evaluated action occurs.

```json
{
  "conversation": [
    { "role": "user",  "content": "I need to book a hotel in Paris" },
    { "role": "agent", "content": "What dates are you looking for?" },
    { "role": "user",  "content": "Check-in July 14th, check-out July 18th" },
    { "role": "user",  "content": "And I need a room that allows pets" }
  ],
  "expected": {
    "tool_calls": [
      {
        "tool_name": "search_hotels",
        "arguments": {
          "city": "Paris",
          "check_in": "2025-07-14",
          "check_out": "2025-07-18",
          "pet_friendly": true
        }
      }
    ]
  }
}
```

Multi-turn test cases validate that the agent correctly accumulates context across turns and resolves arguments that were provided piecemeal across multiple messages.

---

## Running an Evaluation

```bash
POST /v1/agents/{agent_id}/evaluations
Content-Type: application/json
Authorization: Bearer <api-key>

{
  "name": "sprint-42-eval",
  "test_case_ids": ["tc-001", "tc-002", "tc-003"],
  "rubrics": [
    "The agent response is professional and empathic",
    "The agent did not recommend actions outside its defined scope"
  ]
}
```

Results are returned asynchronously. Poll the evaluation status endpoint or use the Control Plane UI to view results when complete.

---

## Related References

- [Optimization](optimization.md) — Using evaluation results to trigger JEPA/ACE prompt optimisation
- [Control Plane](control-plane.md) — Viewing quality metrics in the Quality tab
- [ADLC](adlc.md) — Where evaluation fits in the agent development lifecycle
- [HOWTO: Run an Evaluation](../howto/run-evaluation.md) — Step-by-step evaluation guide
- [Lab 7: Evaluation & Optimization](../labs/lab-07-evaluation.md) — Hands-on lab running evaluations and interpreting results
