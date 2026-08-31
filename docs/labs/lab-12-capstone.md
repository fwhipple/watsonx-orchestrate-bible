# Lab 12: Capstone — End-to-End Enterprise Deployment

**Objective:** Bring together every feature covered in Labs 1–11 into a production-ready enterprise deployment of the Acme Employee Assistant. By the end of this lab, you will have a fully instrumented, multi-agent, multi-channel agent system with authentication, memory, workflows, evaluation, observability, and load testing in place.

**Estimated Time:** 90 minutes

**Prerequisites:**

!!! note "Before you start"
    - Completed Labs 1–11
    - All agents, tools, connections, and channels from previous labs still configured

---

## Capstone Architecture

The completed Acme Employee Assistant system looks like this:

```
┌─────────────────────────────────────────────────────────────┐
│                    Channels                                   │
│   Slack (SSO/Okta) │ Embed Chat (Voice, Watson STT/TTS)     │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│        Acme Employee Assistant (Orchestrator)                │
│  - Weather Tool (direct)                                     │
│  - Memory: enabled                                           │
│  - Context-aware formatting                                  │
└─────────┬──────────────┬──────────────────┬─────────────────┘
          │              │                  │
          ▼              ▼                  ▼
   ┌──────────┐   ┌────────────┐   ┌─────────────────┐
   │HR        │   │IT Helpdesk │   │Research Agent   │
   │Specialist│   │            │   │(LangGraph/Ext.) │
   │          │   │            │   │                 │
   │Tools:    │   │Tools:      │   │Traces forwarded │
   │- PTO Wf  │   │- IT ticket │   │to Control Plane │
   │- Policy  │   │            │   └─────────────────┘
   │- Directory│  └────────────┘
   └──────────┘
          │
    OAuth 2 Connections
    (API Key + OBO pattern)
          │
    Workday / ServiceNow (simulated)
```

---

## Part 1 — Verify the Full Stack

### Step 1.1 — Run the Pre-Deployment Checklist

Go through each item and verify it is working:

**Agents:**
- [ ] `Acme Employee Assistant` (orchestrator) — has Weather tool, HR Specialist collaborator, IT Helpdesk collaborator, Research Agent collaborator
- [ ] `HR Specialist` — has PTO Workflow, HR Policy, Directory tools
- [ ] `IT Helpdesk` — has IT Ticket tool
- [ ] `Research Agent` (external/LangGraph) — running and registered

**Tools:**
- [ ] `get_weather_forecast` (OpenAPI) — attached to orchestrator
- [ ] `get_hr_policy` (OpenAPI with API key auth) — attached to HR Specialist
- [ ] `pto_calculator` (Python) — attached to HR Specialist
- [ ] `Acme Directory Service` (MCP) — attached to HR Specialist
- [ ] `PTO Approval Workflow` (Workflow) — attached to HR Specialist
- [ ] `get_it_ticket_status` (OpenAPI) — attached to IT Helpdesk

**Memory:** Enabled on orchestrator, user opt-in confirmed

**Channels:**
- [ ] Slack channel configured (with Okta SSO)
- [ ] Embed Chat voice widget configured (Watson Speech + Watson TTS)

**Observability:**
- [ ] OTLP export to New Relic (or Datadog) configured

---

## Part 2 — Run the End-to-End User Journeys

Execute each scenario and verify correct behaviour end-to-end:

### Journey 1: New Employee Onboarding

In the Agent Builder chat preview (simulating a new employee's first interaction):

1. "Hi! I just joined Acme. My name is Sam and I'm in the London office."
2. "What are the core working hours here?"
3. "Can you look up my manager? Her name is Alice Chen."
4. "I'd like to book time off for my first week of December. Can you process that?"
5. *(Follow the approval workflow — approve it)*
6. "Thanks. What's the weather forecast for London this week?"

**Verify:**
- Agent remembered Sam's name and London office in subsequent turns
- Directory lookup routed to HR Specialist
- PTO request triggered the PTO Approval Workflow
- Approval gate appeared and was actioned
- Weather query was handled directly by the orchestrator

### Journey 2: IT Support

In Slack (to test the channel):

1. "@Acme Employee Assistant My laptop won't connect to the VPN. Ticket number IT-99876."
2. "The ticket says 'in progress' — what does that mean?"

**Verify:**
- Message received in Slack
- IT ticket lookup routed to IT Helpdesk
- Response appears in Slack with correct formatting (markdown)

### Journey 3: Research Request

Via the voice widget (to test voice):

1. *(Speak)* "I need some research on the latest trends in enterprise AI adoption"
2. *(Speak)* "Can you summarise the key findings?"

**Verify:**
- STT correctly transcribes both utterances
- Research Agent is invoked as collaborator
- TTS reads back the research summary naturally
- Hold message plays during the research processing

---

## Part 3 — Run the Final Evaluation

### Step 3.1 — Upload a Capstone Test Set

Add 3 new test cases to `test-cases.json` that cover the capstone journeys, then re-upload and run a final evaluation:

```json
{
  "id": "tc-capstone-001",
  "description": "Capstone: New employee onboarding PTO request",
  "conversation": [
    { "role": "user",  "content": "I'm Sam, a new employee in London. I want to book December 1st to 5th off." }
  ],
  "expected": {
    "tool_calls": [{ "tool_name": "PTO Approval Workflow", "arguments": {} }],
    "journey_completed": true
  }
},
{
  "id": "tc-capstone-002",
  "description": "Capstone: Multi-domain request (HR + IT)",
  "conversation": [
    { "role": "user", "content": "I need to look up employee EMP-002 and also check IT ticket IT-44567" }
  ],
  "expected": {
    "tool_calls": [
      { "tool_name": "lookup_employee", "arguments": { "employee_id": "EMP-002" } },
      { "tool_name": "get_it_ticket_status", "arguments": { "ticket_id": "IT-44567" } }
    ],
    "journey_completed": true
  }
}
```

Run the evaluation and verify Journey Success ≥ 85%.

---

## Part 4 — Run a Final Load Test

Run a 5-minute load test at 20 concurrent users to validate the full stack under load:

```bash
k6 run \
  -e IBM_API_KEY=<your-api-key> \
  -e AGENT_ID=<your-agent-id> \
  --vus 20 \
  --duration 5m \
  acme-load-test.js
```

Record your P95 latency and error rate.

---

## Part 5 — Review the Control Plane

Navigate to the Control Plane and review each tab for the Acme Employee Assistant:

| Tab | What to Verify |
|-----|---------------|
| **Overview** | Message success rate ≥ 95%, positive feedback ratio |
| **Adoption** | Conversations recorded, users engaged |
| **Analytics** | Latency P95 within SLA, error rate < 2% |
| **FinOps** | Token consumption recorded per model |
| **Quality** | Helpfulness score visible; evaluation runs listed |
| **Reliability** | Deployment readiness green, P95 within target |
| **Security** | No control violations flagged |

Ask the Control Plane AI assistant:
> "Give me a health summary for the Acme Employee Assistant"

---

## Part 6 — Reflection

Answer these questions to consolidate what you built:

1. **Architecture decision:** Why did we use a workflow for PTO approval instead of just asking the agent to handle it conversationally?

2. **Routing accuracy:** If the orchestrator is routing IT queries to the HR Specialist, what would you change to fix it?

3. **Observability:** A P99 latency spike appears in the trace. Which span would you investigate first and why?

4. **Evaluation:** Your Journey Success drops from 90% to 70% after updating the system instructions. What would you do?

5. **Scaling:** The load test shows P95 = 12s at 20 users. What three things would you check first?

---

## Validation Checkpoints

- [ ] All 3 user journeys completed successfully end-to-end
- [ ] Journey 2 works correctly via Slack (not just chat preview)
- [ ] Journey 3 works via voice widget with STT and TTS
- [ ] Final evaluation shows Journey Success ≥ 85%
- [ ] Load test at 20 users completes with error rate < 2%
- [ ] Control Plane shows green health across all 7 tabs
- [ ] Control Plane AI assistant responds correctly to the health summary request

---

## What You Built

Over 12 labs, you have built and deployed a fully production-ready enterprise AI agent system:

| Capability | Lab | Feature |
|-----------|-----|---------|
| Agent creation and tools | 1, 2 | OpenAPI, Python, MCP tools |
| Authentication | 3 | API Key, OAuth 2, team/member credentials |
| Memory | 4 | Cross-session persistence, sensitivity classification |
| Deterministic workflows | 5 | Conditional branching, human approval gates |
| Multi-agent architecture | 6 | Supervisor/collaborator pattern |
| Automated quality assurance | 7 | Evaluation metrics, ACE optimization |
| Distributed tracing | 8 | Custom span attributes, OTLP export |
| Channel deployment | 9 | Slack SSO, voice widget, channel-aware formatting |
| Performance validation | 10 | K6 load test, SLA gate, stress testing |
| External agents | 11 | LangGraph integration, Control Plane AI assistant |
| Full-stack integration | 12 | End-to-end production deployment |

**Congratulations — you have completed the watsonx Orchestrate Bible Lab Series.**
