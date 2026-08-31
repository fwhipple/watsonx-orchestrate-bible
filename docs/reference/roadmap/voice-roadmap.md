# Voice Roadmap

Upcoming features for voice configuration, speech providers, telephony integrations, and latency improvements.

!!! warning "Roadmap accuracy"
    Items reflect information from the IBM watsonx Orchestrate Deep Dive Enablement sessions. Verify current status with your IBM account team or official release notes.

---

## September

| Feature | Details |
|---------|---------|
| **Go Runtime for voice** | Switch voice agents to the Go runtime for significantly lower end-to-end latency; strongly recommended for production voice deployments |
| **Multiple hold messages** | Configure multiple hold message variants that are randomly selected during tool calls, reducing repetitive audio feedback |
| **Additional TTS providers** | Expansion of text-to-speech provider options (exact providers TBD at release) |

## October

| Feature | Details |
|---------|---------|
| **Telephony agent improvements** | Contact centre telephony improvements; Go runtime benefits applied to telephony agents |

## November

| Feature | Details |
|---------|---------|
| **Private link connection for SIP** | Private network (non-public-internet) SIP trunking support for highly secure enterprise telephony deployments |
| **Per-tenant TTS customisation** | Custom voice IDs and pronunciation dictionaries configurable per tenant, enabling brand-voice deployments without requiring a custom TTS endpoint |

## Future (STT Providers)

| Provider | Status | Notes |
|----------|--------|-------|
| **Google STT** | Coming soon | Broad language coverage (125+ languages) |
| **Azure Cognitive Speech** | Coming soon | Strong enterprise support and compliance certifications |
| **ElevenLabs STT** | Coming soon | High-fidelity transcription, particularly strong for accented speech |
