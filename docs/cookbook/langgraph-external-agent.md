# Connect a LangGraph Agent as an External Collaborator

## Problem

An existing LangGraph-based specialist agent (e.g., a research or code-generation agent) needs to be available as a collaborator in a watsonx Orchestrate multi-agent topology, and its execution traces need to appear in the Orchestrate Control Plane alongside native agent traces.

## Solution

Expose the LangGraph agent via an HTTP endpoint compatible with Orchestrate's external agent protocol, register it as an External Agent in Orchestrate, configure collaborator routing on the primary agent, and forward LangGraph traces to the Orchestrate trace import API.

## Prerequisites

- Running LangGraph agent (Python, with LangGraph 0.2+)
- FastAPI or Flask for the HTTP wrapper
- watsonx Orchestrate admin role and API key
- Primary Orchestrate agent already created

## Steps

### Step 1 — Wrap the LangGraph Agent in an HTTP Endpoint

Create `agent_server.py`:

```python
from fastapi import FastAPI, Header, HTTPException
from pydantic import BaseModel
from langgraph.graph import StateGraph, END
from langchain_core.messages import HumanMessage, AIMessage
from typing import Optional
import os

app = FastAPI(title="Research Agent")

# LangGraph graph definition (abbreviated)
from your_agent import create_research_graph  # Your graph factory

class AgentRequest(BaseModel):
    session_id: str
    input: str
    context: Optional[dict] = {}

class AgentResponse(BaseModel):
    output: str
    session_id: str

@app.post("/invoke", response_model=AgentResponse)
async def invoke(
    request: AgentRequest,
    authorization: str = Header(...)
):
    # Validate the Orchestrate service token
    token = authorization.replace("Bearer ", "")
    if token != os.environ["ORCHESTRATE_SERVICE_TOKEN"]:
        raise HTTPException(status_code=401, detail="Unauthorized")

    graph = create_research_graph()
    result = graph.invoke({
        "messages": [HumanMessage(content=request.input)],
        "session_id": request.session_id
    })

    final_message = result["messages"][-1]
    return AgentResponse(
        output=final_message.content,
        session_id=request.session_id
    )

@app.get("/health")
async def health():
    return {"status": "ok"}
```

Run with: `uvicorn agent_server:app --host 0.0.0.0 --port 8080`

### Step 2 — Register as an External Agent in Orchestrate

1. Navigate to **Agents → External Agents → Register External Agent**
2. Name: `Research Agent`
3. Endpoint URL: `https://research-agent.example.com/invoke`
4. Auth type: **Bearer Token** → enter the `ORCHESTRATE_SERVICE_TOKEN`
5. Click **Save**

### Step 3 — Add as Collaborator on Primary Agent

1. Open the primary agent in Agent Builder
2. Navigate to **Collaborators → Add Collaborator**
3. Select `Research Agent` from the external agents list
4. Set the routing description: `"Handles research tasks, market analysis, and in-depth investigative queries"`
5. Save the agent

### Step 4 — Forward LangGraph Traces to Control Plane

Add trace forwarding to your LangGraph agent. Call this after each invocation:

```python
import httpx
import os

ORCHESTRATE_API_KEY = os.environ["ORCHESTRATE_API_KEY"]
ORCHESTRATE_BASE_URL = "https://api.orchestrate.cloud.ibm.com"

async def send_trace_to_orchestrate(
    trace_id: str,
    agent_id: str,
    spans: list[dict]
):
    """Forward OpenTelemetry spans to Orchestrate trace import API."""
    async with httpx.AsyncClient() as client:
        response = await client.post(
            f"{ORCHESTRATE_BASE_URL}/v1/traces/import",
            json={
                "traces": [
                    {
                        "trace_id": trace_id,
                        "agent_id": agent_id,
                        "spans": spans
                    }
                ]
            },
            headers={
                "Authorization": f"Bearer {ORCHESTRATE_API_KEY}",
                "Content-Type": "application/json"
            }
        )
        response.raise_for_status()

# In your invoke endpoint, after running the graph:
# await send_trace_to_orchestrate(
#     trace_id=str(uuid4()),
#     agent_id="research-agent-external",
#     spans=extract_otel_spans(result)  # Convert LangGraph run to OTel spans
# )
```

### Step 5 — Test the Collaborator Routing

Send a message to the primary agent:

> "Research the latest trends in enterprise AI adoption for 2025"

The primary agent should route this to the Research Agent collaborator. Watch the trace in the Orchestrate Observability UI — the collaborator call span should appear.

## Verification

1. In the Orchestrate **Observability UI → Agent Traces**, find the conversation trace
2. Confirm a collaborator span appears showing the call to `Research Agent`
3. In **Control Plane → Analytics tab**, verify the external agent appears as a registered agent entry
4. If trace forwarding is configured, verify the LangGraph spans appear nested under the collaborator span

## See Also

- [Control Plane](../reference/control-plane.md#external-agent-support) — External agent trace import API
- [Agent Building Blocks](../reference/agent-building-blocks.md#collaborators) — Collaborator architecture
- [Lab 11: External Agents & Control Plane](../labs/lab-11-external-agents.md) — Hands-on external agent lab
