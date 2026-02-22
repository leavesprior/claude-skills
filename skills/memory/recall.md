---
description: Retrieve and search information from Neoma memory
---

# Recall - Retrieve from Neoma Memory

Search and retrieve information previously saved to Neoma's memory system.

## Quick Recall

When the user wants to recall something, I will:

1. **Identify what they're looking for** (topic, keyword, or specific memory)
2. **Search appropriate namespaces** based on topic
3. **Retrieve and format** the information
4. **Present in readable format** with context

## Retrieval Methods

### By Specific Key

```bash
# Get a specific memory
neoma-note get <namespace> <key>
```

Example:
```bash
neoma-note get system_config suspend_script_location
neoma-note get fixes gpu_suspend_fix
neoma-note get code_snippets python_async_pattern
```

### By Namespace (Topic)

```bash
# List all memories in a namespace
curl -fsS http://127.0.0.1:8115/list?namespace=<namespace>
```

Available namespaces:
- `system_config` - System settings
- `code_snippets` - Code examples
- `commands` - Shell commands
- `debug_notes` - Debug info
- `tasks` - TODO items
- `ideas` - Future ideas
- `bugs` - Bug reports
- `fixes` - Applied solutions
- `learning` - Learned concepts
- `documentation` - Doc references
- `decisions` - Technical decisions
- `best_practices` - Patterns
- `preferences` - User settings
- `shortcuts` - Aliases
- `notes` - General notes

### By Flag Status

```bash
# Get flag status
neoma-flag get <scope.name>

# Check all agent health flags
for agent in serena scira ollama mcp; do
  echo "$agent: $(neoma-flag get agent.$agent.health)"
done
```

### By Case ID

```bash
# Get case details
neoma-note get neoma_flow "case_<case-id>"

# Update and retrieve
neoma-case update <case-id> <agent> <status> "<note>"
```

## Search Strategies

### Keyword Search

I will search across multiple namespaces for keywords:

```bash
# Search system-related memories
neoma-note get system_config suspend_script_location
neoma-note get system_config port_8121
neoma-note get system_config nvidia_power_mgmt

# Search fix-related memories
neoma-note get fixes gpu_suspend_fix
neoma-note get fixes port_conflict_resolution
```

### Topic-Based Search

Map user intent to relevant namespaces:

- **"What did I learn about..."** → `learning`
- **"How do I fix..."** → `fixes`, `debug_notes`
- **"What command..."** → `commands`, `code_snippets`
- **"Where is..."** → `system_config`, `documentation`
- **"What was that idea..."** → `ideas`
- **"Why did we decide..."** → `decisions`

### Time-Based Search

Filter by timestamp in JSON:
```bash
# Get recent memories (requires parsing timestamps)
neoma-note get <namespace> <key> | jq 'select(.timestamp > 1761000000)'
```

## Example Queries

### User: "What did I save about suspend?"

I will search:
```bash
# Check system config
neoma-note get system_config suspend_script_location

# Check commands
neoma-note get commands ssuspend_usage

# Check fixes
neoma-note get fixes suspend_deep_sleep_fix
```

### User: "Recall the Serena port"

```bash
neoma-note get system_config port_8121
```

Output:
```json
{
  "service": "Serena",
  "port": 8121,
  "description": "Web scraping and fetching agent",
  "health_check": "http://127.0.0.1:8121/health",
  "category": "neoma_agents"
}
```

### User: "Show me all my ideas"

```bash
# List all ideas namespace entries
curl -fsS http://127.0.0.1:8115/list?namespace=ideas | jq .
```

### User: "What fixes have I applied?"

```bash
# Get recent fixes
curl -fsS http://127.0.0.1:8115/list?namespace=fixes | jq .
```

## Advanced Retrieval

### Aggregate Across Namespaces

Search multiple related namespaces:
```bash
# System-related info
for ns in system_config commands fixes; do
  echo "=== $ns ==="
  curl -fsS "http://127.0.0.1:8115/list?namespace=$ns"
done
```

### Filter by Metadata

Use jq to filter results:
```bash
# Get all green flags
neoma-note get neoma_flags <key> | jq 'select(.status == "green")'

# Get recent memories (last hour)
NOW=$(date +%s)
neoma-note get <namespace> <key> | jq "select(.timestamp > $((NOW - 3600)))"
```

### Related Memories

Find related entries by category:
```bash
# Get all Neoma agent configs
neoma-note get system_config port_8121 | jq 'select(.category == "neoma_agents")'
```

## Memory Categories

When recalling, I'll search in priority order:

**For Configuration:**
1. `system_config`
2. `preferences`
3. `documentation`

**For Problems:**
1. `fixes`
2. `debug_notes`
3. `bugs`

**For How-To:**
1. `commands`
2. `code_snippets`
3. `best_practices`

**For Planning:**
1. `tasks`
2. `ideas`
3. `decisions`

## Display Format

I will present recalled information as:

1. **Summary** - Quick overview of what was found
2. **Details** - Full JSON or formatted content
3. **Metadata** - When saved, category, tags
4. **Context** - Related information
5. **Actions** - Suggested next steps

## Special Queries

### Health Status

```bash
# Check agent health flags
neoma-flag get agent.serena.health
neoma-flag get agent.mcp.health
neoma-flag get agent.ollama.health
```

### Task Status

```bash
# Get task status
neoma-case update <task-id> cc_agent green "Completed"
```

### System State

```bash
# Get critical system info
neoma-note get system_config ports_mapping
neoma-note get system_config service_status
```

## Empty Results

If no memory found, I will:
1. Confirm the search was understood
2. Suggest similar keys or namespaces
3. Offer to search broader categories
4. Ask if user wants to create this memory

## Memory Bridge API

Direct API access for advanced queries:

```bash
# Health check
curl http://127.0.0.1:8115/health

# Store
curl -X POST http://127.0.0.1:8115/store \
  -H 'Content-Type: application/json' \
  -d '{"namespace":"test","key":"foo","value":{"data":"bar"}}'

# Retrieve
curl "http://127.0.0.1:8115/retrieve?namespace=test&key=foo"

# List namespace
curl "http://127.0.0.1:8115/list?namespace=test"
```

## Integration with Other Tools

### Obsidian Search

```bash
# Search Obsidian notes
mcp__smithery-obsidian__search_notes "query string"
```

### File System Search

```bash
# Search for related files
grep -r "keyword" ~/.local/share/neoma/
```

## Tips for Better Recall

1. **Use specific keywords** - More specific = better results
2. **Know your categories** - Faster if you know the namespace
3. **Time context helps** - "Recently saved" vs "saved last week"
4. **Related searches** - Ask for similar or related items
5. **Partial matches** - I can search for partial key matches

---

**Ready to recall!** Just ask what you want to retrieve, and I'll search Neoma's memory system to find it.
