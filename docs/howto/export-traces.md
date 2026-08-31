# Export Traces to an External Observability Platform

Configure the Orchestrate OTLP exporter to forward agent traces to Datadog, New Relic, Instana, or any OTLP-compatible platform.

## Prerequisites

!!! note "Prerequisites"
    - SaaS: Admin Console access (Admin role)
    - On-Prem CPD: access to the cluster `values.yaml` or Helm release
    - An OTLP endpoint URL and API key for the target platform

---

=== "SaaS"

    ### Step 1 — Open Export Settings

    Navigate to **Admin Console → Observability → Export Settings**.

    ### Step 2 — Enter OTLP Endpoint

    Enter the OTLP ingestion URL for your platform:

    | Platform | Endpoint | Protocol |
    |----------|----------|----------|
    | Datadog (US) | `https://trace.agent.datadoghq.com` | http/protobuf |
    | Datadog (EU) | `https://trace.agent.datadoghq.eu` | http/protobuf |
    | New Relic | `https://otlp.nr-data.net:4317` | gRPC |
    | Instana | `https://<unit>.instana.io/api/otlp` | gRPC |

    ### Step 3 — Add Authentication Header

    Enter the authentication header:

    | Platform | Header Name | Header Value |
    |----------|-------------|-------------|
    | Datadog | `DD-API-KEY` | Your Datadog API key |
    | New Relic | `api-key` | Your New Relic license key |
    | Instana | `x-instana-key` | Your Instana agent key |

    ### Step 4 — Select Protocol

    Choose **gRPC** or **http/protobuf** based on the platform requirements (see table above).

    !!! warning "Datadog requires http/protobuf"
        Datadog's OTLP endpoint does not support gRPC from external sources. Always select **http/protobuf** for Datadog.

    ### Step 5 — Save and Validate

    Click **Save**. Orchestrate sends a test span to the configured endpoint. A green checkmark confirms successful delivery.

    ### Step 6 — Trigger a Test Conversation

    Send a message to any deployed agent. Within 60 seconds, traces should appear in your external platform.

=== "On-Premises CPD"

    ### Step 1 — Edit values.yaml

    Add the OTLP export configuration to your Orchestrate Helm values:

    ```yaml
    observability:
      otlp_export:
        enabled: true
        endpoint: "https://otlp.nr-data.net:4317"    # Replace with your endpoint
        headers:
          "api-key": "<new-relic-license-key>"         # Replace with your key
        protocol: grpc                                  # grpc or http/protobuf
        batch_size: 512
        export_interval_ms: 5000
    ```

    ### Step 2 — Apply the Configuration

    ```bash
    # Via Helm
    helm upgrade ibm-watson-orchestrate ibm-orchestrate/watson-orchestrate \
      -f values.yaml \
      -n cpd

    # Via oc apply (if using a custom resource override)
    oc apply -f watsonx-orchestrate-values-patch.yaml
    ```

    ### Step 3 — Restart the Observability Exporter Pod

    ```bash
    oc rollout restart deployment/orchestrate-obs-exporter -n cpd
    oc rollout status deployment/orchestrate-obs-exporter -n cpd
    ```

    ### Step 4 — Verify Exporter Logs

    ```bash
    oc logs -l app=orchestrate-obs-exporter -n cpd | grep -i "export\|error" | tail -20
    ```

    Look for: `successfully exported N spans to <endpoint>` — this confirms export is working.

---

## Verification

1. Trigger an agent conversation (at least 2 turns to generate multiple spans)
2. Wait 60 seconds for the batch export cycle
3. Query your external platform:
   - **Datadog**: APM → Services → search for `watsonx-orchestrate`
   - **New Relic**: Distributed Tracing → search for `service.name = watsonx-orchestrate`
   - **Instana**: Analytics → Traces → filter by `service = watsonx-orchestrate`
4. Confirm the `trace_id` from the Orchestrate trace UI matches the `trace_id` in the external platform

## See Also

- [Observability](../reference/observability.md) — Full OTLP export configuration reference and custom span attributes
- [Cookbook: Custom Observability](../cookbook/custom-observability.md) — Detailed Datadog and New Relic setup recipes
- [Lab 8: Observability](../labs/lab-08-observability.md) — Hands-on lab configuring external trace export
