# Lab 1: Your First Agent

**Objective:** Create a watsonx Orchestrate SaaS account, navigate the platform, and build a working agent with a single tool that you can have a real conversation with.

**Estimated Time:** 45 minutes

**Prerequisites:**

!!! note "Before you start"
    - IBM watsonx Orchestrate SaaS account ([sign up here](https://www.ibm.com/products/watsonx-orchestrate))
    - No prior experience with watsonx Orchestrate required

---

## Part 1 — Set Up Your Account

### Step 1.1 — Sign In

Navigate to your Orchestrate instance URL and sign in with your IBM ID. You will land on the **Builder** workspace home page.

Spend 2 minutes exploring the navigation:

- **Agent Builder** — where you create and configure agents
- **Tool Studio** — where you register tools
- **Connections Manager** — where you manage authentication credentials
- **Channels** — where you configure deployment channels
- **Admin Console** — platform-wide settings (admin role required)

### Step 1.2 — Create an API Key

You will need an IBM Cloud API key for later labs. Create one now:

1. Navigate to [cloud.ibm.com/iam/apikeys](https://cloud.ibm.com/iam/apikeys)
2. Click **Create an IBM Cloud API key**
3. Name it `wxo-labs-key`
4. **Copy and save the key value** — it will not be shown again

---

## Part 2 — Register a Tool

You will register a free public API as a tool. We will use the Open-Meteo weather API — no authentication required.

### Step 2.1 — Download the OpenAPI Spec

Save this as `weather-api.yaml` on your computer:

```yaml
openapi: 3.0.0
info:
  title: Open-Meteo Weather API
  version: "1.0"
  description: Get current weather for any location
servers:
  - url: https://api.open-meteo.com/v1
paths:
  /forecast:
    get:
      operationId: get_weather_forecast
      summary: Get the weather forecast for a specific latitude and longitude
      parameters:
        - name: latitude
          in: query
          required: true
          description: The latitude of the location (-90 to 90)
          schema:
            type: number
        - name: longitude
          in: query
          required: true
          description: The longitude of the location (-180 to 180)
          schema:
            type: number
        - name: current
          in: query
          required: false
          description: Comma-separated list of current weather variables to return, e.g. temperature_2m,wind_speed_10m
          schema:
            type: string
            default: "temperature_2m,wind_speed_10m,weather_code"
      responses:
        "200":
          description: Weather forecast data
```

### Step 2.2 — Upload to Tool Studio

1. Navigate to **Tool Studio → New Tool → OpenAPI**
2. Click **Upload file** and select `weather-api.yaml`
3. Review the discovered operation: `get_weather_forecast`
4. Click **Save Tool**

---

## Part 3 — Build the Agent

### Step 3.1 — Create a New Agent

1. Navigate to **Agent Builder → New Agent**
2. Fill in:
   - **Name:** `Acme HR Assistant`
   - **Description:** `Handles HR queries, checks weather for travel planning, and assists with general employee questions for Acme Corp.`

### Step 3.2 — Write System Instructions

In the **Instructions** field, enter:

```
You are the Acme Corp HR Assistant. Your job is to help Acme Corp employees with:
- General HR questions
- Policy lookups
- Weather information to help with travel and event planning

When a user asks about the weather for a location, use the get_weather_forecast tool.
Always convert latitude/longitude values accurately. For major cities, use these coordinates:
- New York: 40.71, -74.01
- London: 51.51, -0.13
- Tokyo: 35.68, 139.69
- Sydney: -33.87, 151.21

Respond in a friendly, professional tone. Keep responses concise.
```

### Step 3.3 — Select a Model

Under **Settings → Model**, select `granite-3-8b-instruct` (fast, appropriate for this simple use case).

### Step 3.4 — Add the Weather Tool

Navigate to **Tools → Add Tool** and select `Open-Meteo Weather API`. Save the agent.

---

## Part 4 — Have a Conversation

The chat preview panel is open on the right side of Agent Builder. Try these prompts:

1. "What's the weather like in London today?"
2. "Is it a good day to hold an outdoor event in Tokyo?"
3. "What are Acme Corp's core working hours?" *(the agent should handle this gracefully even though it doesn't have HR data yet)*

---

## Validation Checkpoints

- [ ] The agent responds to the weather question with actual temperature and wind speed data
- [ ] The agent correctly identified London's coordinates and called `get_weather_forecast`
- [ ] The agent responds gracefully to the HR policy question (it will admit it doesn't have that information — that's correct behaviour)
- [ ] No error banners appear in the Agent Builder

!!! note "On-Prem CPD"
    On CPD, navigate to the equivalent Agent Builder in the Cloud Pak for Data UI. The agent creation steps are identical; only the URL differs.

---

## What You Learned

- How to navigate the watsonx Orchestrate SaaS console
- How to register an OpenAPI tool in Tool Studio
- How to write agent system instructions using the Role–Goal–Context–Constraints pattern
- How to test an agent in the chat preview panel

---

**Next Lab:** [Lab 2 — Tools Deep Dive](lab-02-tools-deep-dive.md) — Add Python and MCP tools to the agent
