# Lab 2: Tools Deep Dive

**Objective:** Extend the Acme HR Assistant with three tool types — an additional OpenAPI tool, a custom Python tool, and an MCP server — learning the key differences in how each is built, tested, and used.

**Estimated Time:** 60 minutes

**Prerequisites:**

!!! note "Before you start"
    - Completed [Lab 1](lab-01-first-agent.md) — the `Acme HR Assistant` agent must exist
    - Python 3.11+ installed locally
    - Docker or Podman installed (for the MCP server section)

---

## Part 1 — Add an OpenAPI Tool (HR Policy API)

For this lab we will use a mock HR API. In a real deployment this would be your ServiceNow, Workday, or custom HR system API.

### Step 1.1 — Register the Mock HR API

Save this as `hr-api.yaml`:

```yaml
openapi: 3.0.0
info:
  title: Acme HR Policy API
  version: "1.0"
servers:
  - url: https://httpbin.org
paths:
  /get:
    get:
      operationId: get_hr_policy
      summary: Retrieve an HR policy document by policy code
      parameters:
        - name: policy_code
          in: query
          required: true
          description: The HR policy code to look up (e.g., PTO-001, REMOTE-002, EXPENSE-003)
          schema:
            type: string
      responses:
        "200":
          description: Policy document returned
```

!!! note "Why httpbin.org?"
    We are using httpbin.org as a mock server — it echoes back your request parameters. This lets you see the tool call working end-to-end without needing a real HR API.

Upload this to **Tool Studio → New Tool → OpenAPI**, then attach it to the `Acme HR Assistant`.

---

## Part 2 — Add a Python Tool (PTO Calculator)

### Step 2.1 — Write the Python Tool

Create a file called `pto_calculator.py`:

```python
from datetime import date, timedelta

async def main(
    start_date: str,
    end_date: str,
    exclude_weekends: bool = True
) -> dict:
    """Calculate the number of PTO days between two dates.
    
    Args:
        start_date: The start date of the PTO period in YYYY-MM-DD format
        end_date: The end date of the PTO period in YYYY-MM-DD format
        exclude_weekends: Set to true to exclude Saturdays and Sundays from the count (default: true)
    
    Returns:
        A dictionary containing the total days and a breakdown
    """
    start = date.fromisoformat(start_date)
    end = date.fromisoformat(end_date)
    
    if end < start:
        return {"error": "end_date must be on or after start_date"}
    
    total_days = 0
    weekend_days = 0
    current = start
    
    while current <= end:
        if current.weekday() >= 5:  # Saturday=5, Sunday=6
            weekend_days += 1
        else:
            total_days += 1
        current += timedelta(days=1)
    
    if not exclude_weekends:
        total_days += weekend_days
    
    return {
        "pto_days_requested": total_days,
        "total_calendar_days": (end - start).days + 1,
        "weekend_days_excluded": weekend_days if exclude_weekends else 0,
        "start_date": start_date,
        "end_date": end_date
    }
```

### Step 2.2 — Test Locally

Before uploading, test your function locally:

```bash
python3 -c "
import asyncio
from pto_calculator import main
result = asyncio.run(main('2025-07-14', '2025-07-18'))
print(result)
"
```

Expected output:
```
{'pto_days_requested': 5, 'total_calendar_days': 5, 'weekend_days_excluded': 0, 'start_date': '2025-07-14', 'end_date': '2025-07-18'}
```

### Step 2.3 — Upload to Tool Studio

Navigate to **Tool Studio → New Tool → Python**. Upload `pto_calculator.py`. Review the auto-generated schema — confirm the three parameters appear with their descriptions.

!!! warning "No direct testing in Tool Studio"
    Python tools cannot be tested standalone. Attach to the agent and test via conversation.

Attach the tool to the `Acme HR Assistant`.

---

## Part 3 — Add an MCP Server (Company Directory)

### Step 3.1 — Create the MCP Server

Create `directory_server.py`:

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Acme Directory Service")

# Simulated company directory data
EMPLOYEES = {
    "EMP-001": {"name": "Alice Chen", "title": "Engineering Manager", "email": "alice.chen@acme.com", "department": "Engineering"},
    "EMP-002": {"name": "Bob Kumar", "title": "HR Business Partner", "email": "bob.kumar@acme.com", "department": "Human Resources"},
    "EMP-003": {"name": "Carol Davis", "title": "Senior Engineer", "email": "carol.davis@acme.com", "department": "Engineering"},
}

@mcp.tool()
async def lookup_employee(employee_id: str) -> dict:
    """Look up an employee's contact details and role.
    
    Args:
        employee_id: The employee ID in format EMP-XXX
    """
    employee = EMPLOYEES.get(employee_id)
    if not employee:
        return {"error": f"Employee {employee_id} not found"}
    return employee

@mcp.tool()
async def find_employees_by_department(department: str) -> list:
    """Find all employees in a specific department.
    
    Args:
        department: The department name to search (e.g., Engineering, Human Resources, Finance)
    """
    results = [
        {"id": eid, **emp}
        for eid, emp in EMPLOYEES.items()
        if emp["department"].lower() == department.lower()
    ]
    return results

if __name__ == "__main__":
    mcp.run(transport="sse", host="127.0.0.1", port=8080)
```

### Step 3.2 — Install Dependencies and Run

```bash
pip install "mcp[server]"
python directory_server.py
```

The server starts on `http://127.0.0.1:8080/sse`.

### Step 3.3 — Register in Tool Studio

Navigate to **Tool Studio → New Tool → MCP Server**. Enter:
- **Endpoint URL:** `http://127.0.0.1:8080/sse`

Click **Connect**. The tools `lookup_employee` and `find_employees_by_department` should be auto-discovered. Save.

Attach the MCP server to the `Acme HR Assistant`.

!!! note "On-Prem CPD / Remote Deployment"
    For a non-local deployment, containerise the server (see [Cookbook: Python MCP Tool](../cookbook/mcp-tool-python.md)) and use an HTTPS endpoint instead of localhost.

---

## Part 4 — Test All Three Tools

Try these prompts in the Agent Builder chat preview:

1. "How many PTO days is July 14th to July 18th?"
2. "Look up employee EMP-001"
3. "Who works in the Engineering department?"
4. "I want to take off from August 4th to August 15th — how many working days is that, and can you look up my manager Alice Chen?"

---

## Validation Checkpoints

- [ ] Python tool correctly calculates 5 PTO days for July 14–18 (Monday–Friday)
- [ ] MCP tool returns Alice Chen's details for EMP-001
- [ ] Department search returns 2 Engineering employees (Alice and Carol)
- [ ] Traces show three distinct tool types (OpenAPI, Python, MCP) in the waterfall view

---

## What You Learned

- How to register and test OpenAPI, Python, and MCP tools
- The difference in testing approach: Python tools require agent invocation; MCP and OpenAPI tools have direct testing support
- How the LLM uses `description` fields to select the right tool and bind the right arguments

---

**Next Lab:** [Lab 3 — Connections & Authentication](lab-03-connections-auth.md)
