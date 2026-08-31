# watsonx Orchestrate Bible — Plan

## Top-Level Overview

Transform 11 sessions of IBM watsonx Orchestrate Deep Dive Enablement material into a
polished, authoritative MkDocs site (Material for MkDocs theme) deployed as GitHub Pages.

**Audience:** IBM field engineers and technical sellers who need deep technical detail to
implement and troubleshoot.

**Approach:** Fresh synthesis — no raw transcript feel. All content is written from the
source material but restructured as clean, authoritative reference documentation.

**Excluded scope:** Migration (Session 08 — WXA to WXO) is explicitly out of scope.

**Output:** A single MkDocs site (`mkdocs.yml` + `docs/` tree) structured as **five Books**,
each a top-level navigation section.

---

## Book Structure

| Book | Purpose | Tone |
|------|---------|------|
| **Reference** | Authoritative deep-dive on every concept, component, and API surface including Roadmap | Comprehensive, precise |
| **Operations** | Install, run, observe, test, and troubleshoot in production | Operational, diagnostic |
| **HOWTO** | Goal-oriented task guides — "how do I accomplish X?" | Step-by-step, concise |
| **Cookbook** | Self-contained recipes for common integration patterns | Example-first, practical |
| **Labs** | Progressive self-run lab series (Lab 1 = zero prior knowledge; builds through all major features) | Instructional, hands-on |

> **Roadmap material** from all sessions lives in the Reference book under
> `docs/reference/roadmap/` — a consolidated summary plus area-specific sub-pages.

> **Agentic Workflows** has a Reference page (technical spec) AND a HOWTO page
> (step-by-step build guide). Reference = "what is it", HOWTO = "how do I build it".

> **Labs** target SaaS primarily. On-Prem differences are surfaced in callout boxes
> where they matter. Lab 1 assumes zero prior experience with watsonx Orchestrate.

---

## Sub-Tasks

---

### Sub-Task 1 — Project Scaffold

**Intent:** Create the MkDocs project skeleton so every subsequent sub-task drops content
into the right place with zero rework.

**Expected Outcomes:**
- `mkdocs.yml` with Material theme configured, nav skeleton for all five books,
  plugins enabled (search, tags, git-revision-date-localized)
- `docs/` directory tree matching the nav skeleton (stub `index.md` files for every section)
- `docs/index.md` — landing page / "About this Bible"
- `docs/stylesheets/extra.css` — IBM palette override (blue/gray)
- `requirements.txt` — pinned MkDocs + Material + plugin versions
- `.github/workflows/deploy.yml` — GitHub Actions workflow to build and deploy to
  `gh-pages` branch on every push to `main`
- `README.md` — repo-level instructions for local preview

**Todo List:**
1. Create `requirements.txt` with pinned `mkdocs`, `mkdocs-material`,
   `mkdocs-git-revision-date-localized-plugin`
2. Create `mkdocs.yml` with site metadata, Material theme (dark+light palette, IBM colours),
   full nav skeleton for all five books, plugins configured
3. Create `docs/index.md` — landing/welcome page with book descriptions
4. Create stub `index.md` files for every nav section and sub-section across all five books
5. Create `docs/stylesheets/extra.css` with IBM blue primary colour overrides
6. Create `.github/workflows/deploy.yml` using `actions/checkout`, `pip install -r requirements.txt`,
   `mkdocs gh-deploy --force`
7. Create `README.md` with local-preview instructions (`pip install`, `mkdocs serve`)

**Relevant Context:**
- Material for MkDocs docs: https://squidfunk.github.io/mkdocs-material/
- All content pages in later sub-tasks must match the nav entries created here exactly.

**Status:** [ ] pending

---

### Sub-Task 2 — Reference Book: Core Concepts

**Intent:** Lay the conceptual foundation for the entire site. Every other book links back
to these pages. Covers the ADLC, the building blocks of an agent, and platform editions.

**Expected Outcomes:**
- `docs/reference/index.md` — overview of the Reference book
- `docs/reference/adlc.md` — Agent Development Lifecycle (Plan→Build→Test→Evaluate→Observe→Refine)
- `docs/reference/agent-building-blocks.md` — Instructions, Skills, Knowledge, Tools,
  Collaborators, Connections
- `docs/reference/tool-types.md` — OpenAPI, Python, MCP, Agent Workflows, Toolkits
  (with performance tiers: small/medium/large, worker counts, concurrency limits)
- `docs/reference/platform-editions.md` — SaaS vs On-Prem CPD (versions, release cadence,
  model serving differences, vector store differences)
- `docs/reference/models.md` — supported models, premium tenant capabilities, context compaction

**Source Material:** Session 01 (ADLC, building blocks, tool types), Session 03 (tool
runtime detail, toolkit tiers)

**Todo List:**
1. Write `reference/adlc.md` — lifecycle phases with Mermaid diagram, iteration loop rationale
2. Write `reference/agent-building-blocks.md` — each building block: role, configuration notes,
   how it relates to other blocks
3. Write `reference/tool-types.md` — comparison table of all tool types; toolkit tier table
4. Write `reference/platform-editions.md` — SaaS vs CPD comparison table, release delta,
   Argo CD cadence, vector store options
5. Write `reference/models.md` — model support table, premium tenant note, context compaction behaviour
6. Write `reference/index.md` — intro paragraph + table of contents linking all reference pages

**Status:** [ ] pending

---

### Sub-Task 3 — Reference Book: Memory, Connections & Security

**Intent:** Document the stateful and security layers that underpin every agent deployment.

**Expected Outcomes:**
- `docs/reference/memory.md` — agentic memory (scope, opt-in, UI/API, retention,
  sensitive data classification, SDK, customisation gaps)
- `docs/reference/connections.md` — connections framework (draft/live environments,
  all auth types, team vs member credentials, lifecycle, token refresh, validation behaviour)
- `docs/reference/security.md` — OBO flow, SSO integration, encryption at rest, RBAC

**Source Material:** Session 03 (memory), Session 05 (connections, OBO, SSO)

**Todo List:**
1. Write `reference/memory.md` — memory architecture, user toggle, SDK availability, 30-day
   retention policy, SaaS vs on-prem vector stores, customisation gaps table
2. Write `reference/connections.md` — connection lifecycle (numbered steps), auth types table
   (OAuth 2 Code/Client Cred/Password/JWT, Bearer, API Key, Key-Value), team vs member
   credential model, validation behaviour per auth type
3. Write `reference/security.md` — OBO flow with Mermaid sequence diagram (2-step Entra+Salesforce
   and 1-step direct variants), SSO architecture, data protection notes

**Status:** [ ] pending

---

### Sub-Task 4 — Reference Book: Agentic Workflows

**Intent:** Provide the complete technical specification for the Agentic Workflows engine.
The HOWTO book (Sub-Task 10) covers how to build one; this page covers what it is.

**Expected Outcomes:**
- `docs/reference/agentic-workflows.md` — definition, stateful/deterministic characteristics,
  all node types, scheduling rules, callbacks, digression handling, channel support matrix
- `docs/reference/agent-vs-workflow.md` — decision framework, positioning vs LangFlow/BAW/BAMOE,
  integration patterns

**Source Material:** Session 09 (slides, transcript, use-case guide PDF)

**Todo List:**
1. Write `reference/agentic-workflows.md` — full node type reference (tool call, agent, user
   activity, conditional, loop, wait), scheduling constraints (5-min minimum), channel support
   matrix table, callback REST pattern
2. Write `reference/agent-vs-workflow.md` — side-by-side comparison table, Mermaid decision
   tree, integration pattern guide (atomic tool vs MCP vs workflow wrapper),
   LangFlow/BAW/BAMOE positioning

**Status:** [ ] pending

---

### Sub-Task 5 — Reference Book: Observability & Control Plane

**Intent:** Document the full observability stack and Control Plane dashboard so operators
and builders can monitor, debug, and govern agents.

**Expected Outcomes:**
- `docs/reference/observability.md` — stack architecture, trace context propagation,
  span attributes, retention, OTLP export, all 11 UI features
- `docs/reference/control-plane.md` — all 7 dashboard tabs, Control Plane agent capabilities,
  external agent trace import/export

**Source Material:** Session 07 (observability), Session 10 (control plane)

**Todo List:**
1. Write `reference/observability.md` — Mermaid architecture diagram (trace producers →
   ClickHouse/PostgreSQL → UI), W3C/B3 trace header reference, custom attributes API (~10 KB
   limit), callback span restoration, parallel tool execution span hierarchy, 11 UI features table
2. Write `reference/control-plane.md` — tab-by-tab reference (Overview, Adoption, Analytics,
   FinOps, Quality, Reliability, Security), multi-persona support, workspace filtering status,
   external collector integration (Datadog, New Relic, Instana via OTLP)

**Status:** [ ] pending

---

### Sub-Task 6 — Reference Book: Channels & Voice

**Intent:** Document every supported channel with its feature parity, configuration schema,
and voice-specific settings.

**Expected Outcomes:**
- `docs/reference/channels.md` — channel matrix, SSO support per channel, feature parity
  table, context_access_enabled flag, channel adaptation best practices
- `docs/reference/voice.md` — voice configuration schema, STT/TTS provider tables,
  DTMF/VAD settings, hold messages, call recording architecture,
  SIP vs Genesys comparison, Go runtime note

**Source Material:** Session 11

**Todo List:**
1. Write `reference/channels.md` — channel comparison table (Teams, Slack, WhatsApp, SMS,
   Facebook, Genesis, Embed Chat), Slack+Okta+Workday SSO flow (Mermaid sequence diagram),
   `context_access_enabled` explanation, language/per-agent best practice
2. Write `reference/voice.md` — STT/TTS provider tables (current + roadmap), VAD/DTMF
   definitions, call recording pipeline diagram (Mermaid, customer-owned, no IBM audio storage),
   SIP trunking vs Genesys connector comparison, Go runtime latency improvement note,
   data residency callout

**Status:** [ ] pending

---

### Sub-Task 7 — Reference Book: Evaluation & Optimization

**Intent:** Document the Agent Ops evaluation and optimization framework as an authoritative
reference.

**Expected Outcomes:**
- `docs/reference/evaluation.md` — core metrics, rubric evaluations, test case upload API,
  multi-turn support, GA status
- `docs/reference/optimization.md` — JEPA and ACE algorithms, early stopping, iteration
  limits, Control Plane integration

**Source Material:** Session 04

**Todo List:**
1. Write `reference/evaluation.md` — metric definitions table (Journey Success, Tool Call F1,
   Journey Completion, Routing Accuracy), rubric/LLM-as-judge evaluators, test case structure,
   multi-turn conversation support, GA timeline note
2. Write `reference/optimization.md` — JEPA vs ACE comparison table, Mermaid workflow diagram,
   early stopping behaviour, Control Plane drill-down integration

**Status:** [ ] pending

---

### Sub-Task 8 — Reference Book: Roadmap

**Intent:** Consolidate all roadmap/upcoming-feature information from all sessions into one
place so field engineers can set expectations with customers.

**Expected Outcomes:**
- `docs/reference/roadmap/index.md` — master consolidated table (Feature, Area, Timeframe, Source Session)
- `docs/reference/roadmap/channels-roadmap.md` — channel-specific upcoming features
- `docs/reference/roadmap/voice-roadmap.md` — voice-specific upcoming features
- `docs/reference/roadmap/platform-roadmap.md` — platform/runtime/infrastructure roadmap

**Source Material:** All sessions (roadmap callouts extracted and synthesised)

**Todo List:**
1. Extract all roadmap items from all session summaries
2. Write `reference/roadmap/index.md` — master table sorted by timeframe
3. Write `reference/roadmap/channels-roadmap.md`
4. Write `reference/roadmap/voice-roadmap.md`
5. Write `reference/roadmap/platform-roadmap.md`

**Status:** [ ] pending

---

### Sub-Task 9 — Operations Book

**Intent:** Give operators a complete guide to installing, running, monitoring, and
performance-testing watsonx Orchestrate in production.

**Expected Outcomes:**
- `docs/operations/index.md`
- `docs/operations/installation.md` — on-prem prerequisites, installation modes, air-gapped
  config, health check script
- `docs/operations/architecture.md` — component architecture diagram, SaaS vs on-prem topology
- `docs/operations/load-testing.md` — 4-module framework, scripting best practices,
  test types, key metrics, SaaS vs on-prem differences
- `docs/operations/troubleshooting.md` — health check script usage, MCP diagnostic server,
  common failure patterns

**Source Material:** Session 02 (install, architecture), Session 06 (load/perf testing)

**Todo List:**
1. Write `operations/installation.md` — prerequisites checklist (OpenShift sizing, storage
   types, GPU for IFM), installation modes, private registry/proxy config, installation flow
2. Write `operations/architecture.md` — Mermaid component diagram, component descriptions,
   SaaS vs on-prem topology comparison table
3. Write `operations/load-testing.md` — 4-module framework, K6/Locust scripting patterns
   (session management, think time, checkpoints), test type definitions (load/stress/spike),
   metric glossary (P50/P95/P99, first token latency), SaaS vs on-prem auth/monitoring diff
4. Write `operations/troubleshooting.md` — health check tool usage, MCP server diagnostics,
   common errors and remediation steps
5. Write `operations/index.md` — intro and nav table

**Status:** [ ] pending

---

### Sub-Task 10 — HOWTO Book

**Intent:** Goal-oriented task guides for the most common field engineer workflows. Each
page answers a single "how do I…?" question. HOWTO pages link to Reference for the deep
technical spec.

**Expected Outcomes:**
- `docs/howto/index.md` — task index grouped by persona (builder, operator, admin)
- `docs/howto/create-agent.md`
- `docs/howto/add-tool.md`
- `docs/howto/configure-connection.md`
- `docs/howto/enable-memory.md`
- `docs/howto/build-agentic-workflow.md` ← Agentic Workflows HOWTO (step-by-step build guide)
- `docs/howto/set-up-voice.md`
- `docs/howto/run-evaluation.md`
- `docs/howto/export-traces.md`

**Source Material:** Sessions 01, 03, 05, 09, 11, 04, 07

**Todo List:**
1. Write each HOWTO page as numbered steps with:
   - Prerequisite callout box at the top
   - Code/config snippets where applicable
   - "See Also" links to the relevant Reference pages
2. `howto/build-agentic-workflow.md` — step-by-step workflow builder guide (node addition,
   human-in-the-loop callback setup, scheduling, testing); links to `reference/agentic-workflows.md`
   for the technical spec
3. Write `howto/index.md` — task index grouped by persona

**Status:** [ ] pending

---

### Sub-Task 11 — Cookbook Book

**Intent:** Self-contained recipes for common integration patterns. Each recipe is fully
standalone with code examples.

**Expected Outcomes:**
- `docs/cookbook/index.md` — recipe index grouped by theme
- `docs/cookbook/slack-sso-workday.md` — Slack + Okta SSO + Workday token exchange
- `docs/cookbook/obo-salesforce.md` — OBO flow with Entra ID + Salesforce
- `docs/cookbook/mcp-tool-python.md` — build and register a Python MCP tool
- `docs/cookbook/langgraph-external-agent.md` — connect LangGraph agent as external
  collaborator and pipe traces to Control Plane
- `docs/cookbook/agentic-workflow-approval.md` — approval workflow with human-in-the-loop callback
- `docs/cookbook/load-test-k6.md` — K6 load test script for a multi-turn agent conversation
- `docs/cookbook/custom-observability.md` — push traces to Datadog/New Relic via OTLP
  external collector

**Source Material:** Sessions 05, 09, 06, 07, 10, 11

**Todo List:**
1. Write each recipe using a consistent template:
   `## Problem` → `## Solution` → `## Prerequisites` → `## Steps` → `## Verification` → `## See Also`
2. Include realistic code blocks (Python, YAML, JSON, shell) throughout each recipe
3. Write `cookbook/index.md` — recipe index grouped by theme
   (Authentication & Security, Workflows, Observability, Performance Testing, Channels)

**Status:** [ ] pending

---

### Sub-Task 12 — Labs Book

**Intent:** A progressive, self-run hands-on lab series. Lab 1 assumes zero prior experience
with watsonx Orchestrate (SaaS). Each lab builds on the previous one; by the end a learner
has touched every major feature area. On-Prem differences appear as callout boxes where relevant.

**Lab Series Structure (10–12 labs):**

| Lab | Title | Feature Areas Covered |
|-----|-------|----------------------|
| 1 | Getting Started — Your First Agent | Account setup, Agent Builder UI, first tool, basic conversation |
| 2 | Tools Deep Dive | OpenAPI tool, Python tool, MCP tool — add all three to one agent |
| 3 | Connections & Authentication | OAuth 2 connection, API key connection, team vs member credentials |
| 4 | Agentic Memory | Enable memory, test persistence across sessions, manage memory via API |
| 5 | Agentic Workflows | Build a multi-step workflow with a conditional branch and human approval |
| 6 | Multi-Agent Collaboration | Orchestrator + sub-agent, collaborator binding, routing |
| 7 | Evaluation & Optimization | Upload test cases, run evaluation, interpret metrics, run JEPA/ACE |
| 8 | Observability | View traces, export to external collector, add custom span attributes |
| 9 | Voice & Channels | Configure Slack channel with SSO, add voice config, test phone interaction |
| 10 | Load Testing | Write a K6 script, run a load test, analyse P95 latency and token usage |
| 11 | External Agents & Control Plane | Register a LangGraph external agent, pipe traces to Control Plane, use the Control Plane AI assistant |
| 12 | Capstone — End-to-End Enterprise Use Case | Combine all features: multi-agent workflow with memory, auth, voice channel, evaluation, and observability |

**Expected Outcomes:**
- `docs/labs/index.md` — lab series overview, prerequisites, how to use the labs
- `docs/labs/lab-01-first-agent.md` through `docs/labs/lab-12-capstone.md`
- Each lab page contains: **Objective**, **Prerequisites**, **Estimated Time**,
  **Step-by-step instructions**, **Validation checkpoints**, **On-Prem note callouts**,
  **What you learned**, **Next Lab** link

**Source Material:** All sessions

**Todo List:**
1. Write `labs/index.md` — series overview, cumulative skill map, SaaS account setup prereqs
2. Write Labs 1–4 (foundation: agent, tools, auth, memory)
3. Write Labs 5–8 (intermediate: workflows, multi-agent, eval, observability)
4. Write Labs 9–11 (advanced: voice/channels, load testing, external agents)
5. Write Lab 12 (capstone: end-to-end enterprise use case tying all labs together)

**Status:** [ ] pending

---

### Sub-Task 13 — Final Polish & Nav Wiring

**Intent:** Ensure the site builds cleanly, all nav entries resolve, internal links are
correct, and the site is ready to deploy to GitHub Pages.

**Expected Outcomes:**
- `mkdocs.yml` nav fully wired to every page created in sub-tasks 2–12
- All stub `index.md` pages replaced with real content
- Site builds with `mkdocs build --strict` (zero warnings)
- `docs/index.md` updated with accurate book descriptions and links
- `docs/about.md` — source attribution, session dates, roadmap accuracy disclaimer

**Todo List:**
1. Update `mkdocs.yml` nav to match all final page paths
2. Run `mkdocs build --strict` and fix any broken refs or missing pages
3. Review and polish `docs/index.md` landing page
4. Write `docs/about.md` — source attribution, disclaimer that roadmap content reflects
   session dates and may have changed

**Status:** [ ] pending

---

## Dependency Order

```
Sub-Task 1  (Scaffold)
    │
    ├─► Sub-Tasks 2–9  (Reference + Operations — independent, can run in any order)
    │
    └─► Sub-Tasks 10–11  (HOWTO + Cookbook — should follow Reference sub-tasks)
    │
    └─► Sub-Task 12  (Labs — should follow Reference and HOWTO)
    │
    └─► Sub-Task 13  (Final Polish — runs last)
```
