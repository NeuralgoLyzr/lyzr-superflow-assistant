---
name: build-superflow
description: How to author a deterministic SuperFlow workflow and push it to the platform.
when_to_use: When building a SuperFlow / workflow — a repeatable, multi-step process with a known path.
---

# Build a SuperFlow

A SuperFlow is a **deterministic** workflow run by a durable engine — same path
every time. (If you'd need to reason per-request about which workers to call, it's
a *managerial agent*, not a SuperFlow.)

## The schema — `tasks` + `edges` (NOT nodes/connections)
The platform stores a workflow as `flow_data.tasks` + `flow_data.edges`. This is
what `/v3/workflows` accepts AND what the studio's SuperFlow viewer renders. To see
the exact shape, pull an existing one: `lyzr_pull types:["workflows"]` →
`lyzr/workflows/*.json`. Author your file the same way:

```json
{
  "_type": "workflows",
  "flow_name": "Lead Triage",
  "flow_data": {
    "flow_name": "Lead Triage",
    "run_name": "lead-triage",
    "default_inputs": { "value": "<a sample input>" },
    "tasks": [
      { "name": "inputs", "function": "default_inputs", "params": {} },
      { "name": "agent_classify", "function": "call_lyzr_agent",
        "params": { "config": { "agent_id": "<real agent_id>", "agent_name": "Classifier" } } },
      { "name": "route", "function": "gpt_conditional_block",
        "params": { "config": { "conditions": [{ "expression": "urgency == 'high'" }] } } }
    ],
    "edges": [
      { "source": "inputs",         "target": "agent_classify", "condition": "true" },
      { "source": "agent_classify", "target": "route",          "condition": "true" },
      { "source": "route",          "target": "agent_escalate", "condition": "true" },
      { "source": "route",          "target": "agent_reply",    "condition": "false" }
    ]
  }
}
```

## Task `function` values
- `default_inputs` — the entry/inputs node.
- `call_lyzr_agent` — an agent step; `params.config.agent_id` + `agent_name`. **The
  agent must exist first** (a real `agent_id`): delegate to the **agent-creator**, or
  author `lyzr/agents/<name>.json` and `lyzr_sync` to get ids back, THEN reference them.
- `gpt_conditional_block` — a branch; its out-edges carry `condition` (`"true"`/`"false"`
  or an expression). `gpt_router` — multi-way routing. `a2a` — agent-to-agent.

## Steps
1. Decide the deterministic path: inputs → steps → branch → terminal(s).
2. Ensure every agent step references a **real agent_id** (create the agents first).
3. Write `lyzr/workflows/<slug>.json` using the `tasks` + `edges` schema above
   (edges reference tasks by `name`).
4. (Optional) validate with `create_workflow`.
5. Push with `lyzr_sync` → created on `/v3/workflows`, and it renders in the studio's
   SuperFlow viewer.

Always use `tasks`/`edges` — the `nodes`/`connections` shape will neither sync nor render.
