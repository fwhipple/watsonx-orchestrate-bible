# Platform Roadmap

Upcoming features and infrastructure changes for the watsonx Orchestrate platform — runtime, infrastructure, memory, evaluation, and the Control Plane.

!!! warning "Roadmap accuracy"
    Items reflect information from the IBM watsonx Orchestrate Deep Dive Enablement sessions. Verify current status with your IBM account team or official release notes.

---

## Runtime & Infrastructure

| Feature | Timeframe | Details |
|---------|-----------|---------|
| **Persistent Threads** | August end | Conversation threads that persist across sessions, enabling long-running multi-session use cases |
| **Tool shortlisting** | September | Limit which tools are exposed to the LLM on each turn, reducing context bloat and improving routing accuracy |
| **Go Runtime for voice** | September | Lower-latency execution runtime; strongly recommended for voice deployments |
| **OpenShift AI replacing bundled IFM** | Upcoming releases | Legacy Internal Foundation Model (IFM) serving stack replaced by Red Hat OpenShift AI (KServe, vLLM, Caikit) on CPD deployments |

## Memory

| Feature | Timeframe | Details |
|---------|-----------|---------|
| **Memory in V2 Chat Completions** | August end | Full memory support available via the V2 Chat Completions API endpoint |
| **SDK preview for threshold tuning** | Future | Expose memory similarity threshold as a per-tenant configurable parameter via the Python SDK (currently requires engineering engagement) |

## Agent Ops (Evaluation & Optimization)

| Feature | Timeframe | Details |
|---------|-----------|---------|
| **Agent Ops GA** | August 2026 | Full General Availability of evaluation (4 core metrics, LLM-as-judge) and optimization (JEPA, ACE) |
| **Control Plane + Agent Ops merge** | Future | The Control Plane AI assistant and Agent Ops evaluation agent merged into a single unified diagnostic + optimization assistant |

## Control Plane

| Feature | Timeframe | Details |
|---------|-----------|---------|
| **Control Plane Dashboard GA** | August 2026 (hotfix) | Full 7-tab dashboard: Overview, Adoption, Analytics, FinOps, Quality, Reliability, Security |
| **Workspace filtering** | September | Builders see only their workspace agents; admins see the full fleet |
| **Dashboard customisation (user-level MVP)** | September | Users can customise their dashboard layout |
| **Draft/Live toggle** | Future | Filter Control Plane metrics by deployment stage (Draft vs Live) |
| **Control Plane scheduling** | Future | Schedule recurring reports and health summaries |
| **Control Plane callback support** | Future | Asynchronous processing for long-running analyses in the AI assistant |
| **Slack channel for Control Plane AI** | Future (testing) | Control Plane AI assistant accessible via Slack |
| **Bedrock Agent Core trace mapping** | Future | Map Amazon Bedrock Agent traces to Orchestrate Control Plane format |

## Security & Connections

| Feature | Timeframe | Details |
|---------|-----------|---------|
| **External vault integration** | Future (roadmap) | HashiCorp Vault, IBM Key Protect, and AWS Secrets Manager integration for externally managed credential secrets |
