---
name: build-superflow
description: How to author a deterministic SuperFlow node graph and push it to the platform.
when_to_use: When building a SuperFlow / workflow — a repeatable, multi-step process with a known path.
---

# Build a SuperFlow

A SuperFlow is a **deterministic** node graph run by a durable engine — same path
every time. (If you'd need to reason per-request about which workers to call, it's
a *managerial agent*, not a SuperFlow.)

## Node vocabulary (`lyzr-nodes-base.*`)
- **Start:** exactly one `trigger`.
- **AI:** `lyzr.agent`, `lyzr.llm`, `lyzr.tool`, `lyzr.a2aAgent`, `lyzr.taskDecomposition`.
- **Safety:** `lyzr.guardrails` — output **0 = allowed**, **1 = blocked**.
- **Control:** `if`, `switch`, `merge`, `splitInBatches`, `filter`, `wait`,
  `waitForApproval`, `stopAndError`, `noOp`.
- **Data:** `set`, `aggregate`, `sort`, `limit`, `removeDuplicates`.
- **I/O:** `code`, `httpRequest`, `executeWorkflow`. **Utility:** `dateTime`, `crypto`.
- For `if`/`guardrails`/`splitInBatches`/`waitForApproval`, **outputs 0 and 1 are the
  two branches.** Edges connect nodes by **name**.

## Steps
1. Map the path: trigger → steps → branches → terminal(s). Keep it deterministic;
   add a `stopAndError` / error path; gate risky actions with `waitForApproval`.
2. Write `lyzr/workflows/<slug>.json`:
   ```json
   {
     "_type": "workflows",
     "flow_name": "Lead Triage",
     "flow_data": {
       "nodes": [
         {"id":"t","name":"Trigger","type":"lyzr-nodes-base.trigger","position":{"x":0,"y":0}},
         {"id":"c","name":"Classify","type":"lyzr-nodes-base.lyzr.agent","parameters":{}},
         {"id":"r","name":"Route","type":"lyzr-nodes-base.if","parameters":{}}
       ],
       "connections": {"Trigger":{"main":[[{"node":"Classify"}]]},
                       "Classify":{"main":[[{"node":"Route"}]]}}
     }
   }
   ```
3. (Optional) validate the node/edge graph with `create_workflow`.
4. Push with `lyzr_sync` → created on the platform (`/v3/workflows`). Report the flow.
