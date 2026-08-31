# Lab 8: Observability

**Objective:** Explore the Orchestrate observability stack — view agent traces, add custom span attributes to a Python tool, and configure the OTLP exporter to forward traces to an external platform.

**Estimated Time:** 45 minutes

**Prerequisites:**

!!! note "Before you start"
    - Completed [Lab 7](lab-07-evaluation.md)
    - A Datadog or New Relic account (free tier is sufficient — or use New Relic's free OTLP endpoint)
    - If using New Relic: sign up at [newrelic.com](https://newrelic.com) and obtain your License Key from **API Keys → License Key**

---

## Part 1 — Explore Agent Traces in the UI

### Step 1.1 — Generate Some Trace Data

Have a 3-turn conversation with the Acme Employee Assistant that involves at least one tool call:

1. "What's the weather in Tokyo?"
2. "How many PTO days is August 11th to August 22nd?"
3. "Look up employee EMP-003"

### Step 1.2 — Open the Observability UI

Navigate to **Observability → Agent Traces**. Find your recent session.

Click on the trace to open the **span waterfall view**.

Explore:

| Span Type | What to Look For |
|-----------|-----------------|
| LLM turn span | Duration of the model inference, token counts |
| Tool call span | HTTP status, tool name, latency |
| Knowledge retrieval span | (if knowledge configured) chunk count, retrieval latency |

### Step 1.3 — Examine a Tool Call Span

Click on the `get_weather_forecast` tool call span. Examine:
- The outbound HTTP URL and parameters
- The response status code
- The span duration
- The parent span (should be the LLM turn span that decided to call this tool)

---

## Part 2 — Add Custom Span Attributes to a Python Tool

### Step 2.1 — Update the PTO Calculator

Update your `pto_calculator.py` to add business-meaningful span attributes:

```python
from datetime import date, timedelta

async def main(
    start_date: str,
    end_date: str,
    exclude_weekends: bool = True,
    context: dict = {}
) -> dict:
    """Calculate the number of PTO days between two dates.
    
    Args:
        start_date: The start date of the PTO period in YYYY-MM-DD format
        end_date: The end date of the PTO period in YYYY-MM-DD format
        exclude_weekends: Set to true to exclude Saturdays and Sundays from the count (default: true)
    """
    # Import the span logger
    try:
        from ibm_watsonx_orchestrate.observability import span_logger
        has_span_logger = True
    except ImportError:
        has_span_logger = False

    start = date.fromisoformat(start_date)
    end = date.fromisoformat(end_date)
    
    if end < start:
        return {"error": "end_date must be on or after start_date"}
    
    total_days = 0
    weekend_days = 0
    current = start
    
    while current <= end:
        if current.weekday() >= 5:
            weekend_days += 1
        else:
            total_days += 1
        current += timedelta(days=1)
    
    if not exclude_weekends:
        total_days += weekend_days
    
    result = {
        "pto_days_requested": total_days,
        "total_calendar_days": (end - start).days + 1,
        "weekend_days_excluded": weekend_days if exclude_weekends else 0,
        "start_date": start_date,
        "end_date": end_date
    }

    # Add custom span attributes for observability
    if has_span_logger:
        span_logger.set_attribute("pto.days_requested", total_days)
        span_logger.set_attribute("pto.calendar_days", (end - start).days + 1)
        span_logger.set_attribute("pto.weekend_days_excluded", weekend_days)
        span_logger.set_attribute("pto.requires_approval", total_days > 5)
        span_logger.set_attribute("pto.start_month", start.strftime("%B"))

    return result
```

### Step 2.2 — Upload the Updated Tool

In Tool Studio, open the `pto_calculator` tool and upload the updated script. Save.

### Step 2.3 — Trigger a Trace

Ask the agent: "How many days PTO is September 1st to September 19th?"

### Step 2.4 — Inspect Custom Attributes in the Trace

Open the trace in the Observability UI. Click the `pto_calculator` span. In the **Span Attributes** panel, you should see:

- `pto.days_requested`: 14 (or similar)
- `pto.requires_approval`: true
- `pto.start_month`: September

These attributes are now queryable and filterable in the observability UI and in any external platform you export to.

---

## Part 3 — Configure External OTLP Export

### Step 3.1 — Get a New Relic Ingest Key (Free Tier)

1. Sign in to New Relic
2. Navigate to **Profile → API Keys**
3. Click **Create a key** → type: **Ingest - License**
4. Copy the key value

### Step 3.2 — Configure Export Settings

Navigate to **Admin Console → Observability → Export Settings**.

Enter:
- **Endpoint:** `https://otlp.nr-data.net:4317`
- **Header name:** `api-key`
- **Header value:** `<your-new-relic-license-key>`
- **Protocol:** gRPC

Click **Save and Validate**. You should see a green confirmation.

### Step 3.3 — Trigger Traces and Verify in New Relic

Send 2–3 messages to the agent. Wait 60 seconds.

In New Relic:
1. Navigate to **APM & Services → Distributed Tracing**
2. Search for `service.name = "watsonx-orchestrate"`
3. Find a trace from your recent conversation
4. Drill into the `pto_calculator` span — confirm your custom attributes (`pto.days_requested`, `pto.requires_approval`) appear as span attributes

---

## Validation Checkpoints

- [ ] Span waterfall view shows LLM turn, tool call, and (if configured) knowledge retrieval spans
- [ ] Tool call span shows the HTTP URL, status code, and duration
- [ ] Updated PTO calculator uploads successfully
- [ ] Custom span attributes appear in the Orchestrate trace UI
- [ ] New Relic (or Datadog) shows traces within 60 seconds of the conversation
- [ ] Custom attributes are visible in the external platform's trace detail view

!!! note "On-Prem CPD"
    On-premises deployments configure OTLP export via `values.yaml`. See [Cookbook: Custom Observability](../cookbook/custom-observability.md) for the CPD configuration.

---

## What You Learned

- How to navigate the Orchestrate observability span waterfall UI
- How to add custom span attributes to Python tools using the `span_logger` API
- How to configure the OTLP exporter for real-time trace forwarding to external platforms
- How custom attributes enable powerful filtering and alerting in Datadog/New Relic

---

**Next Lab:** [Lab 9 — Voice & Channels](lab-09-voice-channels.md)
