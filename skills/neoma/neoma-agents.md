---
description: Manage and interact with Neoma agents (list, info, test, logs)
---

# Neoma Agent Management

Interact with, test, and monitor Neoma's multi-agent system.

## List All Agents

```bash
neoma-agents list
```

Shows all available agents in the Neoma ecosystem.

## Get Agent Information

```bash
neoma-agents info <agent-name>
```

Examples:
```bash
neoma-agents info mcp        # MCP Bridge info
neoma-agents info serena     # Serena web agent info
neoma-agents info cc         # Claude Code agent info
neoma-agents info ollama     # Ollama LLM info
```

## Test Agents

Test a single agent:
```bash
neoma-agents test <agent-name>
```

Test all agents (fast mode):
```bash
neoma-agents test all
```

Deep testing (slower, more comprehensive):
```bash
FAST=0 neoma-agents test <agent-name> --deep
FAST=0 neoma-agents test all --deep
```

## View Agent Logs

```bash
# Last 50 lines
neoma-agents logs <agent-name> -n 50

# Follow logs in real-time
neoma-agents logs <agent-name> --follow

# Combine options
neoma-agents logs <agent-name> -n 100 --follow
```

## Available Agents

**Core Infrastructure:**
- **cc** - Claude Code agent (this assistant)
- **oc** - Codex agent
- **mcp** - Model Context Protocol bridge

**AI/LLM:**
- **ollama** - Local LLM server
- **autogpt** - Autonomous task agent
- **interpreter** - Open Interpreter for code execution
- **swarm** - Multi-agent swarm coordination

**Web & Data:**
- **serena** - Web scraping and fetching
- **scira** - Search and research agent
- **web** - Web interface
- **open-webui** - Web UI for LLMs

**Integrations:**
- **n8n** - Workflow automation
- **obsidian** - Note-taking integration
- **gsheets** - Google Sheets integration
- **clawdbot** - Multi-channel messaging gateway (Discord, Telegram)

**Monitoring:**
- **tsm** - Turing State Machine monitoring
- **ntch** - Network monitoring
- **bench** - Benchmarking
- **report** - Reporting system
- **bg** - Background tasks
- **whisper** - Audio transcription

## Agent Status Quick Reference

Check which agents are currently running:

```bash
neoma status          # Core services
neoma-verify          # Comprehensive check
neoma-agents test all # Test all agents
```

## Register New Agent

```bash
neoma-agent-register <name> <port> <tags>
```

Example:
```bash
neoma-agent-register my_agent 8123 "custom,task"
```

## Unregister Agent

```bash
neoma-agent-unregister <name>
```

## Agent Interaction

For safe, read-only agent operations:

```bash
neoma-agents run <agent> <action>
```

(Only safe actions are permitted through this interface)

Now proceed to help the user with their agent management task.
