# Lab 9: Voice & Channels

**Objective:** Deploy the Acme Employee Assistant to a Slack channel with SSO, then configure a voice setup so the agent can handle telephone interactions.

**Estimated Time:** 60 minutes

**Prerequisites:**

!!! note "Before you start"
    - Completed [Lab 8](lab-08-observability.md)
    - Slack workspace where you have admin access to install apps
    - Okta developer account (free tier at developer.okta.com) — or substitute with any SAML 2.0 IdP

---

## Part 1 — Deploy to Slack

### Step 1.1 — Create a Slack App

1. Navigate to [api.slack.com/apps](https://api.slack.com/apps)
2. Click **Create New App → From Scratch**
3. Name: `Acme Employee Assistant`
4. Select your workspace

In the app settings:
- Enable **Socket Mode** (for development — in production use Events API with HTTPS)
- Under **OAuth & Permissions**, add Bot Token Scopes: `chat:write`, `app_mentions:read`, `channels:history`
- Install the app to your workspace
- Copy the **Bot User OAuth Token** (starts with `xoxb-`)

### Step 1.2 — Configure the Slack Channel in Orchestrate

1. Navigate to **Channels → New Channel → Slack**
2. Enter the **Bot Token** (`xoxb-...`)
3. Enter the **Signing Secret** from your Slack app's Basic Information page
4. **Do not** enable SSO yet — we will test basic Slack first

Click **Save**.

### Step 1.3 — Assign the Agent

Select `Acme Employee Assistant` as the agent for this channel. Save.

### Step 1.4 — Test Basic Slack Integration

Go to your Slack workspace and find the `Acme Employee Assistant` bot. Send:
> "What's the weather in Sydney?"

The bot should respond with the weather data. 🎉

---

## Part 2 — Configure Slack SSO with Okta

### Step 2.1 — Create an Okta Application

1. Sign in to your Okta developer console at `dev-XXXXXX.okta.com`
2. Navigate to **Applications → Create App Integration → SAML 2.0**
3. App name: `watsonx Orchestrate`
4. SAML Settings:
   - **Single sign-on URL:** `https://<your-tenant>.orchestrate.cloud.ibm.com/saml/callback`
   - **Audience URI:** `https://<your-tenant>.orchestrate.cloud.ibm.com`
   - **Name ID format:** EmailAddress
5. Click **Finish**. Download the **Identity Provider metadata XML**.

### Step 2.2 — Register Okta as IdP in Orchestrate

1. Navigate to **Admin → Identity Providers → New IdP**
2. Upload the Okta metadata XML
3. Map attributes:
   - `email` → `${idp.email}`
   - `displayName` → `${idp.displayName}`
4. Save

### Step 2.3 — Enable SSO on the Slack Channel

1. Open the Slack channel in Channels
2. Enable **SSO**
3. Select the Okta IdP configured in Step 2.2
4. Save

### Step 2.4 — Test SSO Flow

In Slack, send a message to the bot. You should be prompted to authenticate via Okta. After authenticating:
- The agent now knows your Okta identity
- Tools using Member credentials can now act on your behalf

---

## Part 3 — Configure a Voice Channel

For this lab we will configure the Embed Chat voice widget (no SIP infrastructure required).

### Step 3.1 — Create a Voice Configuration

1. Navigate to **Admin → Voice Configurations → New Voice Config**
2. Name: `acme-voice-en-us`
3. **STT Provider:** Watson Speech, Language: en-US, Model: broadband
4. **TTS Provider:** Watson TTS, Voice: en-US_AllisonV3Voice
5. **VAD:** silence_threshold_ms: 800, min_speech_duration_ms: 200
6. **Hold Messages:**
   - "Let me check that for you."
   - "One moment please."
   - "I'm looking into that."
7. **DTMF:** Enabled, Terminator: #
8. Save

### Step 3.2 — Create an Embed Chat Channel with Voice

1. Navigate to **Channels → New Channel → Embed Chat**
2. Toggle **Enable Voice** to On
3. Select `acme-voice-en-us`
4. Assign `Acme Employee Assistant`
5. Save and copy the HTML snippet

### Step 3.3 — Test the Voice Widget

Create a test HTML file (`test-voice.html`):

```html
<!DOCTYPE html>
<html>
<head><title>Acme Voice Test</title></head>
<body>
<h1>Acme Employee Assistant</h1>
<!-- Paste the HTML snippet from Orchestrate here -->

</body>
</html>
```

Open in a browser. Click the microphone icon and say: "What's the weather in New York?"

Confirm the agent responds with synthesized speech.

---

## Part 4 — Observe Channel Context Variables

### Step 4.1 — Enable Context Access

Open the `Acme Employee Assistant` in Agent Builder → **Settings → Context**.
Enable `context_access_enabled`.

### Step 4.2 — Update Instructions for Channel Awareness

Add to the system instructions:

```
Context information is available:
- If channel_type is "voice", use short spoken-language sentences. Avoid bullet points, markdown, and lists.
- If channel_type is "slack", you may use markdown formatting.
- If the user's timezone is available, use it when discussing dates and times.
```

### Step 4.3 — Test Channel-Appropriate Formatting

Ask the same question via Slack and via the voice widget:
> "What's the weather in London and Tokyo?"

Observe:
- **Slack**: Response uses markdown formatting, structured
- **Voice**: Response uses natural spoken sentences, no bullet points

---

## Validation Checkpoints

- [ ] Agent responds in Slack to "What's the weather in Sydney?"
- [ ] SSO prompts Okta login on first Slack interaction
- [ ] Voice widget allows microphone input and responds with synthesized speech
- [ ] STT span appears in the agent trace for voice interactions
- [ ] Channel-aware formatting differs between Slack and voice responses

!!! note "On-Prem CPD voice"
    Watson Speech and DeepGram are available on CPD. Cloud STT providers (Google, Azure) require outbound internet access.

---

## What You Learned

- How to deploy an agent to Slack and configure SSO with Okta
- How to create a voice configuration with STT, TTS, VAD, and hold messages
- How to use the Embed Chat voice widget for browser-based voice testing
- How `context_access_enabled` allows a single agent to adapt its responses per channel

---

**Next Lab:** [Lab 10 — Load Testing](lab-10-load-testing.md)
