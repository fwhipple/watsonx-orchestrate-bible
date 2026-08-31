# Lab 11: External Agents & Control Plane

**Objective:** Register a simple LangGraph-based external agent as a collaborator in the Acme Employee Assistant, forward its traces to the Orchestrate Control Plane, and use the Control Plane AI assistant to investigate performance.

**Estimated Time:** 60 minutes

**Prerequisites:**

!!! note "Before you start"
    - Completed [Lab 10](lab-10-load-testing.md)
    - Python 3.11+ with FastAPI and LangGraph: `pip install fastapi uvicorn langgraph langchain-core httpx`
    - An Orchestrate API key

---

## Part 1 — Build the LangGraph External Agent

We will build a simple "Research Agent" that answers questions using a basic web search simulation.

### Step 1.1 — Create the Agent Server

Create `research_agent_server.py`:

```python
from fastapi import FastAPI, Header, HTTPException
from pydantic import BaseModel
from langgraph.graph import StateGraph, END
from langchain_core.messages import HumanMessage, AIMessage
from typing import Optional, TypedDict
import os
import httpx
from uuid import uuid4

app = FastAPI(title="Acme Research Agent")

# ---- LangGraph Graph Definition ----

class AgentState(TypedDict):
    messages: list
    search_results: Optional[str]

async def research_node(state: AgentState) -> AgentState:
    """Simulate a web research step."""
    query = state["messages"][-1].content
    # Simulate research by echoing the query with enrichment
    # In production, this would call a real search API
    search_result = f"Research findings for '{query}': Based on current market data and industry reports, this topic shows significant activity. Key data points include recent developments in Q3 2025."
    return {"search_results": search_result}

async def synthesis_node(state: AgentState) -> AgentState:
    """Synthesise the research into a coherent response."""
    query = state["messages"][-1].content
    research = state.get("search_results", "No research available")
    response = f"Research Summary: {research}\n\nThis analysis addresses your question about: {query}"
    state["messages"].append(AIMessage(content=response))
    return state

def create_research_graph():
    workflow = StateGraph(AgentState)
    workflow.add_node("research", research_node)
    workflow.add_node("synthesis", synthesis_node)
    workflow.set_entry_point("research")
    workflow.add_edge("research", "synthesis")
    workflow.add_edge("synthesis", END)
    return workflow.compile()

# ---- FastAPI Endpoint ----

class AgentRequest(BaseModel):
    session_id: str
    input: str
    context: Optional[dict] = {}

class AgentResponse(BaseModel):
    output: str
    session_id: str
    trace_id: str

SERVICE_TOKEN = os.environ.get("ORCHESTRATE_SERVICE_TOKEN", "lab-service-token-123")

@app.post("/invoke", response_model=AgentResponse)
async def invoke(request: AgentRequest, authorization: str = Header(...)):
    token = authorization.replace("Bearer ", "")
    if token != SERVICE_TOKEN:
        raise HTTPException(status_code=401, detail="Unauthorized")

    graph = create_research_graph()
    result = graph.invoke({
        "messages": [HumanMessage(content=request.input)],
        "search_results": None
    })

    trace_id = str(uuid4()).replace("-", "")
    return AgentResponse(
        output=result["messages"][-1].content,
        session_id=request.session_id,
        trace_id=trace_id
    )

@app.get("/health")
async def health():
    return {"status": "ok", "agent": "Research Agent"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1", port=9090)
```

### Step 1.2 — Start the Research Agent Server

```bash
ORCHESTRATE_SERVICE_TOKEN=lab-service-token-123 python research_agent_server.py
```

The server starts on `http://127.0.0.1:9090`.

### Step 1.3 — Test the Server

```bash
curl -X POST http://127.0.0.1:9090/invoke \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer lab-service-token-123" \
  -d '{"session_id": "test-123", "input": "What are the latest trends in enterprise AI?"}'
```

You should get a research summary response.

---

## Part 2 — Register as External Agent in Orchestrate

### Step 2.1 — Register the Agent

1. Navigate to **Agents → External Agents → Register External Agent**
2. Fill in:
   - **Name:** `Research Agent`
   - **Description:** `Specialist agent for research tasks — provides in-depth analysis, market research, and investigative summaries`
   - **Endpoint URL:** `http://127.0.0.1:9090/invoke` (for local testing)
   - **Auth type:** Bearer Token
   - **Token:** `lab-service-token-123`
3. Click **Save and Test** — verify the health check passes

!!! note "Production deployment"
    In production, deploy the agent to OpenShift or Kubernetes with an HTTPS endpoint. See [Cookbook: LangGraph External Agent](../cookbook/langgraph-external-agent.md) for the full deployment recipe.

---

## Part 3 — Add as Collaborator

### Step 3.1 — Add to Primary Agent

1. Open `Acme Employee Assistant` in Agent Builder
2. Navigate to **Collaborators → Add Collaborator**
3. Select `Research Agent`
4. Routing description is pre-populated from the registration — verify it's accurate
5. Save

### Step 3.2 — Update Instructions

Add to the orchestrator's system instructions:

```
You also have access to a Research Agent specialist.
Delegate to the Research Agent when users ask for:
- Market research or industry analysis
- In-depth investigation of a topic
- Trend analysis or competitive intelligence
```

### Step 3.3 — Test Routing

In the chat preview:
> "I need a research summary on enterprise AI adoption trends in financial services"

Verify in the trace that the orchestrator delegates to the Research Agent collaborator.

---

## Part 4 — Forward Traces to the Control Plane

Add trace forwarding to the research agent server. Update `research_agent_server.py`:

```python
import httpx
import os

ORCHESTRATE_API_KEY = os.environ.get("ORCHESTRATE_API_KEY", "")
ORCHESTRATE_BASE_URL = "https://api.orchestrate.cloud.ibm.com"

async def forward_trace_to_orchestrate(trace_id: str, session_id: str, spans: list):
    """Forward trace spans to Orchestrate Control Plane."""
    if not ORCHESTRATE_API_KEY:
        return  # Skip if no API key configured
    
    payload = {
        "traces": [{
            "trace_id": trace_id,
            "agent_id": "research-agent-external",
            "agent_name": "Research Agent",
            "spans": spans
        }]
    }
    
    async with httpx.AsyncClient(timeout=10.0) as client:
        try:
            await client.post(
                f"{ORCHESTRATE_BASE_URL}/v1/traces/import",
                json=payload,
                headers={
                    "Authorization": f"Bearer {ORCHESTRATE_API_KEY}",
                    "Content-Type": "application/json"
                }
            )
        except Exception as e:
            print(f"Trace forwarding failed (non-critical): {e}")

# In the invoke endpoint, add after graph execution:
# spans = [
#     {
#         "span_id": str(uuid4()).replace("-", "")[:16],
#         "operation_name": "research_agent.invoke",
#         "start_time": start_time_ms,
#         "end_time": end_time_ms,
#         "attributes": {
#             "session.id": request.session_id,
#             "agent.name": "Research Agent",
#             "llm.output_tokens": len(result["messages"][-1].content.split())
#         }
#     }
# ]
# await forward_trace_to_orchestrate(trace_id, request.session_id, spans)
```

Restart the server with your API key:

```bash
ORCHESTRATE_SERVICE_TOKEN=lab-service-token-123 \
ORCHESTRATE_API_KEY=<your-api-key> \
python research_agent_server.py
```

---

## Part 5 — Use the Control Plane AI Assistant

### Step 5.1 — Open the Control Plane

Navigate to **Control Plane**. You should now see the `Research Agent` appear as an external agent in the Analytics tab.

### Step 5.2 — Ask the Control Plane AI Assistant

Click the **AI Assistant** button in the Control Plane. Try these queries:

1. "How many conversations has the Research Agent had today?"
2. "What is the average response latency for the Research Agent?"
3. "Are there any agents with an error rate above 5%?"
4. "Show me the top 3 agents by token consumption this week"

### Step 5.3 — Investigate a Trace

Ask: "Show me the latest trace for the Research Agent"

The assistant should surface the trace detail, including the span data forwarded from the LangGraph agent.

---

## Validation Checkpoints

- [ ] Research Agent server starts and responds to `/health` and `/invoke`
- [ ] External agent registered in Orchestrate and test passes
- [ ] Orchestrator correctly routes research queries to the Research Agent collaborator
- [ ] Research Agent spans appear in the Control Plane Analytics tab
- [ ] Control Plane AI assistant responds to natural language queries about agent performance

---

## What You Learned

- How to build and expose a LangGraph agent as an Orchestrate-compatible external agent
- How to register an external agent and configure collaborator routing
- How to forward traces from an external agent to the Orchestrate Control Plane
- How to use the Control Plane AI assistant for natural language diagnostics

---

**Next Lab:** [Lab 12 — Capstone](lab-12-capstone.md)
