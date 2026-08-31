# Labs

Welcome to the watsonx Orchestrate Hands-On Lab Series. These 12 labs take you from zero experience to a fully instrumented enterprise AI agent deployment — including voice, evaluation, load testing, and external agent integration.

---

## Lab Series Overview

| Lab | Title | Level | Focus Area |
|-----|-------|-------|-----------|
| [Lab 1](lab-01-first-agent.md) | Your First Agent | Beginner | Account setup, Agent Builder, first tool |
| [Lab 2](lab-02-tools-deep-dive.md) | Tools Deep Dive | Beginner | OpenAPI, Python, and MCP tools |
| [Lab 3](lab-03-connections-auth.md) | Connections & Authentication | Beginner | OAuth 2, API Key, team vs member credentials |
| [Lab 4](lab-04-agentic-memory.md) | Agentic Memory | Intermediate | Memory enable, persistence, API management |
| [Lab 5](lab-05-agentic-workflows.md) | Agentic Workflows | Intermediate | Workflow builder, conditions, human approval |
| [Lab 6](lab-06-multi-agent.md) | Multi-Agent Collaboration | Intermediate | Orchestrator + sub-agent, collaborator routing |
| [Lab 7](lab-07-evaluation.md) | Evaluation & Optimization | Intermediate | Test cases, metrics, JEPA/ACE |
| [Lab 8](lab-08-observability.md) | Observability | Intermediate | Traces, custom attributes, external export |
| [Lab 9](lab-09-voice-channels.md) | Voice & Channels | Advanced | Slack SSO, voice config, STT/TTS |
| [Lab 10](lab-10-load-testing.md) | Load Testing | Advanced | K6 scripts, ramp-up, P95 analysis |
| [Lab 11](lab-11-external-agents.md) | External Agents & Control Plane | Advanced | LangGraph, trace import, Control Plane AI |
| [Lab 12](lab-12-capstone.md) | Capstone | Advanced | End-to-end enterprise use case |
---

## Prerequisites for the Lab Series

Before starting Lab 1, you need:

1. **An IBM watsonx Orchestrate SaaS account** — sign up at [ibm.com/products/watsonx-orchestrate](https://www.ibm.com/products/watsonx-orchestrate)
2. **An IBM Cloud account** — needed for API key generation (free tier is sufficient)
3. **Python 3.11+** installed locally — needed for Labs 3, 7, 10
4. **K6** installed locally — needed for Lab 10: `brew install k6`

!!! note "On-Premises CPD"
    All labs target the SaaS edition. Where on-premises CPD behaviour differs materially, a callout box is included.

---

## Cumulative Skill Map

Each lab builds on the previous one. The agent built in Lab 1 is extended in every subsequent lab:

```
Lab 1: Create agent "Acme HR Assistant"
   └─► Lab 2: Add OpenAPI + Python + MCP tools
       └─► Lab 3: Add OAuth 2 connection to Workday
           └─► Lab 4: Enable memory
               └─► Lab 5: Add approval workflow as a tool
                   └─► Lab 6: Add specialist sub-agents
                       └─► Lab 7: Evaluate and optimize
                           └─► Lab 8: Instrument with custom traces
                               └─► Lab 9: Deploy to Slack + voice
                                   └─► Lab 10: Load test the deployment
                                       └─► Lab 11: Register LangGraph collaborator
                                           └─► Lab 12: Capstone — full enterprise deployment
```
