# Lab 5: Agentic Workflows

**Objective:** Build a deterministic PTO approval workflow with a conditional branch and a human-in-the-loop approval gate, then attach it to the Acme HR Assistant as a callable tool.

**Estimated Time:** 60 minutes

**Prerequisites:**

!!! note "Before you start"
    - Completed [Lab 4](lab-04-agentic-memory.md)
    - The PTO calculator tool from Lab 2 must still be registered

---

## Part 1 — Design the Workflow

Before touching the Workflow Builder, map out what the workflow must do:

```
Input: employee_id, start_date, end_date

Step 1: Calculate PTO days (PTO Calculator tool)
Step 2: Check PTO balance (mock check — is balance >= requested days?)
Step 3: If days > 5 → Route to manager approval
        If days <= 5 → Auto-approve
Step 4a (approved): Submit PTO request (mock tool) + notify employee
Step 4b (rejected): Notify employee of rejection
```

---

## Part 2 — Build the Workflow

### Step 2.1 — Create a New Workflow

Navigate to **Workflow Builder → New Workflow**.
- **Name:** `PTO Approval Workflow`
- **Description:** `Processes a PTO request through calculation, balance check, and optional manager approval`

### Step 2.2 — Add the PTO Calculator Node

Click **+ Add Node → Tool Call**. Select `pto_calculator`.

Input mappings:
```
start_date: {{inputs.start_date}}
end_date: {{inputs.end_date}}
exclude_weekends: true
```

### Step 2.3 — Add a Condition Node (Days > 5?)

Click **+ Add Node → Condition**.

Configure branches:
- **Branch A (Approval Required):** `{{nodes.pto_calculator.output.pto_days_requested}} > 5`
- **Branch B (Auto-Approve):** `else`

### Step 2.4 — Add the Approval Node (Branch A)

On Branch A, click **+ Add Node → User Activity → Approval**.

Configure:

**Approval message:**
```
PTO Request Approval Required

Employee ID: {{inputs.employee_id}}
Start Date: {{inputs.start_date}}
End Date: {{inputs.end_date}}
Working Days Requested: {{nodes.pto_calculator.output.pto_days_requested}}

This request exceeds 5 days and requires manager approval.
Please review and approve or reject.
```

**Approver:** Set to yourself (for testing purposes)

**SLA timeout:** 24h

### Step 2.5 — Add a Condition Node on Approval Outcome

After the Approval node, add another **Condition** node:
- **Approved:** `{{nodes.approval_gate.output.decision}} == "approved"`
- **Rejected:** `else`

### Step 2.6 — Add Completion Nodes

For both approved paths (manual approval + auto-approve), add a final **Tool Call** node using the `get_hr_policy` tool (simulating a "submit request" action):

```
policy_code: SUBMIT-{{inputs.employee_id}}-{{nodes.pto_calculator.output.pto_days_requested}}d
```

For the rejected path, use the `get_hr_policy` tool with:
```
policy_code: REJECT-POLICY
```

### Step 2.7 — Configure Workflow Inputs and Outputs

In **Workflow Settings**:

```json
{
  "inputs": {
    "employee_id": { "type": "string", "description": "The employee ID requesting PTO (format: EMP-XXXXXX)" },
    "start_date": { "type": "string", "description": "PTO start date in YYYY-MM-DD format" },
    "end_date": { "type": "string", "description": "PTO end date in YYYY-MM-DD format" }
  },
  "outputs": {
    "result": "{{nodes.approval_gate.output.decision || 'auto-approved'}}",
    "pto_days": "{{nodes.pto_calculator.output.pto_days_requested}}"
  }
}
```

---

## Part 3 — Test the Workflow

### Step 3.1 — Test the Auto-Approve Path

Click **Test** in the Workflow Builder.

Enter:
- `employee_id`: `EMP-001`
- `start_date`: `2025-08-04`
- `end_date`: `2025-08-06`

**Expected path:** PTO calculator → Condition (3 days ≤ 5) → Auto-approve branch → Submit → Complete

### Step 3.2 — Test the Approval Path

Test again with:
- `employee_id`: `EMP-002`
- `start_date`: `2025-08-04`
- `end_date`: `2025-08-15`

**Expected path:** PTO calculator → Condition (8 days > 5) → Approval gate → *(approve it)* → Submit

When the Approval node appears, click **Approve** with a comment.

---

## Part 4 — Attach to the Agent

### Step 4.1 — Register as a Tool

Navigate to **Tool Studio → New Tool → Workflow**. Select `PTO Approval Workflow`. Save.

### Step 4.2 — Add to Agent

Open `Acme HR Assistant` in Agent Builder → **Tools → Add Tool** → select `PTO Approval Workflow`.

Update the system instructions to include:
```
When an employee asks to request PTO or time off, use the PTO Approval Workflow tool.
The workflow will calculate the days and route for approval if needed.
```

Save the agent.

### Step 4.3 — Test via Conversation

In the chat preview:
> "I'd like to book PTO for employee EMP-001 from the 4th to the 6th of August"

---

## Validation Checkpoints

- [ ] Auto-approve path completes without showing an approval prompt (3 days ≤ 5)
- [ ] Approval path shows the approval message with the correct day count
- [ ] Workflow execution history shows all nodes green after completion
- [ ] Agent invokes the workflow when asked for PTO booking

---

## What You Learned

- How to build a multi-step workflow with conditional branching
- How User Activity (Approval) nodes pause workflow execution and wait for human input
- How to expose a workflow as a tool and integrate it into an agent
- The difference between agent-driven responses and workflow-driven responses

---

**Next Lab:** [Lab 6 — Multi-Agent Collaboration](lab-06-multi-agent.md)
