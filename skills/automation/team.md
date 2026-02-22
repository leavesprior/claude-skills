---
description: Coordinate multi-agent teams (spawn parallel agents, use templates, check status)
---

# Agent Teams

Coordinate multiple agents (cc, gem, oc) working together on tasks.

The user's input is: $ARGUMENTS

## Interpreting the Request

Parse the user's input and determine which team operation to perform:

### Quick Patterns

- `/team review <target>` — Invoke the review-team template with target variable
- `/team research "<topic>"` — Invoke the research-team template with topic variable
- `/team refactor <target> "<instructions>"` — Invoke the refactor-team template
- `/team debug "<issue>"` — Invoke the debug-team template
- `/team spawn cc:<task1> gem:<task2> [oc:<task3>]` — Ad-hoc parallel team
- `/team status [team_id]` — Show status of active teams
- `/team harvest <team_id>` — Collect results from a team
- `/team templates` — List available templates

## Executing the Operation

### For template invocations (review, research, refactor, debug):

Run via CLI:
```bash
neoma-agent-team template invoke <template-name> --var key=value [--var key2=value2]
```

### For ad-hoc spawn (spawn cc:task gem:task):

Parse the agent:task pairs and build a JSON members array, then:
```bash
neoma-agent-team spawn --name "<name>" --members '<json_array>'
```

Each member needs: `{"role": "<agent_prefix>", "agent": "<agent>_agent", "task": "<task>"}`

### For status:

```bash
neoma-agent-team status <team_id>
```

If no team_id given, list all teams:
```bash
neoma-agent-team list
```

### For harvest:

```bash
neoma-agent-team harvest <team_id>
```

### For templates:

```bash
neoma-agent-team template list
```

## Response Format

After running the command, report:
1. What operation was performed
2. The team_id (if spawned)
3. Number of members and their roles
4. Any conflicts detected (members that will run sequentially)
5. How to check status: `neoma-agent-team status <team_id>` or `/team status <team_id>`
