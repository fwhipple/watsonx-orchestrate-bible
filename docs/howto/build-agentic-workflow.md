# Build an Agentic Workflow

Build a deterministic, multi-step agentic workflow with a conditional branch and a human approval gate.

!!! note "Reference vs HOWTO"
    This page is the step-by-step build guide. For the full technical specification of node types, scheduling rules, and channel support, see [Agentic Workflows](../reference/agentic-workflows.md).

## Prerequisites

!!! note "Prerequisites"
    - At least two registered tools (one to fetch data, one to execute an action)
    - Builder role in watsonx Orchestrate
    - An agent to attach the completed workflow to

## Steps

### Step 1 — Create a New Workflow

Navigate to **Workflow Builder → New Workflow**. Enter a name and description.

### Step 2 — Add a Tool Call Node

Click **+ Add Node → Tool Call**. Select your data-fetch tool (e.g., `get_expense_request`).

Map the tool's input parameters from workflow inputs:

```
Input parameter: request_id
Value: {{inputs.request_id}}
```

### Step 3 — Add a Condition Node

Click **+ Add Node → Condition**. Define the branch logic based on the output from Step 2:

| Condition | Branch |
|-----------|--------|
| `{{nodes.get_expense_request.output.amount}} > 5000` | Route to Approval |
| `else` | Route to Auto-Approve |

### Step 4 — Add a User Activity (Approval) Node

On the high-value branch, click **+ Add Node → User Activity → Approval**.

Configure the approval message:

```
Expense request from {{nodes.get_expense_request.output.employee_name}}
Amount: ${{nodes.get_expense_request.output.amount}}
Category: {{nodes.get_expense_request.output.category}}

Please review and approve or reject.
```

Set the approver: select a role (e.g., `Manager`) or a specific user.

### Step 5 — Add Action Nodes After Approval

After the Approval node, add a second **Condition** node branching on the approval outcome:

- **Approved** → Add Tool Call node: `process_expense`
- **Rejected** → Add Tool Call node: `notify_rejection`

For the auto-approve branch (amounts ≤ $5,000), add the `process_expense` Tool Call node directly.

### Step 6 — Configure Workflow Inputs and Outputs

In the **Workflow Settings** panel:

```json
{
  "inputs": {
    "request_id": { "type": "string", "description": "The expense request ID to process" }
  },
  "outputs": {
    "status": "{{ nodes.process_expense.output.status }}"
  }
}
```

### Step 7 — Test the Workflow

Click **Test** in the Workflow Builder. Enter a test `request_id`. Follow the flow:

1. The tool call node fetches the request
2. The condition routes to the approval branch
3. An approval prompt appears in the test panel — click Approve
4. The `process_expense` tool call fires
5. Verify the output status is `processed`

### Step 8 — Register the Workflow as a Tool

Navigate to **Tool Studio → New Tool → Workflow**. Select your workflow. It is now available to attach to agents.

In Agent Builder, open your agent → **Tools → Add Tool → Workflow Tools** → select the workflow.

### Step 9 (Optional) — Configure a Schedule

In the Workflow Builder, click **Schedule** and enter a natural language schedule:

> "Every Monday at 8am"

!!! note "Minimum interval"
    Scheduled workflows have a minimum 5-minute interval.

### Callback Configuration (Long-Running Steps)

For tool call nodes that call external systems with indeterminate response times:

1. In the Tool Call node settings, enable **Callback mode**
2. The platform generates a unique callback URL: `https://api.orchestrate.cloud.ibm.com/v1/workflows/{instance_id}/callbacks/{node_id}`
3. Pass this URL to the external system in the request body
4. When the external system completes, it POSTs results to the callback URL
5. The workflow automatically resumes

## Verification

1. Trigger the workflow from the agent: "Process expense request EXP-99912"
2. The approval prompt appears in the chat interface
3. Approve — the `process_expense` tool call fires in the trace
4. The workflow execution history shows all nodes green

## See Also

- [Agentic Workflows](../reference/agentic-workflows.md) — Technical spec for all node types
- [Agent vs Workflow](../reference/agent-vs-workflow.md) — Decision framework
- [Cookbook: Approval Workflow](../cookbook/agentic-workflow-approval.md) — Full recipe with callback configuration
