# Control Plane

The watsonx Orchestrate Control Plane is the administrative and governance layer for agent fleets. It provides a unified dashboard for monitoring agent health, tracking adoption and usage, managing costs, and enforcing operational controls — all in one place.

The Control Plane includes a built-in **AI assistant** that allows administrators and builders to query fleet data in natural language, investigate failing agents, and surface insights without navigating through multiple screens.

---

## Dashboard Tabs

The Control Plane dashboard is organised into seven tabs.

### Tab 1 — Overview

The landing tab. Provides a high-level snapshot of platform health and engagement.

**Metrics displayed:**

- Overall message success rate
- User feedback ratio (positive vs negative)
- Deployment status summary (agents in draft vs live)
- Agent type breakdown (native, imported, external)
- Control enforcement status (what percentage of agents have controls applied)
- Usage trend charts (messages over time)
- Token consumption trend

**Use for:** Executive-level health checks; detecting platform-wide degradation at a glance.

### Tab 2 — Adoption

Tracks how widely agents are being used and whether adoption is growing.

**Metrics displayed:**

- Active users per agent
- Conversation counts per agent
- Messages per conversation (engagement depth)
- User retention trends

**Use for:** Identifying underused agents, validating rollout success, prioritising which agents need improvement.

### Tab 3 — Analytics

Row-level agent performance data in sortable, searchable tables.

**Metrics per agent:**

| Metric | Description |
|--------|-------------|
| Conversations | Total conversation count in period |
| Unique Users | Count of distinct users |
| Avg. Duration | Mean conversation length |
| Token Consumption | Input + output tokens |
| LLM Calls | Number of model inference calls |
| Error Rate | Percentage of failed sessions |
| Latency P95 | 95th percentile end-to-end response time |

**Use for:** Identifying the highest-cost or highest-error agents; SLA reporting; capacity planning.

### Tab 4 — FinOps

Token spend tracking and model-level cost breakdown.

**Features:**

- Total token spend split by input vs output tokens
- Cost breakdown by model (Granite vs Claude vs Llama)
- LLM execution count trends
- Per-agent token cost ranking (highest spenders at the top)
- Trend visualisations (daily/weekly/monthly views)

**Use for:** Cost governance, identifying agents with runaway token consumption, rightsizing model selection.

### Tab 5 — Quality

User satisfaction and agent accuracy metrics.

**Metrics displayed:**

- Thumbs up / thumbs down ratio per agent
- Tool-specific success rates (which tools fail most often)
- Helpfulness score (LLM-as-judge, automatic)
- Hallucination score (LLM-as-judge, automatic)
- Agent performance ranking by quality score

**Use for:** Identifying agents that need prompt refinement; prioritising Agent Ops evaluation campaigns; tracking quality improvement over time.

### Tab 6 — Reliability

Latency and error rate metrics with operational readiness indicators.

**Metrics displayed:**

- Latency distribution: P50, P95, P99 per agent
- Error rate percentage per agent
- Deployment readiness indicators
- Runtime inventory (agents, tools, knowledge bases — counts and status)

**Use for:** Pre-production readiness checks; SLA compliance monitoring; post-incident analysis.

### Tab 7 — Security

Governance and controls enforcement across the agent fleet.

**Metrics displayed:**

- Controls enforcement status (percentage of agents with controls applied)
- Per-agent control compliance view
- Model usage policy enforcement
- Recent control creation events

**Use for:** Security posture reviews; compliance audits; ensuring all production agents meet governance requirements.

---

## Multi-Persona Support

The Control Plane serves two primary personas:

| Persona | Access Level | Typical Use |
|---------|-------------|-------------|
| **Platform Administrator** | Full fleet view — all agents across all workspaces | Platform health, cost governance, security audits |
| **Agent Builder** | Workspace-scoped view — agents in their workspace only | Individual agent performance, quality monitoring |

!!! note "Workspace filtering"
    Workspace-level filtering (showing only the current builder's workspace) is in the development pipeline targeting the **September release**. Until then, builders see fleet-wide data.

---

## Control Plane AI Assistant

The Control Plane includes a built-in AI assistant that allows natural language interaction with fleet data.

### Capabilities

- **Natural language queries** — ask in plain English instead of clicking through multiple screens
  - "Which agents have an error rate above 5% this week?"
  - "Show me the top 3 token consumers in the last 30 days"
  - "Why is the Hotel Finder agent failing?"

- **Data aggregation** — the assistant compiles multi-page insights into a single response

- **Automatic visualisation** — generates charts automatically when the answer benefits from a visual representation

- **Root cause investigation** — analyses trace data to surface the specific error message, tool failure, or configuration issue causing agent failures

### Example: Investigating a Failing Agent

> **Admin:** Why is the Hotel Finder agent failing?
>
> **Control Plane AI:** The Hotel Finder agent has a 23% error rate over the last 48 hours. Trace analysis shows the failures are concentrated in the `search_hotels` tool. The errors are HTTP 403 responses — the API key in the Hotel Finder connection appears to have expired or been revoked. I recommend rotating the API key in the Connections Manager and re-testing.

### Roadmap

!!! note "Upcoming Control Plane AI enhancements"
    - **Merge with Agent Ops** — the Control Plane AI assistant and the Agent Ops evaluation agent will be merged into a single assistant for diagnostics + optimisation in a future release
    - **Callback support** — asynchronous processing for long-running analyses
    - **Scheduling** — schedule recurring health reports to be delivered automatically
    - **Channel expansion** — currently tested on web embed; Slack testing in progress

---

## External Agent Support

The Control Plane supports ingestion of traces from agents built outside watsonx Orchestrate.

### Trace Export API

External agents (LangGraph, Amazon Bedrock, CrewAI, etc.) can submit trace data to Orchestrate via the batch trace import API. Once imported, traces appear in the Control Plane alongside native agent traces.

```bash
POST /v1/traces/import
Content-Type: application/json
Authorization: Bearer <api-key>

{
  "traces": [
    {
      "trace_id": "abc123",
      "agent_id": "my-external-agent",
      "spans": [ ... ]  // OpenTelemetry span format
    }
  ]
}
```

### External OTLP Collector

Conversely, Orchestrate can push its own traces outward to customer observability platforms. See [Observability — External OTLP Export](observability.md#external-otlp-export) for configuration details.

---

## Related References

- [Observability](observability.md) — Trace stack architecture, span details, OTLP export
- [Evaluation](evaluation.md) — Agent Ops evaluation that feeds Quality tab data
- [Optimization](optimization.md) — JEPA/ACE prompt optimisation connected to Control Plane
- [Cookbook: LangGraph External Agent](../cookbook/langgraph-external-agent.md) — Registering a LangGraph agent and piping traces to Control Plane
- [Lab 11: External Agents & Control Plane](../labs/lab-11-external-agents.md) — Hands-on lab using the Control Plane AI assistant
