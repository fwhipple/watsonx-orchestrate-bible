# Reference

The Reference section is the authoritative technical specification for every concept, component, operational parameter, and configuration pattern in IBM watsonx Orchestrate. Designed for field engineers, enterprise architects, and technical sellers, this reference provides low-level architecture insights, runtime behaviours, sizing metrics, and execution invariants across SaaS and on-premises CPD deployments.

---

## Contents

### Core Concepts

Foundational architectural paradigms, operational lifecycles, execution models, and platform topology.

| Page | Description |
|------|-------------|
| [Agent Development Lifecycle (ADLC)](adlc.md) | The 6-stage iterative cycle — Plan, Build, Test, Evaluate, Observe, Refine — for robust agent engineering |
| [Agent Building Blocks](agent-building-blocks.md) | Technical deep-dive into Instructions, Skills, Knowledge, Tools, Collaborators, and Connections |
| [Tool Types](tool-types.md) | Specifications for OpenAPI, Python, MCP Server, Agent Workflow, and Toolkit tools including worker concurrency tiers |
| [Platform Editions](platform-editions.md) | SaaS vs CPD comparison — release cadences, model serving, vector stores, and infrastructure roadmap |
| [Supported Models](models.md) | Model availability matrix, premium tenant entitlements, context compaction, and model selection guidance |

### Stateful Layer & Security

State persistence, identity governance, credential management, and delegated authorisation.

| Page | Description |
|------|-------------|
| [Memory](memory.md) | Agentic memory architecture — per-user/tenant scope, vector stores, 30-day retention, SDK, and customisation gaps |
| [Connections](connections.md) | Centralised credential management — all auth types, team vs member credentials, connection lifecycle, token refresh |
| [Security](security.md) | OBO flows, SSO integration, SAML assertions, credential encryption, and embed chat OAuth |

### Orchestration

Routing engines, deterministic state machines, and multi-agent topologies.

| Page | Description |
|------|-------------|
| [Agentic Workflows](agentic-workflows.md) | Node type reference, scheduling rules, human-in-the-loop callbacks, channel support matrix |
| [Agent vs Workflow](agent-vs-workflow.md) | Architectural decision framework — when to use an agent, when to use a workflow, and integration patterns |

### Observability & Governance

Telemetry, distributed tracing, fleet administration, and FinOps.

| Page | Description |
|------|-------------|
| [Observability](observability.md) | Stack architecture, OpenTelemetry tracing, W3C/B3 headers, custom span attributes, 11 UI features |
| [Control Plane](control-plane.md) | 7-tab dashboard reference, Control Plane AI assistant, external agent trace import/export |

### Channels & Multimodal

Interaction interfaces, SSO federation, voice pipelines, and telephony integrations.

| Page | Description |
|------|-------------|
| [Channels](channels.md) | Channel matrix — Teams, Slack, WhatsApp, SMS, Facebook, Genesis, Embed Chat — with feature parity and SSO support |
| [Voice](voice.md) | Voice configuration schema, STT/TTS providers, DTMF/VAD, call recording architecture, SIP vs Genesys |

### Agent Ops

Systematic evaluation, benchmark datasets, and continuous optimisation.

| Page | Description |
|------|-------------|
| [Evaluation](evaluation.md) | Core metrics, LLM-as-judge evaluators, test case upload API, multi-turn support |
| [Optimization](optimization.md) | JEPA and ACE algorithms, early stopping, iteration limits, Control Plane integration |

### Roadmap

Upcoming features and platform infrastructure evolution, useful for setting customer expectations.

| Page | Description |
|------|-------------|
| [Roadmap Overview](roadmap/index.md) | Consolidated master table of all upcoming features across all areas |
| [Platform Roadmap](roadmap/platform-roadmap.md) | Runtime upgrades, OpenShift AI migration, scaling milestones |
| [Channels Roadmap](roadmap/channels-roadmap.md) | Upcoming channel endpoints and SSO improvements |
| [Voice Roadmap](roadmap/voice-roadmap.md) | Speech provider additions, Go runtime, latency improvements |
