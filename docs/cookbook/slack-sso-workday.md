# Slack + Okta + Workday SSO

## Problem

Users interact with an agent via Slack. The agent needs to query Workday on behalf of each individual user — not as a shared service account. The authentication chain spans Slack, Okta, and Workday, requiring SAML federation and a two-step OBO token exchange.

## Solution

Configure Slack as a channel with Okta SAML SSO, register Orchestrate as a SAML Service Provider in Okta, and set up a Workday OBO connection that exchanges an Okta token for a Workday access token.

## Prerequisites

- Okta tenant with SAML SSO configured
- Slack workspace admin access
- Workday OAuth 2.0 app registration (Client Credential or JWT Bearer)
- watsonx Orchestrate admin role
- A Workday OpenAPI tool registered in Tool Studio

## Steps

### Step 1 — Register Orchestrate as a SAML SP in Okta

1. In Okta Admin, navigate to **Applications → Create App Integration → SAML 2.0**
2. Set the **Single Sign-On URL** (ACS URL) to your Orchestrate tenant's SAML callback endpoint:
   ```
   https://<your-tenant>.orchestrate.cloud.ibm.com/saml/callback
   ```
3. Set the **Audience URI** (Entity ID):
   ```
   https://<your-tenant>.orchestrate.cloud.ibm.com/saml/metadata
   ```
4. Set **Name ID format** to `EmailAddress`
5. Add attribute statements: `email`, `displayName`, `groups`
6. Download the **Okta metadata XML** file

### Step 2 — Configure Okta as IdP in Orchestrate

1. Navigate to **Admin → Identity Providers → New IdP**
2. Upload the Okta metadata XML downloaded in Step 1
3. Map IdP attributes to Orchestrate user fields:
   ```yaml
   email: "${idp.email}"
   display_name: "${idp.displayName}"
   roles: "${idp.groups}"  # Map Okta groups to Orchestrate roles
   ```
4. Click **Save**

### Step 3 — Create the Workday OBO Connection

1. Navigate to **Connections Manager → New Connection**
2. Name: `workday-obo-member`
3. Auth type: **OAuth 2 Client Credential** (or JWT Bearer if your Workday app uses JWT)
4. Token endpoint: your Workday OAuth 2 token endpoint
5. For OBO configuration, enable **On-Behalf-Of** and set:
   - Assertion source: `okta-saml-session-token`
   - Audience: your Workday OAuth app client ID
6. Credential model: **Member**
7. Click **Save**

### Step 4 — Configure the Slack Channel

1. Navigate to **Channels → New Channel → Slack**
2. Enter your Slack app Bot Token and App credentials
3. Enable **SSO**: select the Okta IdP configured in Step 2
4. Under **Auth Flow**, select **OAuth 2 Code** (for member credentials)
5. Set the redirect URI in your Slack app to the Orchestrate OAuth callback URL
6. Click **Save**

### Step 5 — Bind the Workday Connection to Your Tools

In Tool Studio, open each Workday tool. In the **Authentication** section, select `workday-obo-member`. Save each tool.

### Step 6 — Test the Flow

1. Message the Orchestrate bot in Slack
2. You will be prompted to log in via Okta
3. Authenticate with your Okta credentials (MFA if configured)
4. Ask the agent: "What is my Workday PTO balance?"

The agent executes: Slack message → Okta SAML assertion → Orchestrate session → Workday OBO token exchange → Workday API call → response

## Verification

Open the agent trace. Confirm:
1. An authentication span shows the Okta SAML assertion was received
2. A connection span shows the OBO exchange producing a Workday token
3. The Workday API response is user-scoped (contains the user's employee ID, not a service account)

## See Also

- [Security](../reference/security.md#sso-integration) — SAML assertion flow and OBO architecture
- [Channels](../reference/channels.md) — Channel matrix and SSO support
- [Connections](../reference/connections.md) — Auth types and connection lifecycle
