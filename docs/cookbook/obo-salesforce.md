# OBO Flow with Entra ID and Salesforce

## Problem

An agent needs to call Salesforce APIs as the authenticated end user — not as a shared service account — in an environment where Salesforce is secured by Microsoft Entra ID (Azure AD). Salesforce does not directly accept the user's Orchestrate session token; it requires an Entra-issued token first.

## Solution

Configure a two-step OBO connection chain: Orchestrate exchanges the user's session token for an Entra access token, then exchanges the Entra token for a Salesforce session token. Both exchanges are handled transparently by the Connections Manager.

## Prerequisites

- Microsoft Entra ID tenant with Orchestrate registered as an app (`urn:ietf:params:oauth:grant-type:jwt-bearer` grant enabled)
- Salesforce Connected App with OAuth JWT Bearer flow enabled and Entra ID as a trusted IdP
- watsonx Orchestrate admin role
- A Salesforce OpenAPI tool registered in Tool Studio

## Steps

### Step 1 — Register Orchestrate in Entra ID

1. In Azure Portal → App Registrations → **New Registration** → name: `watsonx-orchestrate`
2. Under **API Permissions**, add the Salesforce resource with the `api` scope
3. Enable the **On-Behalf-Of** grant:
   - Navigate to **Expose an API → Add a scope** → `api://orchestrate/user_impersonation`
4. Note the **Application (client) ID** and **Directory (tenant) ID**
5. Create a **Client Secret** and note the value

### Step 2 — Create the Entra OBO Connection

1. **Connections Manager → New Connection**
2. Name: `entra-obo`
3. Auth type: **OAuth 2 JWT Bearer**
4. Token endpoint: `https://login.microsoftonline.com/<tenant-id>/oauth2/v2.0/token`
5. Client ID: your Orchestrate Entra app client ID
6. Client secret: the secret from Step 1
7. Scope: `https://salesforce.com/.default` (or your Salesforce resource URI)
8. Credential model: **Member**
9. Click **Save**

### Step 3 — Create the Salesforce OBO Connection

1. **Connections Manager → New Connection**
2. Name: `salesforce-obo-member`
3. Auth type: **OAuth 2 JWT Bearer**
4. Token endpoint: `https://login.salesforce.com/services/oauth2/token`
5. Client ID: your Salesforce Connected App consumer key
6. Configure the **OBO chain**: set `upstream_connection = entra-obo` (the connection from Step 2 provides the assertion token)
7. Credential model: **Member**
8. Click **Save**

### Step 4 — Bind to Salesforce Tools

In Tool Studio, open each Salesforce tool. In **Authentication**, select `salesforce-obo-member`. Save.

### Step 5 — Test the Two-Step Flow

Invoke the agent with a request that triggers a Salesforce tool:

> "Show me my open Salesforce opportunities"

Watch the trace. You should see two consecutive connection spans:
1. `entra-obo`: POST to Entra token endpoint → Entra access token issued
2. `salesforce-obo-member`: POST to Salesforce token endpoint (with Entra token as assertion) → Salesforce session token issued

## Verification

In the agent trace, confirm:
1. Step 1 span: HTTP 200 from `login.microsoftonline.com` with an `access_token` in the response
2. Step 2 span: HTTP 200 from `login.salesforce.com` with a Salesforce `access_token`
3. Salesforce API call span: `access_token` matches step 2; response contains the user's own opportunities (not all opportunities)

!!! tip "Diagnosing 401 errors"
    If step 1 succeeds but step 2 returns 401, verify that the Salesforce Connected App has the Entra app registered as a trusted IdP in Salesforce Setup → Identity Provider.

## See Also

- [Security](../reference/security.md#on-behalf-of-obo-flow) — OBO flow architecture (1-step and 2-step variants)
- [Connections](../reference/connections.md) — Auth type reference and connection lifecycle
- [HOWTO: Configure a Connection](../howto/configure-connection.md) — Step-by-step connection setup
