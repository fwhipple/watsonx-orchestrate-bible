# Lab 4: Agentic Memory

**Objective:** Enable agentic memory on the Acme HR Assistant, verify that user preferences persist across separate sessions, and manage memory records via the UI and Python SDK.

**Estimated Time:** 30 minutes

**Prerequisites:**

!!! note "Before you start"
    - Completed [Lab 3](lab-03-connections-auth.md)
    - Python 3.11+ with `ibm-watsonx-orchestrate` SDK installed: `pip install ibm-watsonx-orchestrate`

---

## Part 1 — Enable Memory on the Agent

### Step 1.1 — Enable Memory Toggle

1. Open the `Acme HR Assistant` in Agent Builder
2. Navigate to **Settings → Memory**
3. Toggle **Enable agentic memory** to **On**
4. Click **Save**

### Step 1.2 — User Opt-In

Memory only captures facts for users who have opted in. Enable it for your own account:

1. Click your profile icon (top-right)
2. Navigate to **Profile → Memory**
3. Toggle **Enable memory collection** to **On**

---

## Part 2 — Test Memory Persistence

### Step 2.1 — Session 1: Establish a Preference

Open the chat preview and have this conversation:

> **You:** "My name is Alex and I'm in the Sydney office. I always want dates displayed in DD/MM/YYYY format."
>
> **Agent:** *(acknowledges and uses your preference)*
>
> **You:** "How many days PTO do I have left? I've taken 8 days so far and my annual allowance is 20."
>
> **Agent:** *(calculates and responds — note the date format it uses)*

Close the chat preview panel.

### Step 2.2 — Session 2: Verify Persistence

Click **New Session** (or reload the page and reopen the agent). Start a completely fresh conversation:

> **You:** "I need to book time off from 1st March to 5th March next year. How many working days is that?"

The agent should:
1. Remember your name is Alex
2. Remember you're in the Sydney office (relevant for public holidays)
3. Use DD/MM/YYYY format for dates without you asking

---

## Part 3 — Manage Memories via the UI

### Step 3.1 — View Stored Memories

Navigate to **Profile → Memory panel**. You should see memory records similar to:

- "User's name is Alex"
- "User is located in the Sydney office"
- "User prefers dates in DD/MM/YYYY format"
- "User's annual PTO allowance is 20 days"

### Step 3.2 — Delete a Memory

Click the delete icon next to the date format memory. In the next session, the agent should no longer remember your date preference.

---

## Part 4 — Manage Memories via Python SDK

### Step 4.1 — List All Memories

```python
from ibm_watsonx_orchestrate import MemoryClient
import os

client = MemoryClient(
    api_key=os.environ["IBM_API_KEY"],
    base_url="https://api.orchestrate.cloud.ibm.com"
)

# List all memories
memories = client.memory.list()
for m in memories:
    print(f"ID: {m.id}")
    print(f"Content: {m.content}")
    print(f"Created: {m.created_at}")
    print(f"Sensitivity: {m.sensitivity}")
    print("---")
```

Run this script:
```bash
IBM_API_KEY=<your-api-key> python list_memories.py
```

### Step 4.2 — Delete a Specific Memory

```python
# Delete the first memory in the list
if memories:
    client.memory.delete(memory_id=memories[0].id)
    print(f"Deleted memory: {memories[0].id}")
```

---

## Part 5 — Observe Sensitive Data Handling

Try establishing a memory that contains sensitive information:

> **You:** "My employee ID is EMP-001234 and my payroll reference is PAY-9876."

Navigate to **Profile → Memory panel**. The payroll reference should be flagged with a sensitivity classification indicator (or suppressed entirely if the platform's classifier identifies it as financial data).

!!! note "Sensitivity classification"
    The exact behaviour depends on the platform's classifier configuration. The key point is that the platform applies sensitivity checks automatically — no developer action is required.

---

## Validation Checkpoints

- [ ] Agent correctly recalled name, location, and date format preference in Session 2
- [ ] Memory records are visible in the Profile memory panel
- [ ] Deleting a memory via UI removes it from the panel
- [ ] Python SDK successfully lists and deletes memory records
- [ ] Sensitive data (payroll reference) is handled with a sensitivity label

!!! warning "30-day retention"
    Memory records expire after 30 days of inactivity. For the purposes of this lab, all memories will persist until you delete them or they expire.

---

## What You Learned

- How to enable agentic memory on an agent and for a user account
- How memory persists across sessions and reduces repetition
- How to manage memory via the UI and the Python SDK
- How the platform handles sensitive data in memory records

---

**Next Lab:** [Lab 5 — Agentic Workflows](lab-05-agentic-workflows.md)
