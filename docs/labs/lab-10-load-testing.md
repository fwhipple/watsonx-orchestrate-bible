# Lab 10: Load Testing

**Objective:** Write a realistic K6 load test script against the SaaS Acme Employee Assistant, execute a load test, and analyse the results to determine if the deployment meets SLA targets.

**Estimated Time:** 60 minutes

**Prerequisites:**

!!! note "Before you start"
    - Completed [Lab 9](lab-09-voice-channels.md)
    - K6 installed: `brew install k6` (macOS) or see [k6.io/docs/get-started/installation](https://k6.io/docs/get-started/installation/)
    - IBM Cloud API key from Lab 1
    - Agent ID of the `Acme Employee Assistant`

---

## Part 1 — Get Your Agent ID

If you don't already have your Agent ID:

```bash
# Install the IBM Cloud CLI if not already installed
# Then get an IAM token
ibmcloud login --apikey $IBM_API_KEY
TOKEN=$(ibmcloud iam oauth-tokens --output json | python3 -c "import sys,json; print(json.load(sys.stdin)['iam_token'].split(' ')[1])")

# List your agents
curl -s "https://api.orchestrate.cloud.ibm.com/v1/agents" \
  -H "Authorization: Bearer $TOKEN" | python3 -m json.tool | grep -A2 '"name": "Acme'
```

Note the agent's `id` field.

---

## Part 2 — Write the Load Test Script

Create `acme-load-test.js`:

```javascript
import http from 'k6/http';
import { check, sleep, group } from 'k6';
import { uuidv4 } from 'https://jslib.k6.io/k6-utils/1.4.0/index.js';
import { Trend, Rate, Counter } from 'k6/metrics';

// Custom metrics
const toolCallSuccessRate = new Rate('tool_call_success');
const firstTurnLatency = new Trend('first_turn_latency');
const weatherTurns = new Counter('weather_turns');
const ptoTurns = new Counter('pto_turns');

// Test configuration
export const options = {
  stages: [
    { duration: '2m', target: 10 },   // Warm-up: ramp to 10 users
    { duration: '10m', target: 10 },  // Steady state: 10 users for 10 minutes
    { duration: '2m', target: 0 },    // Ramp-down
  ],
  thresholds: {
    http_req_duration: ['p(95)<10000'],      // P95 < 10s (including LLM latency)
    http_req_failed: ['rate<0.02'],          // Error rate < 2%
    tool_call_success: ['rate>0.90'],        // Tool calls succeed > 90% of the time
  },
};

const BASE_URL = 'https://api.orchestrate.cloud.ibm.com';
const AGENT_ID = __ENV.AGENT_ID;

// Authenticate once during setup
export function setup() {
  const tokenRes = http.post(
    'https://iam.cloud.ibm.com/identity/token',
    `grant_type=urn:ibm:params:oauth:grant-type:apikey&apikey=${__ENV.IBM_API_KEY}`,
    { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } }
  );
  
  check(tokenRes, { 
    'IAM token obtained': (r) => r.status === 200,
    'token is not empty': (r) => r.json('access_token') !== ''
  });
  
  return { token: tokenRes.json('access_token') };
}

// Main test function — runs once per virtual user per iteration
export default function(data) {
  const sessionId = uuidv4();
  const headers = {
    'Authorization': `Bearer ${data.token}`,
    'Content-Type': 'application/json',
  };

  // Scenario 1: Weather query (direct tool, handled by orchestrator)
  group('Weather Query', () => {
    const cities = ['London', 'Tokyo', 'Sydney', 'New York', 'Paris'];
    const city = cities[Math.floor(Math.random() * cities.length)];
    
    const startTime = Date.now();
    const res = http.post(
      `${BASE_URL}/v1/agents/${AGENT_ID}/chat`,
      JSON.stringify({ session_id: sessionId, input: { text: `What is the weather in ${city}?` } }),
      { headers, timeout: '30s' }
    );
    firstTurnLatency.add(Date.now() - startTime);

    const success = check(res, {
      'weather: status 200': (r) => r.status === 200,
      'weather: has response': (r) => {
        try { return r.json('output.text') !== undefined; }
        catch { return false; }
      },
    });
    toolCallSuccessRate.add(success);
    weatherTurns.add(1);
    sleep(Math.random() * 5 + 3);
  });

  // Scenario 2: PTO calculation (delegates to HR Specialist)
  group('PTO Calculation', () => {
    const months = ['August', 'September', 'October'];
    const month = months[Math.floor(Math.random() * months.length)];

    const res = http.post(
      `${BASE_URL}/v1/agents/${AGENT_ID}/chat`,
      JSON.stringify({ session_id: sessionId, input: { text: `How many working days is the first two weeks of ${month} 2025?` } }),
      { headers, timeout: '30s' }
    );

    const success = check(res, {
      'pto: status 200': (r) => r.status === 200,
      'pto: has response': (r) => {
        try { return r.json('output.text') !== undefined; }
        catch { return false; }
      },
    });
    toolCallSuccessRate.add(success);
    ptoTurns.add(1);
    sleep(Math.random() * 8 + 5);
  });
}

// Print summary at the end
export function teardown(data) {
  console.log(`Load test complete. Token was: ${data.token.substring(0, 10)}...`);
}
```

---

## Part 3 — Run a Smoke Test First

Before the full load test, run a quick smoke test with 1 user to verify the script works:

```bash
k6 run \
  -e IBM_API_KEY=<your-api-key> \
  -e AGENT_ID=<your-agent-id> \
  --vus 1 \
  --duration 30s \
  acme-load-test.js
```

Confirm:
- No errors in the output
- Both scenarios execute successfully
- Responses contain weather data and PTO day counts

---

## Part 4 — Run the Full Load Test

```bash
k6 run \
  -e IBM_API_KEY=<your-api-key> \
  -e AGENT_ID=<your-agent-id> \
  --out json=lab10-results.json \
  acme-load-test.js
```

The test runs for approximately 14 minutes (2min ramp-up + 10min steady + 2min ramp-down).

---

## Part 5 — Analyse the Results

### Step 5.1 — Review the K6 Summary

K6 prints a summary at the end. Record your results:

```
http_req_duration.............: avg=___s  p(50)=___s  p(95)=___s  p(99)=___s
http_req_failed...............: ___% 
tool_call_success.............: ___%
first_turn_latency............: avg=___ms p(95)=___ms
weather_turns.................: ___ total
pto_turns.....................: ___ total
```

### Step 5.2 — Compare Against SLA Targets

| Metric | Your Result | SLA Target | Pass/Fail |
|--------|------------|------------|-----------|
| P95 end-to-end latency | ___ | < 10s | |
| Error rate | ___ | < 2% | |
| Tool call success rate | ___ | > 90% | |

### Step 5.3 — Identify Bottlenecks

If P95 > 10s, open the Orchestrate Observability UI during the test (or review after) to identify whether latency is in:
- **LLM inference span** → model is slow; consider switching to a faster model (Granite-8B)
- **Tool call span** → the external API (httpbin, Open-Meteo) is the bottleneck
- **Routing/agent turn overhead** → check if the orchestrator is routing correctly without extra LLM calls

### Step 5.4 — Run a Stress Test

To find the breaking point, edit the `stages` to progressively increase load:

```javascript
stages: [
  { duration: '5m', target: 20 },   // 2× expected
  { duration: '5m', target: 30 },   // 3× expected
  { duration: '5m', target: 50 },   // 5× expected
  { duration: '2m', target: 0 },
],
```

Note the concurrency level where error rate exceeds 5% or P95 > 15s.

---

## Validation Checkpoints

- [ ] Smoke test passes (1 user, 30 seconds, no errors)
- [ ] Full load test completes (14 minutes, 10 users)
- [ ] P95 latency and error rate recorded
- [ ] K6 threshold pass/fail status documented
- [ ] At least one bottleneck identified and investigated in traces

!!! note "On-Prem CPD load testing"
    For on-premises CPD, use the CPD authentication endpoint instead of IBM Cloud IAM. See [Operations: Load Testing](../operations/load-testing.md) for the CPD auth pattern and Prometheus-based monitoring.

---

## What You Learned

- How to write a realistic multi-scenario K6 load test with custom metrics
- How to authenticate to the SaaS API and cache tokens across virtual users
- How to interpret K6 summary output and correlate with Orchestrate trace data
- The difference between a load test (validate SLA) and a stress test (find breaking point)

---

**Next Lab:** [Lab 11 — External Agents & Control Plane](lab-11-external-agents.md)
