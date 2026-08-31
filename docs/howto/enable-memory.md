# Enable and Manage Agentic Memory

Turn on agentic memory for an agent so it remembers user preferences and context across sessions, and manage memory records via the UI and API.

## Prerequisites

!!! note "Prerequisites"
    - Agent already created
    - User has enabled the memory opt-in toggle in their profile (users control their own memory)
    - Builder or Admin role to enable memory on the agent

## Steps

### Step 1 — Enable Memory on the Agent

In Agent Builder, navigate to the agent's **Settings → Memory** panel. Toggle **Enable agentic memory** to on.

This activates memory capture for all users who have opted in on their profile. Users who have not opted in remain unaffected.

### Step 2 — Test Persistence Across Sessions

**Session 1:**

Start a conversation with the agent and state a preference:

> "I always want cost reports in USD, not local currency."

End the session (close the browser tab or explicitly end the conversation).

**Session 2 (new browser tab or incognito window):**

Start a fresh conversation:

> "Generate a cost summary for last quarter."

The agent should use USD currency without being told again, drawing on the stored memory.

### Step 3 — View Stored Memories in the UI

Navigate to your **Profile → Memory** panel. A list of all stored facts appears, with creation timestamps and sensitivity classifications.

### Step 4 — Delete a Specific Memory via API

To remove a single memory record:

```bash
curl -X DELETE https://api.orchestrate.cloud.ibm.com/v1/memory/mem-abc123 \
  -H "Authorization: Bearer $TOKEN"
```

### Step 5 — List and Manage Memories via Python SDK

```python
from ibm_watsonx_orchestrate import MemoryClient

client = MemoryClient(
    api_key="<your-api-key>",
    base_url="https://api.orchestrate.cloud.ibm.com"
)

# List all memories for the current user
memories = client.memory.list()
for m in memories:
    print(f"{m.id}: {m.content} (created: {m.created_at})")

# Delete a specific memory
client.memory.delete(memory_id="mem-abc123")

# Delete all memories
for m in memories:
    client.memory.delete(memory_id=m.id)
```

Install the SDK: `pip install ibm-watsonx-orchestrate`

## Verification

1. After Session 2, navigate to **Profile → Memory** — the preference from Session 1 should appear as a stored record
2. After deleting a memory via API, refresh the Memory panel — the record should be gone
3. Start a new session — the deleted preference should no longer be recalled

!!! warning "30-day retention"
    Memory records expire after 30 days of inactivity. A memory is considered active if it was retrieved or updated within the 30-day window. This retention period is not configurable.

## See Also

- [Memory](../reference/memory.md) — Architecture, vector stores, SDK, and customisation gaps
- [Lab 4: Agentic Memory](../labs/lab-04-agentic-memory.md) — Hands-on lab exercising memory persistence
