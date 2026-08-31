# Connections

Connections are watsonx Orchestrate's centralised credential management layer. They provide a secure, auditable mechanism for agents to authenticate against external enterprise systems when executing tools — without ever exposing raw credentials in the agent context window or system prompt.

A connection object encapsulates: the target service's authentication endpoints, the authentication type, the credential values (encrypted at rest), and the delegation model (team vs member).

---

## Draft and Live Environments

Every connection exists in one of two environments:

| Environment | Purpose | When to Use |
|-------------|---------|-------------|
| **Draft** | Development and testing; not exposed to end users | While building and iterating on an agent in the Builder workspace |
| **Live** | Production; available to deployed agents serving real users | After an agent has been tested and promoted to production |

!!! tip "Align connection and agent environments"
    A tool bound to a Draft connection will fail when the agent is promoted to Live. Always promote connections to Live at the same time as the agent.

---

## Credential Model

Connections support two credential delegation models:

### Team Credentials (Functional IDs)

A shared service account credential used by all users interacting with the agent. The agent always authenticates as the same identity, regardless of which end user is in the conversation.

**Use when:** The target system does not need to distinguish between individual users, or when a dedicated integration service account is the appropriate identity (e.g., a read-only data lookup).

### Member Credentials (User-Specific)

Each user authenticates individually — the agent acts on behalf of the specific end user who is conversing. The platform stores and manages per-user tokens, refreshing them automatically.

**Use when:** The target system enforces per-user permissions (e.g., Salesforce records owned by the user, Workday personal HR data), or when audit trails must reflect individual user identity.

!!! note "On-Behalf-Of (OBO) for member credentials"
    For complex enterprise SSO environments, member credentials can be acquired via the OBO token exchange flow. See [Security](security.md) for the full OBO flow specification.

---

## Supported Authentication Types

| Auth Type | How It Works | Validation Timing | Typical Use Case |
|-----------|-------------|------------------|-----------------|
| **OAuth 2 Code** | Authorization code flow; user completes browser-based consent | Real-time during connection | End-user delegated access (Salesforce, Google, Microsoft) |
| **OAuth 2 Client Credential** | Machine-to-machine; client ID + secret exchanged for bearer token | Real-time during connection | Service-to-service integrations, back-end APIs |
| **OAuth 2 Password** | Resource owner password flow; username + password traded for token | Real-time during connection | Legacy systems that don't support code flow |
| **OAuth 2 JWT** | Signed JWT assertion exchanged for bearer token | Real-time during connection | High-assurance enterprise service accounts |
| **Bearer Token** | Static token provided directly; no exchange step | At execution time | Simple API tokens, personal access tokens |
| **API Key** | Key passed in header or query parameter | At execution time | Third-party SaaS APIs with key-based auth |
| **Key-Value** | Custom header name + value pairs | At execution time | Proprietary authentication schemes, internal APIs |

!!! note "Validation timing"
    OAuth 2 variants are validated in real-time when the connection is created — an invalid client ID or mismatched redirect URI will fail immediately. Bearer Token, API Key, and Key-Value connections are only validated at tool execution time; a misconfigured static token will not surface until the first tool call.

---

## Connection Lifecycle

Creating a connection follows a six-step process:

1. **Create the connection object** — Give the connection a name and select the target service. This creates a shell connection record in the platform.

2. **Configure authentication type and endpoints** — Select the auth type from the table above and provide the relevant URLs (token endpoint, authorisation endpoint, audience, scopes).

3. **Set the credential model** — Choose Team or Member credentials based on the delegation requirement.

4. **Enter credentials** — Provide the client ID, client secret, API key, or other credential values. These are encrypted immediately using AES-256 at rest and never returned in plaintext after storage.

5. **Bind to a consumer** — Associate the connection with one or more tools or agents that will use it. Multiple tools can share a single connection.

6. **Execute with authentication** — When a tool is invoked by the agent, the Tool Runtime Manager retrieves the connection, performs the auth handshake (or injects the stored token), and passes the authorised request to the target API.

---

## Token Refresh

For OAuth 2 connections, token refresh is handled automatically by the platform:

- The platform tracks access token expiry and performs a **lazy refresh** — tokens are refreshed just before they expire, not on a fixed schedule
- Refresh tokens are stored encrypted alongside access tokens
- If a refresh token itself expires (e.g., due to long inactivity), the user must re-authenticate through the consent flow

!!! warning "Refresh token expiry"
    For member credential connections using OAuth 2 Code flow, if a user does not interact with the agent for an extended period, their refresh token may expire. The agent will receive an authentication error on the next tool call and the user will need to re-authorise. Design agent prompts to handle this gracefully with a clear re-authorisation message.

---

## Binding Connections to Tools

Once created, a connection is bound to a tool at tool registration time:

- OpenAPI tools specify the connection in the authentication section of the tool definition
- Python tools access the injected token via the `context.auth_token` parameter in the script
- A single connection can be bound to multiple tools — changes to the connection (e.g., rotating a client secret) propagate to all bound tools without requiring tool re-registration

---

## Related References

- [Security](security.md) — OBO flows, SSO integration, and credential encryption details
- [Tool Types](tool-types.md) — How the Tool Runtime Manager consumes connection credentials
- [HOWTO: Configure a Connection](../howto/configure-connection.md) — Step-by-step configuration guide
- [Cookbook: Slack + Okta + Workday SSO](../cookbook/slack-sso-workday.md) — End-to-end OAuth 2 connection recipe
- [Cookbook: OBO Flow with Salesforce](../cookbook/obo-salesforce.md) — On-Behalf-Of token exchange recipe
