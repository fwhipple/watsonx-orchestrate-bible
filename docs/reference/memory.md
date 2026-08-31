# Agentic Memory

Agentic memory gives watsonx Orchestrate agents the ability to remember information about a user across separate sessions and conversation threads. Without memory, every conversation starts from zero — the agent has no knowledge of prior interactions, stated preferences, or previously resolved context. Memory eliminates this repetition and enables agents to provide genuinely personalised, continuity-aware experiences.

---

## How It Works

When memory is enabled for an agent, the platform captures salient facts and preferences from conversations (e.g., "User prefers metric units", "User's default cost centre is CC-4412") and stores them as structured memory records in the tenant's vector database. On subsequent turns — even in entirely new sessions — the agent retrieves relevant memories via semantic search and injects them into the context window before responding.

Memory is **per-user and per-tenant**: a user's memories are scoped to their identity within a specific tenant. There is no cross-tenant memory leakage.

---

## Key Features

### User Opt-In Toggle

Memory is subject to user consent. Each user can enable or disable memory capture via the UI toggle in their profile settings. Agents can only retain memories for users who have opted in.

### Memory API and UI

Memories can be managed programmatically or through the UI:

| Operation | API Endpoint | UI |
|-----------|-------------|-----|
| List all memories | `GET /v1/memory` | Memory panel in user settings |
| Retrieve a memory | `GET /v1/memory/{id}` | Individual memory detail view |
| Update a memory | `PUT /v1/memory/{id}` | Inline edit in memory panel |
| Delete a memory | `DELETE /v1/memory/{id}` | Delete button in memory panel |
| Delete all memories | `DELETE /v1/memory` | "Clear all" in memory panel |

### Sensitive Data Classification

The platform automatically classifies captured memories against a sensitive data taxonomy. Memories containing PII categories (phone numbers, financial account details, health information) are flagged and handled with additional access controls. Sensitive memories can be configured to be suppressed from context injection or redacted in UI views.

---

## Memory Architecture

Memory storage is backed by the same vector database used for knowledge search, scoped to a dedicated memory index per tenant.

| Deployment | Vector Store | Notes |
|------------|-------------|-------|
| SaaS — AWS | Amazon OpenSearch Service (managed) | Shared with knowledge index, tenant-isolated namespace |
| SaaS — IBM Cloud | Elasticsearch (managed) | Shared with knowledge index, tenant-isolated namespace |
| On-Prem CPD | Elasticsearch (containerised on OpenShift) | Bound to same PVC storage class as knowledge index |

---

## Data Retention

!!! warning "30-day rolling retention"
    Memory records are automatically purged after **30 days of inactivity**. A memory is considered active if it has been retrieved or updated within the retention window. There is currently no configuration option to extend or reduce this window — changes require engineering engagement.

What gets stored per memory record:

- The captured fact or preference (text)
- The user ID (scoped to tenant)
- Timestamp of creation and last access
- Sensitivity classification label
- Source conversation ID (for auditability)

---

## IBM Watson Orchestrate Python SDK

External agents and custom integrations can interact with the memory system programmatically using the **IBM Watson Orchestrate Python SDK**.

```python
from ibm_watsonx_orchestrate import MemoryClient

client = MemoryClient(
    api_key="<your-api-key>",
    base_url="https://api.orchestrate.cloud.ibm.com"
)

# List all memories for the current user
memories = client.memory.list()
for m in memories:
    print(m.id, m.content)

# Delete a specific memory
client.memory.delete(memory_id="mem-abc123")
```

The SDK is compatible with Python 3.11+ and can be installed via:

```bash
pip install ibm-watsonx-orchestrate
```

---

## Customisation and Limitations

| Capability | Current Status | Notes |
|-----------|---------------|-------|
| Retention period | Not configurable | Fixed at 30 days; changes require engineering engagement |
| Memory threshold tuning | Not configurable via UI/API | Controls what gets captured; requires engineering involvement |
| Sensitive data taxonomy | Platform-defined | Custom categories not currently supported |
| Cross-agent memory sharing | Not supported | Memory is agent-scoped per user |
| SDK preview (threshold tuning) | On roadmap | Future release will expose tuning parameters via SDK |

!!! note "On-Prem CPD"
    Memory is fully supported on Cloud Pak for Data deployments. The vector store is the same Elasticsearch instance used for knowledge search, partitioned by tenant namespace.

---

## Related References

- [Security](security.md) — User data protection, encryption at rest, and tenant isolation
- [Connections](connections.md) — Credential management for tool calls triggered from memory-aware agents
- [HOWTO: Enable Memory](../howto/enable-memory.md) — Step-by-step guide to enabling and managing agentic memory
- [Lab 4: Agentic Memory](../labs/lab-04-agentic-memory.md) — Hands-on lab exercising memory persistence and API management
