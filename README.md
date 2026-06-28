# lyzr-superflow-agent

An [OpenGAP](https://github.com/open-gitagent/opengap) sub-agent (spec 0.1.0): **this repo IS the `superflow` agent**. Loaded by the Lyzr harness as a named sub-agent (`subagent_type: superflow`) — its `SOUL.md` is the system prompt, `agent.yaml` scopes its tools/model, and `skills/` are its playbooks.

- model: `high`
- tools: create_workflow, lyzr_pull, lyzr_sync, Read, Write, Edit, Glob, Grep, Skill
- skill: `build-superflow`
