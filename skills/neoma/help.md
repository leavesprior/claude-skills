---
description: Show all available Claude Code slash commands and skills
---

# Available Claude Code Skills

Quick reference for all custom slash commands available in this environment.

## System Management

### `/suspend`
Schedule or execute system suspend with deep sleep (CPU+GPU+monitors)
- Supports multiple time formats: `23:00`, `2300`, `@ 23:00`
- Pre-suspend health checks and snapshots
- 3-second cancellation window
- PID tracking for easy cancellation

### `/suspend-help`
Detailed help and examples for the suspend command

## Neoma Multi-Agent Framework

### `/neoma-overview`
Comprehensive overview of the Neoma consciousness framework
- System architecture and components
- Philosophical framework and principles
- Key endpoints and quick commands
- Available agents and testing framework

### `/neoma-health`
Check Neoma system health and status
- Quick status checks (`neoma status`)
- Comprehensive verification (`neoma-verify`)
- Agent-specific checks
- Service port monitoring

### `/neoma-start`
Start, stop, or restart Neoma services
- Service management commands
- Startup troubleshooting
- Agent-specific operations
- Smoke testing procedures

### `/neoma-agents`
Manage and interact with Neoma agents
- List all agents
- Get agent information
- Test agents (fast and deep modes)
- View agent logs in real-time
- Register/unregister agents

### `/neoma-debug`
Debug and troubleshoot Neoma issues
- Quick diagnostics
- Port checks
- Service logs
- Common issues and solutions
- Emergency reset procedures

## Memory Management

### `/remember`
Save information to Neoma memory with automatic categorization
- Automatic topic detection and categorization
- Structured storage with metadata
- Support for notes, flags, and cases
- Integration with Memory Bridge (port 8115)
- Multiple namespace categories (system, code, tasks, ideas, fixes)

### `/recall`
Retrieve and search information from Neoma memory
- Search by keyword, topic, or specific key
- Query across multiple namespaces
- Filter by metadata and timestamps
- Smart search with context awareness
- Integration with Obsidian notes

## Usage Examples

```bash
# In Claude Code, just type the command:
/suspend 23:00
/neoma-health
/neoma-agents
/neoma-debug
```

## Command Syntax

All commands support natural language interaction. You can also:
- Ask questions: "check neoma health"
- Request actions: "suspend at 11pm"
- Get help: "how do I debug neoma?"

## Quick Reference

**Suspend System:**
- Immediate: `/suspend`
- Scheduled: `/suspend 23:00`
- Help: `/suspend-help`

**Neoma Operations:**
- Overview: `/neoma-overview`
- Check health: `/neoma-health`
- Start/stop: `/neoma-start`
- Manage agents: `/neoma-agents`
- Debug issues: `/neoma-debug`

## Creating Custom Commands

Slash commands are stored in `~/.claude/commands/` as Markdown files.

Format:
```markdown
---
description: Brief description of the command
---

# Command Title

Your command instructions here...
```

## Tips

1. Commands are context-aware - they understand your environment
2. You can chain operations naturally
3. Ask follow-up questions for clarification
4. Commands integrate with your existing tools

## Getting More Help

- Type `/help` to see this menu
- Ask "what can you do?" for capabilities
- Request specific command help: "/suspend-help"

---

**Total Commands Available:** 10 custom skills + system commands

Use these commands to efficiently manage your system and Neoma framework!
