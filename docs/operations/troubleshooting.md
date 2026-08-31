# Troubleshooting

This page covers the primary diagnostic tools available for watsonx Orchestrate, along with common failure patterns and their remediation steps.

---

## Health Check Script

IBM provides an open-source health check script for on-premises CPD deployments. It validates the status of every Orchestrate component and produces a structured pass/fail report.

### Running the Health Check

```bash
# Download the latest release
curl -O https://github.com/IBM/wxo-health-check/releases/latest/download/wxo-health-check.sh
chmod +x wxo-health-check.sh

# Run against your CPD namespace
./wxo-health-check.sh --namespace cpd

# Run with verbose output for debugging
./wxo-health-check.sh --namespace cpd --verbose

# Run and write output to a file
./wxo-health-check.sh --namespace cpd > health-report.txt
```

### Interpreting Results

```
✓ All Orchestrate pods running           → All pods in Running state
✓ PostgreSQL cluster healthy             → Primary + replicas reachable
✗ ClickHouse cluster unhealthy           → One or more ClickHouse pods not ready
✓ Elasticsearch cluster healthy          → All shards allocated, cluster green
✓ Tool Runtime Manager ready             → TRM responding to health probes
✓ Agent Runtime ready                    → AR serving inference requests
✗ Observability stack: exporter failing  → Spans not being batched/stored
```

Any `✗` (fail) result indicates a degraded or broken component. Use the verbose output to see the specific pod status and recent logs.

---

## Orchestrate MCP Diagnostic Server

For advanced diagnostics, IBM provides an MCP server that exposes Orchestrate operational data as tools accessible by an AI assistant (including the Bob assistant in your development environment).

The MCP server enables natural language diagnostics:

- "Show me all pods with restart count > 5 in the cpd namespace"
- "What errors appeared in the Tool Runtime Manager logs in the last hour?"
- "Check the ClickHouse cluster status and report any issues"

See the [IBM Orchestrate Health Check GitHub repository](https://github.com/IBM/wxo-health-check) for MCP server setup instructions.

---

## Common Failure Patterns

### Agent Returns Error: "Tool execution failed"

**Symptoms:** Agent responds with an error message; no tool output is returned to the user.

**Diagnostic steps:**
1. Open the agent trace in the Observability UI (Traces tab)
2. Find the failing tool span — it will show a red error indicator
3. Expand the span to see the HTTP status code and response body from the external API

**Common causes and fixes:**

| Cause | Fix |
|-------|-----|
| HTTP 401/403 from target API | Check the Connection — the API key or OAuth token may have expired; rotate credentials |
| HTTP 404 | Verify the OpenAPI spec operationId and path match the actual API endpoint |
| HTTP 429 (rate limited) | The target API is rate-limiting Orchestrate; implement retry logic or request a quota increase |
| Tool timeout (>30 seconds for Python tools) | Optimise the Python tool script; break long operations into smaller async steps |
| Schema validation error | The API returned an undeclared field; update the OpenAPI spec to match the actual response schema |

### Agent Not Responding / Session Hangs

**Symptoms:** The user sends a message and receives no response; the chat interface shows a loading indicator indefinitely.

**Diagnostic steps:**
1. Check Agent Runtime pod logs: `oc logs -l app=agent-runtime -n cpd --tail=100`
2. Check if the LLM gateway is reachable: `oc exec -it <agent-runtime-pod> -- curl -k https://llm-gateway/health`

**Common causes and fixes:**

| Cause | Fix |
|-------|-----|
| LLM gateway unreachable | Verify AI Gateway / IFM pods are running; check network policies |
| Context window exhausted | Enable context compaction; review if tool outputs are excessively large |
| Model rate limit | The LLM provider is throttling requests; check quota in IBM Cloud or upgrade tier |
| Agent stuck in infinite tool loop | Add explicit termination conditions to system instructions; set `max_tool_calls` limit |

### ClickHouse / Observability Not Working

**Symptoms:** Traces are not appearing in the Observability UI; the Control Plane shows no data.

**Diagnostic steps:**
```bash
# Check ClickHouse pod status
oc get pods -l app=clickhouse -n cpd

# Check Observability Exporter logs
oc logs -l app=orchestrate-obs-exporter -n cpd --tail=50

# Check if spans are being queued (look for backpressure warnings)
oc logs -l app=agent-runtime -n cpd | grep -i "span\|exporter\|trace"
```

**Common causes and fixes:**

| Cause | Fix |
|-------|-----|
| ClickHouse pod not running | Check PVC binding — ClickHouse requires block storage; verify storage class |
| Exporter backpressure | ClickHouse is overloaded; scale the ClickHouse StatefulSet or increase resource limits |
| Network policy blocking exporter → ClickHouse | Review NetworkPolicy objects in the `cpd` namespace |

### Tool Runtime Manager Pods CrashLoopBackOff

**Symptoms:** `oc get pods` shows TRM pods in `CrashLoopBackOff` state.

**Diagnostic steps:**
```bash
# Get the crash reason
oc describe pod <trm-pod-name> -n cpd
oc logs <trm-pod-name> -n cpd --previous
```

**Common causes and fixes:**

| Cause | Fix |
|-------|-----|
| Insufficient memory limits | Increase TRM pod memory limits in the Orchestrate custom resource |
| Python dependency build failure | Check if a custom Python tool has an incompatible dependency; review TRM init logs |
| File storage not mounted | Verify file storage PVC is bound and accessible |

### Authentication Errors Across All Tools

**Symptoms:** Every tool call fails with HTTP 401; all connections appear broken.

**Common causes and fixes:**

| Cause | Fix |
|-------|-----|
| IBM Cloud IAM token expired (SaaS) | Rotate the IAM API key used by the Connections Manager service account |
| CPD admin token expired | Re-authenticate the CPD service account used by the Connections Manager |
| Enterprise IdP certificate expired | Renew the SAML signing certificate in the enterprise IdP and update Orchestrate's IdP configuration |

---

## Log Locations (On-Premises CPD)

| Component | How to Access Logs |
|-----------|-------------------|
| Agent Runtime | `oc logs -l app=agent-runtime -n cpd` |
| Tool Runtime Manager | `oc logs -l app=tool-runtime-manager -n cpd` |
| Flow Runtime | `oc logs -l app=flow-runtime -n cpd` |
| Observability Exporter | `oc logs -l app=orchestrate-obs-exporter -n cpd` |
| ClickHouse | `oc logs -l app=clickhouse -n cpd` |
| PostgreSQL | `oc logs -l app=postgresql -n cpd` |

For SaaS deployments, logs are accessible through the IBM Cloud Logs dashboard or via the Orchestrate admin console.

---

## Related References

- [Architecture](architecture.md) — Understanding which component is responsible for what
- [Installation](installation.md) — Post-installation health check steps
- [Observability](../reference/observability.md) — Using the trace UI to diagnose agent behaviour
- [Control Plane](../reference/control-plane.md) — Fleet-level error rate monitoring and the Control Plane AI assistant
