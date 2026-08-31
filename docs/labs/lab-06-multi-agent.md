# Lab 6: Multi-Agent Collaboration

**Objective:** Restructure the Acme HR setup as a multi-agent system — a primary orchestrator routes to specialist sub-agents for HR queries and IT helpdesk queries.

**Estimated Time:** 45 minutes

**Prerequisites:**

!!! note "Before you start"
    - Completed [Lab 5](lab-05-agentic-workflows.md)
    - The `Acme HR Assistant` agent from previous labs

---

## Part 1 — Understand the Architecture

Currently the `Acme HR Assistant` does everything. As more tools and capabilities are added, this becomes unwieldy — the context window grows, routing accuracy drops, and the agent becomes hard to maintain.

The solution is a **supervisor pattern**:

```
User
  │
  ▼
Acme Employee Assistant (orchestrator / supervisor)
  │
  ├─► HR Specialist Agent (handles PTO, policy, benefits)
  │
  └─► IT Helpdesk Agent (handles password resets, software requests)
```

The orchestrator receives all user messages, determines intent, and delegates to the right specialist. Each specialist has its own tools, instructions, and context window.

---

## Part 2 — Create the HR Specialist Agent

### Step 2.1 — Create the Agent

1. Agent Builder → **New Agent**
2. Name: `HR Specialist`
3. Description: `Specialist agent for HR queries — handles PTO requests, policy lookups, and benefits questions for Acme Corp employees`

### Step 2.2 — Write Instructions

```
You are the Acme Corp HR Specialist. Handle all HR-related requests including:
- PTO requests and calculations (use the PTO Approval Workflow)
- HR policy lookups (use get_hr_policy)
- Employee directory queries (use the MCP directory tools)

Be thorough and accurate. When you complete a request, summarise what you did.
```

### Step 2.3 — Add Tools

Add to this agent (move them from the orchestrator):
- `PTO Approval Workflow`
- `get_hr_policy`
- `Acme Directory Service` (MCP)
- `pto_calculator`

---

## Part 3 — Create the IT Helpdesk Agent

### Step 3.1 — Create the Agent

1. Agent Builder → **New Agent**
2. Name: `IT Helpdesk`
3. Description: `Specialist agent for IT support — handles password resets, software requests, and hardware issues for Acme Corp employees`

### Step 3.2 — Write Instructions

```
You are the Acme Corp IT Helpdesk. Handle IT support requests including:
- Password reset guidance
- Software access requests
- Hardware issue triage
- Network connectivity troubleshooting

If a request requires a human IT technician, escalate with a summary of the issue.
```

### Step 3.3 — Add a Simulated IT Tool

Register this simple OpenAPI spec as `it-tools.yaml`:

```yaml
openapi: 3.0.0
info:
  title: Acme IT Self-Service
  version: "1.0"
servers:
  - url: https://httpbin.org
paths:
  /get:
    get:
      operationId: get_it_ticket_status
      summary: Check the status of an IT support ticket
      parameters:
        - name: ticket_id
          in: query
          required: true
          description: The IT ticket ID to check (format: IT-XXXXX)
          schema:
            type: string
      responses:
        "200":
          description: Ticket status returned
```

Upload to Tool Studio and attach to the IT Helpdesk agent.

---

## Part 4 — Restructure the Orchestrator

### Step 4.1 — Update the Acme Employee Assistant

Open the `Acme HR Assistant`. Rename it to `Acme Employee Assistant`.

Update the instructions:

```
You are the Acme Corp Employee Assistant. Your job is to understand what each employee needs and route them to the right specialist.

You have two specialists:
- HR Specialist: handles everything HR-related (PTO, policies, benefits, employee directory)
- IT Helpdesk: handles everything IT-related (passwords, software, hardware, network)

For general questions you can answer directly (e.g., office hours, company information), answer them yourself.
For HR or IT requests, always delegate to the appropriate specialist.

Do not attempt to answer HR or IT questions yourself — always route them.
```

### Step 4.2 — Add Collaborators

In Agent Builder → **Collaborators → Add Collaborator**:
1. Add `HR Specialist` 
2. Add `IT Helpdesk`

### Step 4.3 — Remove Directly Attached Tools

Remove all the HR and IT tools from the orchestrator. The orchestrator should now have:
- Weather tool (direct tool — it can answer this itself)
- HR Specialist (collaborator)
- IT Helpdesk (collaborator)

---

## Part 5 — Test Routing

In the chat preview, try these:

| Message | Expected Router | Expected Agent |
|---------|----------------|----------------|
| "I need to request PTO for next week" | → HR Specialist | PTO Approval Workflow |
| "My password has expired" | → IT Helpdesk | IT support guidance |
| "What's the weather in Sydney?" | Direct | Orchestrator (weather tool) |
| "Can you look up employee EMP-002 and also check my IT ticket IT-54321?" | Both | HR Specialist + IT Helpdesk |

---

## Validation Checkpoints

- [ ] PTO request routes to the HR Specialist (verify in trace)
- [ ] IT ticket request routes to the IT Helpdesk (verify in trace)
- [ ] Weather question is handled directly by the orchestrator
- [ ] Multi-domain request correctly delegates to both specialists

---

## What You Learned

- The supervisor/collaborator pattern for multi-agent architectures
- How to decompose a monolithic agent into specialist sub-agents
- How routing works: the orchestrator reads collaborator descriptions to decide delegation
- Why clear collaborator descriptions are critical for correct routing

---

**Next Lab:** [Lab 7 — Evaluation & Optimization](lab-07-evaluation.md)
