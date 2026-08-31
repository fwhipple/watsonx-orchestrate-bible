# Cookbook

The Cookbook provides self-contained recipes for common integration patterns. Each recipe can be followed independently and includes complete code examples.

---

## Authentication & Security

| Recipe | Description |
|--------|-------------|
| [Slack + Okta + Workday SSO](slack-sso-workday.md) | Configure Slack channel with Okta SAML federation and two-step OBO token exchange to Workday |
| [OBO Flow with Salesforce](obo-salesforce.md) | Set up a two-step On-Behalf-Of flow through Entra ID to Salesforce |

## Workflows

| Recipe | Description |
|--------|-------------|
| [Approval Workflow](agentic-workflow-approval.md) | Build a human-in-the-loop approval workflow with async callback |

## Observability

| Recipe | Description |
|--------|-------------|
| [Custom Observability](custom-observability.md) | Push agent traces to Datadog or New Relic via OTLP |

## Performance Testing

| Recipe | Description |
|--------|-------------|
| [Load Test with K6](load-test-k6.md) | Complete K6 script for multi-turn agent load testing against the SaaS service |

## External Integrations

| Recipe | Description |
|--------|-------------|
| [Python MCP Tool](mcp-tool-python.md) | Build, containerise, and register a Python MCP server |
| [LangGraph External Agent](langgraph-external-agent.md) | Register a LangGraph agent as a collaborator and pipe traces to the Control Plane |
