# Add a Tool to an Agent

Register a tool with watsonx Orchestrate and attach it to an agent.

## Prerequisites

!!! note "Prerequisites"
    - Agent already created (see [Create an Agent](create-agent.md))
    - For OpenAPI tools: a valid OpenAPI 3.0 or 3.1 specification file
    - For Python tools: a Python 3.11+ script with an async `main` entrypoint
    - For MCP tools: a running MCP server with a reachable SSE or stdio endpoint

---

## Adding an OpenAPI Tool

### Step 1 — Upload the Specification

Navigate to **Tool Studio → New Tool → OpenAPI**. Upload your OpenAPI JSON or YAML file, or paste the spec directly.

### Step 2 — Review Operations

The platform parses each operation (`operationId`). Review and, if needed, improve the `summary` and parameter `description` fields directly in the Tool Studio editor.

!!! tip "Description quality is critical"
    The LLM reads `description` fields to map user utterances to tool arguments. A vague description like `"The ID"` causes hallucinated values. Write it like `"The unique customer identifier, format CUST-XXXXXX"`.

### Step 3 — Bind a Connection

In the tool's **Authentication** section, select the connection that provides credentials for this API. If no connection exists yet, see [Configure a Connection](configure-connection.md) first.

### Step 4 — Attach to Agent

Open the agent in Agent Builder. Navigate to **Tools → Add Tool** and select the newly registered OpenAPI tool.

---

## Adding a Python Tool

### Step 1 — Write the Script

Create a Python 3.11+ script with a typed async entrypoint:

```python
async def main(customer_id: str, include_history: bool = False) -> dict:
    """Look up customer details from the CRM.
    
    Args:
        customer_id: The unique customer identifier (format: CUST-XXXXXX)
        include_history: Set to true to include order history in the response
    """
    # Your business logic here
    data = await crm_client.get_customer(customer_id, history=include_history)
    return {"name": data.name, "tier": data.tier, "orders": data.orders if include_history else []}
```

The docstring is parsed to generate the tool's JSON schema. Type hints determine parameter types.

### Step 2 — Upload to Tool Studio

Navigate to **Tool Studio → New Tool → Python**. Upload the script file. The platform parses the entrypoint and shows the generated schema for review.

!!! warning "Testing requires agent invocation"
    Python tools cannot be tested standalone in Tool Studio. Test your script locally with mock inputs first, then validate by invoking the agent.

### Step 3 — Attach to Agent

Same as for OpenAPI tools: Agent Builder → Tools → Add Tool.

---

## Adding an MCP Server

### Step 1 — Register the Endpoint

Navigate to **Tool Studio → New Tool → MCP Server**. Enter the MCP server URL:

- **SSE transport**: `https://your-mcp-server.example.com/sse`
- **stdio transport**: configure via the MCP server manifest

### Step 2 — Auto-Discovery

Click **Connect**. The platform queries the MCP server for its tool manifest and displays all discovered tools. Review and select which tools to expose to agents.

### Step 3 — Attach to Agent

Agent Builder → Tools → Add Tool → select the MCP server. All discovered tools become available to the agent.

---

## Verification

1. Open the agent in the Builder chat preview
2. Send a message designed to trigger the tool (e.g., "Look up customer CUST-001234")
3. Open the **Traces** panel — confirm the tool span appears with status `success` and the expected response payload

## See Also

- [Tool Types](../reference/tool-types.md) — Full technical specification for each tool type
- [Configure a Connection](configure-connection.md) — Set up authentication for OpenAPI tools
- [Cookbook: Python MCP Tool](../cookbook/mcp-tool-python.md) — End-to-end recipe for building an MCP server
