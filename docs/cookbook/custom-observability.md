# Push Agent Traces to an External Observability Platform

## Problem

The operations team wants to see watsonx Orchestrate agent traces alongside infrastructure and application metrics in their existing Datadog or New Relic platform — without switching between tools.

## Solution

Configure the Orchestrate OTLP exporter to forward all agent spans to the external platform. Spans include LLM turns, tool calls, workflow node execution, and knowledge retrieval — fully correlated by trace ID.

## Prerequisites

- Datadog API key or New Relic license key
- SaaS Admin Console access, or On-Prem CPD cluster access (Helm/values.yaml)
- At least one deployed agent generating trace data

## Steps

=== "Datadog"

    ### Step 1 — Get Your Datadog OTLP Endpoint

    | Region | Endpoint | Protocol |
    |--------|----------|----------|
    | US1 | `https://trace.agent.datadoghq.com` | http/protobuf |
    | EU | `https://trace.agent.datadoghq.eu` | http/protobuf |
    | US3 | `https://trace.agent.us3.datadoghq.com` | http/protobuf |

    !!! warning "Use http/protobuf for Datadog"
        Datadog's OTLP endpoint does not accept gRPC from external sources. Always select **http/protobuf**.

    ### Step 2 — Configure in Orchestrate (SaaS)

    1. Navigate to **Admin Console → Observability → Export Settings**
    2. Endpoint: `https://trace.agent.datadoghq.com`
    3. Header name: `DD-API-KEY`
    4. Header value: your Datadog API key
    5. Protocol: **http/protobuf**
    6. Click **Save and Validate** — confirm the test span appears in Datadog APM

    ### Step 2 — Configure in values.yaml (On-Prem CPD)

    ```yaml
    observability:
      otlp_export:
        enabled: true
        endpoint: "https://trace.agent.datadoghq.com"
        headers:
          "DD-API-KEY": "<your-datadog-api-key>"
        protocol: http/protobuf
    ```

    ### Step 3 — Enrich with Custom Tags

    In Python tools, add Datadog-friendly tags as custom span attributes:

    ```python
    from ibm_watsonx_orchestrate.observability import span_logger

    async def main(params: dict, context: dict) -> dict:
        span_logger.set_attribute("dd.service", "hr-agent")
        span_logger.set_attribute("dd.env", "production")
        span_logger.set_attribute("customer.tier", params.get("tier", "standard"))
        # ... your tool logic
    ```

    These attributes appear as facets in Datadog APM, enabling powerful filtering.

    ### Step 4 — View in Datadog APM

    Navigate to **APM → Services**. Within 60 seconds, `watsonx-orchestrate` appears as a service. Click it to see:
    - Service map showing dependencies (agent → tools → external APIs)
    - Latency distribution flame graphs per operation
    - Error traces with full span detail

=== "New Relic"

    ### Step 1 — Get Your New Relic OTLP Endpoint

    | Region | gRPC Endpoint | HTTP Endpoint |
    |--------|--------------|---------------|
    | US | `otlp.nr-data.net:4317` | `otlp.nr-data.net:4318` |
    | EU | `otlp.eu01.nr-data.net:4317` | `otlp.eu01.nr-data.net:4318` |

    ### Step 2 — Configure in Orchestrate (SaaS)

    1. Navigate to **Admin Console → Observability → Export Settings**
    2. Endpoint: `https://otlp.nr-data.net:4317`
    3. Header name: `api-key`
    4. Header value: your New Relic license key (not a user API key — use the **ingest license key**)
    5. Protocol: **gRPC**
    6. Click **Save and Validate**

    ### Step 2 — Configure in values.yaml (On-Prem CPD)

    ```yaml
    observability:
      otlp_export:
        enabled: true
        endpoint: "https://otlp.nr-data.net:4317"
        headers:
          "api-key": "<your-new-relic-license-key>"
        protocol: grpc
        batch_size: 512
        export_interval_ms: 5000
    ```

    Apply and restart the exporter:
    ```bash
    helm upgrade ibm-watson-orchestrate ibm-orchestrate/watson-orchestrate -f values.yaml -n cpd
    oc rollout restart deployment/orchestrate-obs-exporter -n cpd
    ```

    ### Step 3 — Create a Service Map in New Relic

    After traces start flowing:

    1. Navigate to **New Relic One → APM & Services**
    2. Search for `service.name = watsonx-orchestrate`
    3. Click **Service Map** to see the full dependency graph (agent → tools → external APIs)

    ### Step 4 — Set Up Alerts

    In **New Relic → Alerts**:
    ```sql
    SELECT percentile(duration.ms, 95) FROM Span
    WHERE service.name = 'watsonx-orchestrate'
    FACET agent.id
    ```
    Set an alert when P95 exceeds your SLA threshold (e.g., 8000ms).

## Verification

1. Trigger an agent conversation with 2–3 turns (to generate multiple spans)
2. Wait 60 seconds
3. In your external platform, search for `service.name = watsonx-orchestrate`
4. Confirm you can see:
   - Individual LLM turn spans
   - Tool call spans with HTTP status and latency
   - A `trace_id` that matches the trace ID shown in the Orchestrate Observability UI for the same conversation

## See Also

- [Observability](../reference/observability.md#external-otlp-export) — OTLP export configuration reference
- [HOWTO: Export Traces](../howto/export-traces.md) — Step-by-step export configuration guide
- [Lab 8: Observability](../labs/lab-08-observability.md) — Hands-on observability lab
