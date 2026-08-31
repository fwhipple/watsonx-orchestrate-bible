# Load Testing

Load testing watsonx Orchestrate agents verifies that your deployment meets throughput and latency requirements before go-live, and identifies the failure point under stress. This page describes the four-module testing framework, scripting patterns, test types, and the key metrics to collect and analyse.

---

## Testing Framework — Four Modules

### Module 1 — Requirements Gathering

Define the performance goals before writing a single test script.

**Deliverables:**
- Target concurrent user count (peak and steady-state)
- Acceptable P95 response latency (e.g., first token < 3 seconds, full response < 8 seconds)
- Acceptable error rate threshold (e.g., < 1% HTTP errors)
- Token budget estimate (average input/output tokens per conversation turn)
- Conversation structure: average turns per session, realistic message lengths

**Key questions to answer:**
- What is the expected ramp-up rate? (Users per minute)
- Is the load pattern steady, bursty, or scheduled?
- Are there peak times (e.g., Monday morning login surge)?

### Module 2 — Performance Script Creation

Select a load testing tool and build the test script.

**Recommended tools:**

| Tool | Best For | Notes |
|------|---------|-------|
| **K6** | Modern cloud-scale testing; JavaScript-based | Excellent for REST APIs; good metrics export |
| **Locust** | Python-based; easy to customise for complex scenarios | Great for multi-step conversation simulation |
| **JMeter** | Enterprise standard; GUI-driven | Heavier setup; good for teams already using it |
| **LoadRunner** | Enterprise; existing LoadRunner investments | Use if already licensed and used by the team |

### Module 3 — Performance Testing

Execute the test types described below (load, stress, spike) against the target environment.

### Module 4 — Analysis & Metrics

Collect, analyse, and report on the results. See the [Key Metrics](#key-metrics) section.

---

## Scripting Best Practices

### Session Management

Authenticate once per virtual user session, not per request. Token acquisition is expensive.

```javascript
// K6 example — authenticate once, reuse token throughout session
import http from 'k6/http';

export function setup() {
  // Obtain IAM token once during setup (SaaS)
  const res = http.post('https://iam.cloud.ibm.com/identity/token', {
    grant_type: 'urn:ibm:params:oauth:grant-type:apikey',
    apikey: __ENV.IBM_API_KEY,
  });
  return { token: res.json('access_token') };
}

export default function (data) {
  const headers = {
    'Authorization': `Bearer ${data.token}`,
    'Content-Type': 'application/json',
  };
  // Use headers for all agent API calls
}
```

!!! note "On-Prem CPD auth"
    On-premises deployments use the enterprise IdP (LDAP/Keycloak) rather than IBM Cloud IAM. Obtain a CPD user token via `POST /icp4d-api/v1/authorize` and cache it for the session duration (~1 hour).

### Conversation Structure

Model realistic conversations. Real users don't send a single message and disconnect.

```javascript
// Simulate a 5-turn conversation
const turns = [
  "What is my PTO balance?",
  "How many days have I taken this year?",
  "Can I book a week off in August?",
  "What's the approval process?",
  "Thanks, I'll submit the request"
];

for (const message of turns) {
  const payload = JSON.stringify({
    session_id: sessionId,
    input: { text: message }
  });
  const res = http.post(agentUrl, payload, { headers });
  check(res, { 'status 200': (r) => r.status === 200 });
  sleep(thinkTime()); // Simulate reading/typing
}
```

**Guidelines:**
- Target 5 turns per conversation on average; realistic max is 10–20
- Do not use the same static session ID across virtual users — generate unique session IDs

### Checkpoints

Don't just validate HTTP 200 — verify the response content.

```javascript
check(res, {
  'status 200': (r) => r.status === 200,
  'has output text': (r) => r.json('output.text') !== undefined,
  'not an error response': (r) => !r.json('output.text').includes('error occurred'),
});
```

### Think Time

Inject realistic delays between turns to simulate human reading and typing behaviour.

```javascript
function thinkTime() {
  // Random delay between 3 and 12 seconds
  return Math.random() * 9 + 3;
}
```

### Error Handling

LLMs are non-deterministic. Scripts must handle unexpected outputs gracefully.

- Retry on HTTP 429 (rate limit) with exponential backoff
- Log the full response body for any non-200 status — don't just count errors
- Validate expected tool calls appear in the response when testable

---

## Test Types

### Load Test

Validates baseline performance under expected production load.

```
Ramp-up:    5–10 minutes per 100 concurrent users
Steady state: 30 minutes at target concurrency
Ramp-down:  5 minutes
```

**Goal:** Confirm P95 latency and error rate meet targets at expected concurrency.

### Stress Test

Finds the system's breaking point by progressively increasing load.

```
Phase 1: 50% of expected load × 10 minutes
Phase 2: 100% of expected load × 10 minutes
Phase 3: 150% of expected load × 10 minutes
Phase 4: 200% of expected load × 10 minutes
... continue until error rate exceeds threshold or latency degrades unacceptably
```

**Goal:** Identify the concurrency ceiling and failure mode (graceful degradation vs hard failure).

### Spike Test

Simulates a sudden traffic surge.

```
Baseline:   10% of expected load × 5 minutes
Spike:      200% of expected load × 2 minutes (sudden jump)
Recovery:   10% of expected load × 5 minutes
```

**Goal:** Verify the system recovers gracefully after a sudden spike rather than remaining degraded.

---

## Key Metrics

| Metric | Description | Target (Typical) |
|--------|-------------|-----------------|
| **First Token Latency (TTFT)** | Time from request to first response token received | < 2–3 seconds |
| **Full Response Time** | Time to receive the complete response | < 8–10 seconds for most queries |
| **P50 Latency** | Median response time | Establishes baseline "normal" |
| **P95 Latency** | 95th percentile response time | Primary SLA measurement |
| **P99 Latency** | 99th percentile response time | Identifies tail latency outliers |
| **Error Rate** | Percentage of requests resulting in HTTP errors or invalid responses | < 1% (strict SLA) |
| **Throughput** | Successful requests per second | Varies by use case |
| **Input Token Usage** | Average input tokens per turn | Baseline for cost modelling |
| **Output Token Usage** | Average output tokens per turn | Baseline for cost modelling |

---

## SaaS vs On-Premises Differences

| Aspect | SaaS | On-Premises CPD |
|--------|------|-----------------|
| **Authentication** | IBM Cloud IAM token (`/identity/token`) | CPD user token (`/icp4d-api/v1/authorize`) |
| **Rate limits** | Governed by IBM SRE; request increases via support | Customer-configured; adjust OpenShift HPA (Horizontal Pod Autoscaler) |
| **Monitoring** | Orchestrate analytics only; no raw Prometheus access | Full Prometheus + Grafana; custom dashboards |
| **Tuning** | IBM SRE responsibility | Customer responsibility (pod replicas, resource limits, GPU scheduling) |
| **Latency baseline** | Shared infrastructure; additional network hop | On-premises; lower baseline latency possible |

---

## Related References

- [Architecture](architecture.md) — Component architecture underlying what you are testing
- [Operations: Troubleshooting](troubleshooting.md) — Diagnosing errors found during load tests
- [Cookbook: Load Test with K6](../cookbook/load-test-k6.md) — Complete K6 script for a multi-turn agent load test
- [Lab 10: Load Testing](../labs/lab-10-load-testing.md) — Hands-on lab running a K6 load test against the SaaS service
