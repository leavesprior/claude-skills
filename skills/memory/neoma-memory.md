# Neoma Memory Bridge

## Overview
Neoma Memory Bridge is the persistent storage backbone of the Neoma multi-agent system. It provides a simple HTTP API for storing and retrieving structured data across sessions and agents.

## Core Capabilities
- **Persistent Storage**: Data survives across conversations and system reboots
- **Namespace Organization**: Organize data into logical categories
- **JSON-based**: Structured data storage with full object support
- **Multi-Agent Access**: Shared memory accessible by all Neoma agents
- **Simple HTTP API**: Easy integration via curl or any HTTP client

## API Endpoints

### 1. Store Data
**POST** `http://127.0.0.1:8115/store`

Store a key-value pair in a namespace.

**Request Body:**
```json
{
  "namespace": "system_config",
  "key": "my_setting",
  "value": {
    "enabled": true,
    "version": "1.0",
    "created": "2025-10-25"
  }
}
```

**Response:**
```json
{
  "ok": true,
  "path": "/home/granny/Neoma_project/memory/system_config/my_setting.json"
}
```

### 2. Retrieve Data
**GET** `http://127.0.0.1:8115/retrieve/{namespace}/{key}`

Retrieve stored data by namespace and key.

**Example:**
```bash
curl http://127.0.0.1:8115/retrieve/system_config/my_setting
```

**Response:** The stored JSON object

### 3. List Keys in Namespace
**GET** `http://127.0.0.1:8115/list/{namespace}`

List all keys stored in a namespace.

**Example:**
```bash
curl http://127.0.0.1:8115/list/system_config
```

**Response:**
```json
["my_setting", "another_config", "serena_mcp_http_bridge"]
```

### 4. Health Check
**GET** `http://127.0.0.1:8115/health`

Check if memory bridge is running.

**Response:**
```json
{
  "status": "healthy",
  "service": "neoma-memory-bridge",
  "port": 8115
}
```

## Common Namespaces

### system_config
System and agent configurations, integration settings, service endpoints

**Examples:**
- `serena_mcp_http_bridge` - Serena MCP integration config
- `claude_flow_agent` - Claude-Flow orchestration settings
- `discord_fixes` - Discord compatibility configurations

### preferences
User preferences and persistent settings

**Examples:**
- `save_memory_instructions` - Memory save behavior
- `claude_flow_usage` - Token optimization preferences
- `ui_theme` - User interface preferences

### tasks
Pending tasks and work items

**Examples:**
- `pending_serena_mcp_fix` - Unfinished Serena work
- `file_picker_investigation` - Ongoing troubleshooting

### fixes
Applied fixes and solutions for reference

**Examples:**
- `nautilus_dark_theme_fix` - File picker theme solution
- `discord_epipe_fix` - Discord error resolution

### skills
Skill configurations and metadata

**Examples:**
- `serena_skill_integration` - Serena skill setup
- `neoma_memory_skill` - This skill's configuration

## Usage in Claude Code

I use Neoma Memory throughout our conversations to:

1. **Save configurations**: When we set up integrations (Serena, Claude-Flow)
2. **Remember fixes**: Document solutions for future reference
3. **Track tasks**: Mark work as pending for later completion
4. **Store preferences**: Remember your choices and settings
5. **Share state**: Enable multi-agent coordination

## Storage Location

All data is stored as JSON files in:
```
~/Neoma_project/memory/
├── system_config/
│   ├── serena_mcp_http_bridge.json
│   ├── claude_flow_agent.json
│   └── ...
├── preferences/
│   ├── save_memory_instructions.json
│   └── ...
├── tasks/
│   └── ...
└── fixes/
    └── ...
```

## Best Practices

1. **Use Descriptive Keys**: Choose clear, meaningful key names
   - Good: `serena_mcp_http_bridge`
   - Bad: `config1`

2. **Include Metadata**: Add timestamps, versions, descriptions
   ```json
   {
     "created": "2025-10-25",
     "version": "1.0.0",
     "description": "Serena MCP integration settings",
     "data": { ... }
   }
   ```

3. **Organize by Namespace**: Group related data logically
   - `system_config` for infrastructure
   - `preferences` for user choices
   - `tasks` for work items

4. **Document Important Configs**: Add `description` fields for clarity

## Multi-Agent Integration

Neoma Memory is accessible by all Neoma agents:
- cc_agent (Claude Code) - Me!
- oc_agent (Codex)
- serena (Web scraping)
- essence_weaver (Metacognitive processing)
- turing_infuser (Turing machine operations)

This enables coordination and state sharing across the entire Neoma ecosystem.

## Technical Details

**Port:** 8115
**Protocol:** HTTP REST API
**Format:** JSON
**Storage:** File-based (JSON files per key)
**Access:** localhost only (127.0.0.1)
**Status:** Core infrastructure - always running

## Example Session

```bash
# Store a configuration
curl -X POST http://127.0.0.1:8115/store \
  -H 'Content-Type: application/json' \
  -d '{
    "namespace": "preferences",
    "key": "my_app_settings",
    "value": {
      "theme": "dark",
      "auto_save": true,
      "version": "2.1"
    }
  }'

# List all preferences
curl http://127.0.0.1:8115/list/preferences

# Retrieve the settings
curl http://127.0.0.1:8115/retrieve/preferences/my_app_settings

# Check memory bridge health
curl http://127.0.0.1:8115/health
```

## When to Use Neoma Memory

✅ **DO use for:**
- Saving important configurations
- Documenting applied fixes
- Tracking multi-session work
- Storing user preferences
- Building knowledge base
- Agent coordination state

❌ **DON'T use for:**
- Temporary data (use variables)
- Large files (use filesystem)
- Binary data (JSON only)
- Real-time streaming (use other agents)
- Secrets (no encryption)

Neoma Memory is your persistent brain - use it to remember everything important across conversations!
