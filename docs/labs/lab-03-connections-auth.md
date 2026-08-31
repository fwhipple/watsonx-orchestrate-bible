# Lab 3: Connections & Authentication

**Objective:** Secure the Acme HR Assistant's tool calls with proper authentication — configuring an OAuth 2 Client Credential connection and an API Key connection, and observing the difference in validation timing.

**Estimated Time:** 45 minutes

**Prerequisites:**

!!! note "Before you start"
    - Completed [Lab 2](lab-02-tools-deep-dive.md)
    - Access to a test API that accepts API key authentication (we will use HTTPBin for simulation)

---

## Part 1 — Understand Connection Environments

Before creating connections, notice the **environment toggle** in the top-right corner of the Connections Manager: **Draft** and **Live**.

- Work in **Draft** during labs
- Draft connections are isolated from Live — a mistake here won't affect production
- When you promote an agent to Live, you must promote its connections too

---

## Part 2 — Create an API Key Connection

We will secure the HR Policy tool with an API key.

### Step 2.1 — Create the Connection

1. Navigate to **Connections Manager → New Connection**
2. Fill in:
   - **Name:** `hr-policy-api-key-draft`
   - **Auth type:** API Key
   - **Header name:** `X-API-Key`
   - **Key value:** `lab-demo-key-12345`
3. **Credential model:** Team
4. Click **Save**

!!! warning "Validation timing"
    Notice there is no "Test" button for API Key connections. The key is only validated when the tool actually calls the API. This is different from OAuth 2 connections, which validate in real-time.

### Step 2.2 — Bind to the HR Policy Tool

1. Open the `get_hr_policy` tool in Tool Studio
2. Navigate to **Authentication → Connection**
3. Select `hr-policy-api-key-draft`
4. Click **Save Tool**

### Step 2.3 — Test

In the Agent Builder chat, send: "Look up HR policy PTO-001"

Open the trace. In the `get_hr_policy` span, confirm the outbound HTTP request includes the `X-API-Key: lab-demo-key-12345` header.

---

## Part 3 — Create an OAuth 2 Client Credential Connection

### Step 3.1 — Use a Public OAuth 2 Server

For this lab, we will use Auth0's public demo endpoint to practice the OAuth 2 flow without needing a real enterprise IdP.

1. Navigate to **Connections Manager → New Connection**
2. Fill in:
   - **Name:** `demo-oauth2-client-cred-draft`
   - **Auth type:** OAuth 2 Client Credential
   - **Token endpoint:** `https://dev-xxxxxxxx.us.auth0.com/oauth/token`

    !!! note "Use httpbin as a substitute"
        Since we don't have a real Auth0 tenant, enter `https://httpbin.org/post` as the token endpoint. The validation will fail (httpbin echoes but doesn't issue tokens) — this is intentional and illustrates real-time OAuth validation.

   - **Client ID:** `test-client-id`
   - **Client secret:** `test-client-secret`
   - **Scope:** `read:hr`
3. **Credential model:** Team
4. Click **Save**

### Step 3.2 — Observe Real-Time Validation Failure

When you click **Save**, the platform immediately attempts the token exchange. Since the endpoint is httpbin (not a real IdP), it fails.

**This failure is intentional.** It demonstrates that OAuth 2 connections are validated at connection creation time — not just when the tool is first called. This is the key difference from API Key connections.

!!! note "Takeaway"
    OAuth 2 validation catches misconfigurations immediately at setup time. API Key validation only surfaces errors at execution time during a live agent conversation.

### Step 3.3 — Team vs Member Credential Model

Review the two options:

| Model | When to use | Example |
|-------|-------------|---------|
| **Team** | All users share the same service account | Read-only HR policy lookup |
| **Member** | Each user authenticates individually; agent acts as that user | Personal PTO balance query — must see only that user's data |

For the PTO balance scenario, **Member** credentials are required — a service account would expose every employee's data to every other employee.

We will configure Member credentials properly in Lab 9 (with SSO). For now, note the design decision.

---

## Part 4 — Rotate a Credential

Simulate rotating the API key (a common operational task):

1. Open `hr-policy-api-key-draft` in Connections Manager
2. Update the **Key value** to `lab-demo-key-99999`
3. Click **Save**

Because the connection is bound to the tool (not the tool definition itself), the key rotation takes effect immediately for all tools bound to this connection — no tool re-registration needed.

Test again: "Look up HR policy EXPENSE-003"

Confirm the new key value appears in the trace.

---

## Validation Checkpoints

- [ ] API Key connection created successfully with header name `X-API-Key`
- [ ] The HR Policy tool trace shows the API key header in the outbound request
- [ ] OAuth 2 real-time validation failure was observed and understood
- [ ] Key rotation was successful and reflected in the next tool call

---

## What You Learned

- How to create API Key and OAuth 2 Client Credential connections
- The critical difference in validation timing (OAuth 2: at creation; API Key: at execution)
- How Team vs Member credential models differ and when to use each
- How to rotate credentials without re-registering tools

---

**Next Lab:** [Lab 4 — Agentic Memory](lab-04-agentic-memory.md)
