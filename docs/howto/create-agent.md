# Create an Agent

Create a new agent in watsonx Orchestrate using the Agent Builder UI or the REST API.

## Prerequisites

!!! note "Prerequisites"
    - watsonx Orchestrate SaaS account (or CPD access) with Builder role
    - A clear understanding of the agent's purpose, tools it will need, and any external systems it will call

## Steps

### Step 1 — Navigate to Agent Builder

Log in to the Orchestrate console and select **Agent Builder** from the left navigation. Click **New Agent**.

### Step 2 — Set Name and Description

Enter a descriptive name (e.g., `HR Assistant`) and a short description. The description helps other builders understand the agent's purpose and is used by collaborator routing when this agent is used as a sub-agent.

### Step 3 — Write System Instructions

System instructions define the agent's persona, behaviour, and constraints. Follow the **Role–Goal–Context–Constraints** pattern:

```
You are an HR Assistant for Acme Corp. Your role is to help employees 
with HR queries, PTO requests, and policy questions.

Goal: Answer HR questions accurately and help employees complete 
HR-related tasks using the available tools.

Context: You have access to the Workday HR system via the 
get_pto_balance, submit_pto_request, and search_hr_policy tools.

Constraints:
- Never provide legal or medical advice
- If a request is outside your scope, direct the employee to hr@acme.com
- Always confirm actions before executing them (e.g., before submitting a PTO request)
```

!!! tip "Tool descriptions matter"
    Write clear, unambiguous descriptions for every tool parameter. The LLM reads these to extract argument values from user messages — vague descriptions cause incorrect tool calls. See [Tool Types](../reference/tool-types.md) for guidance.

### Step 4 — Select a Model

Choose the foundation model. See [Supported Models](../reference/models.md) for guidance on model selection.

| Need | Recommended Model |
|------|-----------------|
| Low latency, simple intents | `granite-3-8b-instruct` |
| Standard enterprise use case | `granite-3-20b-instruct` |
| Complex reasoning, multi-tool chains | `claude-3-5-sonnet` (Premium tenant required) |

### Step 5 — Save and Test

Click **Save**. The chat preview panel opens automatically. Send a test message to verify the agent responds correctly with no system errors.

### Step 6 (Optional) — Create via REST API

```bash
curl -X POST https://api.orchestrate.cloud.ibm.com/v1/agents \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "HR Assistant",
    "description": "Helps employees with HR queries and PTO requests",
    "model": "granite-3-20b-instruct",
    "instructions": "You are an HR Assistant..."
  }'
```

## Verification

- The agent appears in the Agent Builder agent list
- The chat preview returns a coherent response to a test message
- No error banners appear in the builder console

## See Also

- [ADLC](../reference/adlc.md) — Where agent creation fits in the development lifecycle
- [Agent Building Blocks](../reference/agent-building-blocks.md) — Technical reference for all six building blocks
- [Add a Tool](add-tool.md) — Next step: register tools for the agent to use
