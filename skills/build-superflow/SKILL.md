---
name: build-superflow
description: How to author a deterministic SuperFlow ({nodes, connections}) the studio's SuperFlow canvas renders.
when_to_use: When building a SuperFlow / workflow — a repeatable, multi-step process with a known path.
---

# Build a SuperFlow

A SuperFlow is a **deterministic** workflow run by a durable engine — same path
every time. The studio's SuperFlow canvas renders `flow_data` as
**`{ nodes, connections }`** (node `type` = `lyzr-nodes-base.*`), so author exactly
that shape — NOT `tasks/edges`.

## Node vocabulary (`lyzr-nodes-base.*`)
- **Start:** exactly one `trigger`.
- **AI:** `lyzr.agent` (a Lyzr agent step), `lyzr.llm`, `lyzr.tool`, `lyzr.a2aAgent`,
  `lyzr.taskDecomposition`. **Safety:** `lyzr.guardrails`.
- **Control:** `if`, `switch`, `merge`, `filter`, `wait`, `waitForApproval`,
  `stopAndError`, `noOp`. **Data:** `set`, `aggregate`, `sort`, `limit`.
- For `if`/`switch`/`guardrails`/`waitForApproval`, **output index 0 and 1 are the
  branches** (`connections[name].main[0]` = first branch, `main[1]` = second).

## The schema
```json
{
  "_type": "workflows",
  "flow_name": "Loan Approval",
  "flow_data": {
    "nodes": [
      {"id":"trigger","name":"New Application","type":"lyzr-nodes-base.trigger","position":[0,300],"parameters":{}},
      {"id":"intake","name":"Intake & Validate","type":"lyzr-nodes-base.lyzr.agent","position":[260,300],
       "parameters":{"agent_id":"<real agent_id>","agent_name":"Intake Validator"}},
      {"id":"gate","name":"Valid?","type":"lyzr-nodes-base.if","position":[520,300],"parameters":{}},
      {"id":"reject","name":"Reject","type":"lyzr-nodes-base.stopAndError","position":[780,460],"parameters":{}},
      {"id":"approve","name":"Approve","type":"lyzr-nodes-base.lyzr.agent","position":[780,200],
       "parameters":{"agent_id":"<real agent_id>","agent_name":"Approver"}}
    ],
    "connections": {
      "New Application": { "main": [[{"node":"Intake & Validate","type":"main","index":0}]] },
      "Intake & Validate": { "main": [[{"node":"Valid?","type":"main","index":0}]] },
      "Valid?": { "main": [ [{"node":"Approve","type":"main","index":0}], [{"node":"Reject","type":"main","index":0}] ] }
    }
  }
}
```

## Rules
- **`nodes`**: each `{ id, name, type:"lyzr-nodes-base.*", position:[x,y], parameters }`.
  `position` is an **`[x, y]` array**; lay the flow out left→right (x += ~260) and offset
  branches on y. Exactly one `trigger`.
- **`lyzr.agent` nodes** carry `parameters.agent_id` + `parameters.agent_name` — the
  agent must EXIST first (create it via the agent-creator, or author `lyzr/agents/<name>.json`
  + `lyzr_sync` to get a real id), THEN reference that id.
- **`connections`** are keyed by source node **name**; `main` is an array of output
  branches, each a list of `{ "node": "<target name>", "type": "main", "index": 0 }`
  (always include `type` + `index` so the edge connects). For an `if`, `main[0]` is the
  true branch and `main[1]` the false branch.
- Keep it deterministic: add a `stopAndError` path; gate risky actions with `waitForApproval`.

## Steps
1. Ensure each `lyzr.agent` step references a **real agent_id** (create the agents first).
2. Write `lyzr/workflows/<slug>.json` in the `{nodes, connections}` schema above.
3. (Optional) validate with `create_workflow`.
4. Push with `lyzr_sync`.

## It renders automatically — don't send the user away
The moment you **Write `lyzr/workflows/<name>.json`** it renders LIVE in the studio's
**SuperFlow** tab on the right (the file write is the trigger — so always Write the full
file, not only call the API). When done, say it's ready and point the user to the
**SuperFlow tab on the right**. NEVER tell them to open the external/cloud Lyzr studio or
navigate to Workflows manually — it's already on their screen. If it isn't showing,
re-Write the workflow file (that re-triggers the render).
