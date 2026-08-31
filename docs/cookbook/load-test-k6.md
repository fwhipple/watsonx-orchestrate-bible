# Load Test a Multi-Turn Agent with K6

## Problem

You need to validate that an agent meets latency and error rate SLAs before a production launch, simulating realistic multi-turn conversations at scale.

## Solution

Write a K6 load test script that authenticates once per virtual user, simulates multi-turn conversations with realistic think time, and validates response content — not just HTTP status codes.

## Prerequisites

- K6 installed: `brew install k6` (macOS) or from [k6.io/docs/getting-started/installation](https://k6.io/docs/getting-started/installation/)
- An IBM Cloud API key with access to the target Orchestrate SaaS tenant
- The Agent ID of the agent to test

## Steps

### Step 1 — Create the K6 Script

Save as `agent-load-test.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';
import { uuidv4 } from 'https://jslib.k6.io/k6-utils/1.4.0/index.js';

// Test configuration: ramp to 50 users over 5 minutes, hold for 30 minutes, ramp down
export const options = {
  stages: [
    { duration: '5m', target: 50 },    // Ramp-up
    { duration: '30m', target: 50 },   // Steady state
    { duration: '5m', target: 0 },     // Ramp-down
  ],
  thresholds: {
    http_req_duration: ['p(95)<8000'], // P95 latency < 8 seconds
    http_req_failed: ['rate<0.01'],    // Error rate < 1%
  },
};

const BASE_URL = 'https://api.orchestrate.cloud.ibm.com';
const AGENT_ID = __ENV.AGENT_ID;

// Authenticate once during setup; return the token for all VUs
export function setup() {
  const tokenRes = http.post(
    'https://iam.cloud.ibm.com/identity/token',
    `grant_type=urn:ibm:params:oauth:grant-type:apikey&apikey=${__ENV.IBM_API_KEY}`,
    { headers: { 'Content-Type': 'application/x-www-form-urlencoded' } }
  );

  check(tokenRes, { 'IAM token obtained': (r) => r.status === 200 });
  return { token: tokenRes.json('access_token') };
}

// Simulate realistic multi-turn conversations
export default function (data) {
  const sessionId = uuidv4();
  const headers = {
    'Authorization': `Bearer ${data.token}`,
    'Content-Type': 'application/json',
  };

  // Represent a realistic conversation for your use case
  const turns = [
    "What is my current PTO balance?",
    "How many days have I taken this year?",
    "Can I take a week off in August? I need July 14th to July 18th.",
    "What is the approval process for PTO requests?",
    "Thanks, I'll submit the request now.",
  ];

  for (const message of turns) {
    const res = http.post(
      `${BASE_URL}/v1/agents/${AGENT_ID}/chat`,
      JSON.stringify({
        session_id: sessionId,
        input: { text: message },
      }),
      { headers, timeout: '30s' }
    );

    const checks = check(res, {
      'status 200': (r) => r.status === 200,
      'has response text': (r) => {
        try { return r.json('output.text') !== undefined; }
        catch { return false; }
      },
      'no error in response': (r) => {
        try { return !r.json('output.text').toLowerCase().includes('error occurred'); }
        catch { return true; }
      },
    });

    // Log failures for investigation
    if (!checks) {
      console.error(`FAIL session=${sessionId} turn="${message.substring(0, 30)}" status=${res.status}`);
    }

    // Realistic think time: 3–12 seconds between turns
    sleep(Math.random() * 9 + 3);
  }
}
```

### Step 2 — Run the Load Test

```bash
# Load test against SaaS
k6 run \
  -e IBM_API_KEY=<your-ibm-cloud-api-key> \
  -e AGENT_ID=<your-agent-id> \
  agent-load-test.js

# With output to a file for analysis
k6 run \
  -e IBM_API_KEY=<your-api-key> \
  -e AGENT_ID=<your-agent-id> \
  --out json=results.json \
  agent-load-test.js
```

### Step 3 — Interpret Results

K6 outputs a summary at the end of the run. Key values to check:

```
http_req_duration............: avg=3.2s   min=1.1s   med=2.8s   max=18.4s  p(90)=5.9s   p(95)=7.2s
http_req_failed..............: 0.12%   ✓ 2884  ✗ 3
checks.......................: 99.9%  ✓ 8652  ✗ 3
```

| K6 Output | What to Look For |
|-----------|-----------------|
| `p(95)` latency | Must be below your SLA target (typically < 8s for full response) |
| `http_req_failed` rate | Must be below threshold (typically < 1%) |
| `checks` pass rate | Should be > 99% if content validation is included |

### Step 4 — Run a Stress Test

To find the breaking point, modify the stages:

```javascript
export const options = {
  stages: [
    { duration: '10m', target: 50 },   // 50 users (expected load)
    { duration: '10m', target: 100 },  // 100 users (2× expected)
    { duration: '10m', target: 150 },  // 150 users (3× expected)
    { duration: '10m', target: 200 },  // 200 users (4× expected)
    { duration: '5m', target: 0 },     // Ramp down
  ],
};
```

Stop the test when the error rate exceeds 5% or P95 latency exceeds 15 seconds. The stage where degradation begins is your capacity ceiling.

## Verification

A successful load test result shows:
- P95 latency within your SLA target throughout the 30-minute steady state
- Error rate below 1%
- No sustained increase in error rate over time (which would indicate a memory leak or connection pool exhaustion)

## See Also

- [Operations: Load Testing](../operations/load-testing.md) — Full scripting patterns, test types, and SaaS vs on-prem differences
- [Lab 10: Load Testing](../labs/lab-10-load-testing.md) — Hands-on K6 load test lab against SaaS
