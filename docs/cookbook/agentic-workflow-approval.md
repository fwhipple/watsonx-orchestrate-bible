# Approval Workflow with Human-in-the-Loop

## Problem

A business process requires a manager to explicitly approve or reject an action before it executes — for example, provisioning a resource, approving an expense, or granting access. The approval must be captured and auditable.

## Solution

Build an Agentic Workflow with a User Activity (Approval) node. The workflow suspends at the approval gate, notifies the approver via the chat channel, and resumes when the approver responds — either executing the action or sending a rejection notification.

## Prerequisites

- Two registered tools: one to fetch the pending request details, one to execute the approved action
- Builder role in watsonx Orchestrate
- An agent to invoke the workflow

## Steps

### Step 1 — Create the Workflow

Navigate to **Workflow Builder → New Workflow**. Name: `Expense Approval Workflow`.

### Step 2 — Add "Get Request" Tool Call Node

Click **+ Add Node → Tool Call**. Select `get_expense_request`.

Input mapping:
```
request_id: {{inputs.request_id}}
```

### Step 3 — Add the Approval Node

Click **+ Add Node → User Activity → Approval**.

Configure the approval message template:

```
New expense request requires your approval.

Employee: {{nodes.get_expense_request.output.employee_name}}
Amount: ${{nodes.get_expense_request.output.amount}}
Category: {{nodes.get_expense_request.output.category}}
Business justification: {{nodes.get_expense_request.output.justification}}

Please approve or reject this request.
```

Set **Approver** to the user's manager (resolved from the employee's HR record) or a static role.

Set **SLA timeout**: `48h` — if not approved within 48 hours, routes to the reject branch automatically.

### Step 4 — Add a Condition Node on the Approval Outcome

Click **+ Add Node → Condition**. Branch on the approval node output:

| Condition | Branch |
|-----------|--------|
| `{{nodes.approval_gate.output.decision}} == "approved"` | Approved path |
| `else` | Rejected path |

### Step 5 — Add "Process Expense" on the Approved Path

Click **+ Add Node → Tool Call**. Select `process_expense`.

Input mapping:
```
request_id: {{inputs.request_id}}
approved_by: {{nodes.approval_gate.output.approver_id}}
approval_comment: {{nodes.approval_gate.output.comment}}
```

### Step 6 — Add "Notify Rejection" on the Rejected Path

Click **+ Add Node → Tool Call**. Select `send_rejection_notification`.

Input mapping:
```
request_id: {{inputs.request_id}}
reason: {{nodes.approval_gate.output.comment}}
```

### Step 7 — Configure Workflow Inputs and Outputs

```json
{
  "inputs": {
    "request_id": {
      "type": "string",
      "description": "The expense request ID to process (format: EXP-XXXXXX)"
    }
  },
  "outputs": {
    "status": "{{nodes.process_expense.output.status || 'rejected'}}",
    "decision": "{{nodes.approval_gate.output.decision}}"
  }
}
```

### Step 8 — Configure the Async Callback (Optional)

If the approval system is external (e.g., a manager approval email link rather than in-chat), configure callback mode:

1. In the Approval node settings, enable **External Callback**
2. The platform generates a callback URL for this node instance:
   ```
   POST https://api.orchestrate.cloud.ibm.com/v1/workflows/{instance_id}/callbacks/{node_id}
   Authorization: Bearer <approval-system-token>
   Content-Type: application/json

   {
     "decision": "approved",
     "approver_id": "manager@example.com",
     "comment": "Approved for Q3 budget"
   }
   ```
3. Pass this callback URL to the external approval system via the notification tool

### Step 9 — Register as a Tool and Attach to Agent

In **Tool Studio → New Tool → Workflow**, select this workflow. Attach to the target agent in Agent Builder.

## Verification

1. Invoke the agent: "Process expense request EXP-99912"
2. The workflow starts; the approval prompt appears in the chat interface
3. Click **Approve** with a comment
4. The condition routes to the approved path; `process_expense` fires
5. In **Workflow Builder → Execution History**, all nodes show green completion status
6. The target system (expense processing API) reflects the approved expense

## See Also

- [Agentic Workflows](../reference/agentic-workflows.md) — Full node type specification
- [Agent vs Workflow](../reference/agent-vs-workflow.md) — When to use workflows vs agents
- [HOWTO: Build an Agentic Workflow](../howto/build-agentic-workflow.md) — Step-by-step build guide
