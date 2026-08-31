# Build and Register a Python MCP Tool

## Problem

You need a custom tool with Python logic (data transformations, multi-step aggregation, custom libraries) that isn't expressible as a simple OpenAPI call, and you want to follow the open MCP standard so it works across AI frameworks.

## Solution

Build a Python MCP server using the `mcp` library, containerise it with a Red Hat UBI base image, deploy it, and register the SSE endpoint in Orchestrate Tool Studio.

## Prerequisites

- Python 3.11+
- Docker or Podman
- A container registry (IBM Container Registry, Docker Hub private, or similar)
- watsonx Orchestrate Builder role

## Steps

### Step 1 — Scaffold the MCP Server

Install the MCP library:

```bash
pip install mcp[server]
```

Create `server.py`:

```python
from mcp.server.fastmcp import FastMCP
import httpx

mcp = FastMCP("Customer Lookup Service")

@mcp.tool()
async def get_customer_details(customer_id: str) -> dict:
    """Look up customer details by ID.
    
    Args:
        customer_id: The unique customer identifier (format: CUST-XXXXXX)
    """
    async with httpx.AsyncClient() as client:
        resp = await client.get(
            f"https://crm.internal.example.com/api/customers/{customer_id}",
            headers={"Authorization": f"Bearer {get_service_token()}"}
        )
        resp.raise_for_status()
        return resp.json()

@mcp.tool()
async def list_customer_orders(customer_id: str, status: str = "all") -> list:
    """List orders for a customer.
    
    Args:
        customer_id: The unique customer identifier (format: CUST-XXXXXX)
        status: Filter by order status — one of: all, open, closed, cancelled
    """
    async with httpx.AsyncClient() as client:
        resp = await client.get(
            f"https://crm.internal.example.com/api/customers/{customer_id}/orders",
            params={"status": status},
            headers={"Authorization": f"Bearer {get_service_token()}"}
        )
        resp.raise_for_status()
        return resp.json()

def get_service_token() -> str:
    import os
    return os.environ["CRM_SERVICE_TOKEN"]

if __name__ == "__main__":
    mcp.run(transport="sse", host="0.0.0.0", port=8080)
```

### Step 2 — Write the Dockerfile

```dockerfile
FROM registry.redhat.io/ubi9/python-311-minimal:latest

# Create non-root user
RUN useradd -m -u 1001 mcpuser

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY --chown=mcpuser:mcpuser server.py .

USER 1001

EXPOSE 8080

CMD ["python", "server.py"]
```

`requirements.txt`:
```
mcp[server]>=1.0.0
httpx>=0.27.0
```

### Step 3 — Build and Push

```bash
# Build
docker build -t registry.example.com/myorg/customer-mcp:latest .

# Push
docker push registry.example.com/myorg/customer-mcp:latest
```

### Step 4 — Deploy to OpenShift or Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: customer-mcp
  namespace: orchestrate-tools
spec:
  replicas: 2
  selector:
    matchLabels:
      app: customer-mcp
  template:
    metadata:
      labels:
        app: customer-mcp
    spec:
      containers:
      - name: customer-mcp
        image: registry.example.com/myorg/customer-mcp:latest
        ports:
        - containerPort: 8080
        env:
        - name: CRM_SERVICE_TOKEN
          valueFrom:
            secretKeyRef:
              name: crm-service-credentials
              key: token
        securityContext:
          runAsNonRoot: true
          runAsUser: 1001
---
apiVersion: v1
kind: Service
metadata:
  name: customer-mcp
  namespace: orchestrate-tools
spec:
  selector:
    app: customer-mcp
  ports:
  - port: 443
    targetPort: 8080
```

```bash
kubectl apply -f customer-mcp-deployment.yaml
```

### Step 5 — Register in Orchestrate

1. Navigate to **Tool Studio → New Tool → MCP Server**
2. Enter the SSE endpoint URL: `https://customer-mcp.orchestrate-tools.svc.cluster.local/sse`
   (or the external URL if deploying outside the cluster)
3. Click **Connect** — Orchestrate auto-discovers `get_customer_details` and `list_customer_orders`
4. Review the discovered tool schemas and click **Save**

### Step 6 — Attach to Agent

In Agent Builder, open the target agent → **Tools → Add Tool** → select the Customer Lookup MCP server. Save the agent.

## Verification

Ask the agent: "Look up customer CUST-001234"

In the agent trace, confirm:
1. An MCP tool span appears for `get_customer_details`
2. The tool returns the expected customer JSON
3. The agent incorporates the data into its response

## See Also

- [Tool Types](../reference/tool-types.md#mcp-servers) — MCP server reference, transport options
- [HOWTO: Add a Tool](../howto/add-tool.md) — Attaching MCP tools to agents
- [Lab 2: Tools Deep Dive](../labs/lab-02-tools-deep-dive.md) — Hands-on MCP tool lab
