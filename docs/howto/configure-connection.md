# Configure a Connection

Set up a connection to authenticate agent tool calls against an external enterprise system.

## Prerequisites

!!! note "Prerequisites"
    - Admin or Builder role in watsonx Orchestrate
    - For OAuth 2: client ID, client secret, and token endpoint URL from the target system's IdP
    - For API Key: the API key value and the header name expected by the target API

---

=== "OAuth 2 Client Credential"

    Use this when the agent calls a back-end API using a shared service account (machine-to-machine auth).

    ### Step 1 — Create the Connection Object

    Navigate to **Connections Manager → New Connection**. Enter a descriptive name (e.g., `salesforce-service-account-live`).

    ### Step 2 — Select Auth Type

    Choose **OAuth 2 — Client Credential** from the authentication type dropdown.

    ### Step 3 — Configure Endpoints and Parameters

    | Field | Value |
    |-------|-------|
    | Token endpoint URL | `https://login.salesforce.com/services/oauth2/token` (example) |
    | Client ID | Your app's client ID |
    | Client secret | Your app's client secret |
    | Scope | Space-separated scopes (e.g., `api refresh_token`) |
    | Audience (optional) | Required for some IdPs (e.g., Workday) |

    ### Step 4 — Set Credential Model

    Select **Team** (the connection uses one shared service account credential for all users).

    ### Step 5 — Validate

    Click **Connect**. For OAuth 2 variants, the platform validates credentials in real-time by attempting a token exchange. An invalid client secret or wrong token endpoint URL will fail here immediately.

    !!! note "Credentials are encrypted immediately"
        Once saved, the client secret is encrypted with AES-256 and cannot be retrieved in plaintext. Only the last 4 characters are shown for identification.

    ### Step 6 — Promote to Live

    If creating for production, switch the environment to **Live** before binding to any production tool.

    !!! tip "Environment alignment"
        Tools in Draft must use Draft connections. Tools in Live must use Live connections. Always promote both together.

=== "API Key"

    Use this when the target API authenticates via a static key in a request header or query parameter.

    ### Step 1 — Create the Connection Object

    **Connections Manager → New Connection**. Enter a descriptive name.

    ### Step 2 — Select Auth Type

    Choose **API Key** from the dropdown.

    ### Step 3 — Configure the Key

    | Field | Value |
    |-------|-------|
    | Header name | The header the API expects, e.g., `X-API-Key` or `Authorization` |
    | Key value | The API key value |

    ### Step 4 — Set Credential Model

    Choose **Team** for a shared service key, or **Member** if each user has their own key.

    !!! warning "Validation at execution time only"
        API Key connections are not validated when saved — only when a tool actually calls the API. A misconfigured key will not surface until the first agent interaction that triggers the tool.

    ### Step 5 — Bind to a Tool

    Open the target tool in Tool Studio. In the **Authentication** section, select this connection from the dropdown.

---

## Verification

1. Invoke the agent with a message that triggers the connected tool
2. Open the agent trace — find the tool span
3. Confirm the tool span shows HTTP 200 and a valid response payload (not 401/403)

## See Also

- [Connections](../reference/connections.md) — Full auth type reference, connection lifecycle, and token refresh
- [Security](../reference/security.md) — OBO flows for user-delegated access patterns
- [Cookbook: OBO Flow with Salesforce](../cookbook/obo-salesforce.md) — On-Behalf-Of recipe for Salesforce
