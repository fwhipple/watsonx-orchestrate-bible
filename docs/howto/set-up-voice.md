# Set Up a Voice Channel

Configure a complete voice channel for a watsonx Orchestrate agent — from speech provider selection through to telephony integration.

## Prerequisites

!!! note "Prerequisites"
    - watsonx Orchestrate SaaS account (or CPD with Watson Speech installed)
    - For SIP: account with a SIP trunk provider (Five9, Nice CX One, or similar)
    - For Genesys: Genesys Cloud CX admin access
    - Agent already created and tested via the web chat interface

## Steps

### Step 1 — Create a Voice Configuration

Navigate to **Admin → Voice Configurations → New Voice Config**. Give it a descriptive name (e.g., `en-us-enterprise-voice`).

### Step 2 — Configure Speech-to-Text (STT)

Select your STT provider and configure:

| Provider | Status | When to Use |
|----------|--------|-------------|
| Watson Speech | GA | Default for IBM Cloud; fully on-premises capable |
| DeepGram | GA | Strong accuracy for noisy telephony environments |
| Google STT | Coming soon | Broadest language coverage |

!!! note "On-Prem CPD"
    On-premises deployments are limited to Watson Speech and DeepGram. Cloud-based STT providers require outbound internet access.

Set **Language** (e.g., `en-US`) and **Model** (choose `telephony` for phone calls; `broadband` for web/headset).

### Step 3 — Configure Text-to-Speech (TTS)

Select TTS provider and voice. For the most natural-sounding voice, ElevenLabs offers the widest selection. For a fully on-premises deployment, use Watson TTS.

Set **Speed** (1.0 = normal) and **Stability** (0.75 = balanced consistency/expressiveness).

### Step 4 — Configure VAD (Voice Activity Detection)

| Parameter | Recommended Value | Effect |
|-----------|-----------------|--------|
| `silence_threshold_ms` | 800ms | Wait 800ms of silence before processing — good balance for most callers |
| `min_speech_duration_ms` | 200ms | Ignore clicks and brief sounds under 200ms |

For callers who tend to pause mid-sentence (elderly callers, non-native speakers), increase `silence_threshold_ms` to 1200ms.

### Step 5 — Add Hold Messages

Add at least 3–5 hold message variants. These are randomly selected when the agent invokes a tool:

```
"I'm looking into that for you now."
"Just a moment while I check."
"Let me pull that up."
"One moment please."
"I'm checking that right away."
```

### Step 6 — Enable DTMF (if needed)

Toggle DTMF on if callers need to use their phone keypad (e.g., for PIN entry or numeric input). Set the terminator key (default: `#`).

### Step 7 — Save the Voice Configuration

Click **Save**. The voice configuration is now available to attach to channels.

---

### Step 8 — Create the Channel

=== "Embed Chat (Voice Widget)"

    Navigate to **Channels → New Channel → Embed Chat**.
    
    Toggle **Enable Voice** to on. Select the voice configuration created above.
    
    Copy the updated HTML snippet and add it to your web page:
    
    ```html
    <script>
      window.WatsonxOrchestrate = {
        agentId: 'your-agent-id',
        channels: { voice: { enabled: true } }
      };
    </script>
    <script src="https://api.orchestrate.cloud.ibm.com/embed/widget.js"></script>
    ```
    
    A microphone icon appears in the chat widget for voice interactions.

=== "SIP Trunk"

    Navigate to **Channels → New Channel → SIP Trunk**.
    
    Enter your SIP trunk provider credentials:
    
    | Field | Value |
    |-------|-------|
    | SIP domain | Your provider's SIP domain (e.g., `company.sip.five9.com`) |
    | Username | SIP account username |
    | Password | SIP account password |
    | Phone number | The DID/number assigned to this channel |
    
    Select the voice configuration. Click **Save**.
    
    Provide the Orchestrate SIP endpoint URI to your SIP trunk provider to complete the trunk configuration.

=== "Genesys Cloud"

    Navigate to **Channels → New Channel → Genesys Cloud**.
    
    Enter your Genesys Cloud credentials:
    
    | Field | Value |
    |-------|-------|
    | Region | Your Genesys Cloud region (e.g., `mypurecloud.com`) |
    | Client ID | Genesys Cloud OAuth client ID |
    | Client Secret | Genesys Cloud OAuth client secret |
    
    Select the voice configuration. In Genesys Architect, point the bot flow to the Orchestrate Genesys connector endpoint.

---

### Step 9 — Assign Agent to Channel

In the channel settings, select the agent that should handle voice interactions on this channel.

### Step 10 — Test

- **Embed Chat**: Open the web page containing the widget; click the microphone icon; speak a test utterance
- **SIP/Genesys**: Call the phone number assigned to the channel

## Verification

Open the agent trace after a test call. Confirm three span types appear:
1. STT span — shows the transcribed text
2. Tool call span (if a tool was triggered) — shows the tool invocation
3. TTS span — shows the text that was synthesised to speech

## See Also

- [Voice](../reference/voice.md) — Full voice config schema, all STT/TTS providers, DTMF/VAD reference
- [Channels](../reference/channels.md) — Channel matrix and SSO configuration
- [Voice Roadmap](../reference/roadmap/voice-roadmap.md) — Upcoming providers and Go runtime latency improvements
