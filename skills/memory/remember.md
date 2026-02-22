---
description: Save information to Neoma memory with automatic topic categorization
---

# Remember - Neoma Memory Management

Save important information to Neoma's memory system with automatic categorization by topic.

## Quick Save

When the user wants to remember something, I will:

1. **Extract the information** from the conversation
2. **Identify the topic/category** (e.g., system, code, config, task, idea, bug, fix)
3. **Generate a unique key** based on the content
4. **Save to Neoma memory** with proper namespace
5. **Confirm storage** and provide retrieval instructions

## Memory Storage Formats

### Simple Notes (neoma-note)
For quick key-value storage:

```bash
# Save a note
neoma-note put <namespace> <key> '<json_value>'

# Retrieve a note
neoma-note get <namespace> <key>
```

### Flags (neoma-flag)
For state tracking with success counters:

```bash
# Set a flag with status
neoma-flag set <scope.name> <status> "<reason>"

# Get flag status
neoma-flag get <scope.name>

# Promote flag after 3 successive greens
neoma-flag promote <scope.name>
```

### Cases (neoma-case)
For task/workflow tracking:

```bash
# Create a case
neoma-case create <case-id> <agent> <status> "<note>"

# Update case
neoma-case update <case-id> <agent> <status> "<note>"

# Route case
neoma-case route <case-id>
```

## Topic Categories (Namespaces)

I will automatically categorize information into these namespaces:

**Technical:**
- `system_config` - System configuration and settings
- `code_snippets` - Useful code examples and patterns
- `commands` - Shell commands and scripts
- `debug_notes` - Debugging information and solutions
- `api_keys` - API endpoints and keys (sanitized)

**Tasks & Projects:**
- `tasks` - TODO items and task tracking
- `ideas` - Future ideas and improvements
- `bugs` - Bug reports and issues
- `fixes` - Solutions and fixes applied

**Knowledge:**
- `learning` - Things learned during sessions
- `documentation` - Important documentation references
- `decisions` - Technical decisions and rationale
- `best_practices` - Patterns and best practices

**Personal:**
- `preferences` - User preferences and settings
- `shortcuts` - Personal shortcuts and aliases
- `notes` - General notes and reminders

## Example Usage

### User Says: "Remember that the suspend script is at /home/granny/bin/ssuspend"

I will execute:
```bash
neoma-note put system_config suspend_script_location '{
  "path": "/home/granny/bin/ssuspend",
  "description": "Enhanced suspend script with time scheduling",
  "features": ["deep_sleep", "timed_scheduling", "pre_suspend_checks"],
  "timestamp": 1761144660,
  "category": "system"
}'
```

### User Says: "Remember that port 8121 is for Serena"

I will execute:
```bash
neoma-note put system_config port_8121 '{
  "service": "Serena",
  "port": 8121,
  "description": "Web scraping and fetching agent",
  "health_check": "http://127.0.0.1:8121/health",
  "category": "neoma_agents"
}'
```

### User Says: "Remember I fixed the GPU suspend issue by setting deep sleep mode"

I will execute:
```bash
neoma-note put fixes gpu_suspend_fix '{
  "issue": "GPU not suspending properly",
  "solution": "Set deep sleep mode via /sys/power/mem_sleep",
  "command": "echo deep | sudo tee /sys/power/mem_sleep",
  "date": "2025-10-22",
  "status": "fixed",
  "category": "system_fixes"
}'
```

### User Says: "Remember to test the new suspend feature tomorrow"

I will execute:
```bash
neoma-case create test_suspend_2025-10-23 cc_agent unknown "Test new suspend timing feature with ssuspend @ 23:00"
```

## Retrieving Memories

### By Namespace
```bash
# Get specific memory
neoma-note get system_config suspend_script_location

# Get all memories in a namespace (requires listing support)
curl -fsS http://127.0.0.1:8115/list?namespace=system_config
```

### By Flag Status
```bash
neoma-flag get agent.serena.health
```

### By Case ID
```bash
neoma-case update test_suspend_2025-10-23 cc_agent green "Test passed"
```

## Search Memories

For searching across memories, I can:
1. Query multiple namespaces
2. Filter by category/topic
3. Search by timestamp range
4. Find related entries

## Memory Structure

Each memory includes:
- **Content**: The actual information
- **Metadata**: Category, timestamp, tags
- **Context**: Related information, agent, session
- **Retrieval**: Key for future access

## Integration with Obsidian

For richer note-taking, memories can also be saved to Obsidian:
```bash
# Save to Obsidian vault
obsidian-mcp-advanced put <vault> <note-name> '{...}'
```

## Status Codes

**Flags:**
- `green` - Working/successful
- `yellow` - Needs attention
- `red` - Failed/error
- `unknown` - Not yet tested

## Memory Bridge

All operations go through Neoma Memory Bridge at port 8115:
- Endpoint: `http://127.0.0.1:8115`
- Health: `http://127.0.0.1:8115/health`
- Store: `POST /store`
- Retrieve: `GET /retrieve`

## Best Practices

1. **Use descriptive keys** - Make them searchable and meaningful
2. **Include timestamps** - Always add creation/update time
3. **Add categories** - Help with organization and retrieval
4. **Rich metadata** - Include context for future reference
5. **Status tracking** - Use flags for state management

## Security

- Sanitize sensitive data before storage
- Use appropriate namespaces for access control
- Never store plain passwords or tokens
- Consider encryption for sensitive notes

---

**Ready to remember!** Just tell me what you want to save, and I'll categorize and store it in Neoma's memory system.
