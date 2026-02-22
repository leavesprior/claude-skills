---
description: Check Neoma system health and status across all agents
---

# Neoma Health Check

Check the health and operational status of all Neoma agents and services.

## Quick Status Check

Run a quick health check of core Neoma services:

```bash
neoma status
```

This checks:
- Port availability (8120, 8115, 8121, 9121)
- Health endpoints for each service
- HTTP MCP bridge status

## Comprehensive Verification

For a more detailed health check including all agents:

```bash
neoma-verify
```

This performs:
- Health checks on ports 8120, 8115, 8121
- Serena queue sanity test
- Wheelwright specs validation
- Optional Ollama capability test

**Timeout:** 20 seconds (configurable via `NEOMA_VERIFY_TIMEOUT`)

## Agent-Specific Checks

Check individual agents:

```bash
# List all agents
neoma-agents list

# Get info about a specific agent
neoma-agents info mcp
neoma-agents info serena

# Test a specific agent
neoma-agents test mcp
neoma-agents test all

# Deep testing (slower but comprehensive)
FAST=0 neoma-agents test all --deep
```

Available agents: cc, oc, mcp, tsm, ntch, ollama, n8n, obsidian, bench, report, bg, whisper, web, open-webui, gsheets, scira, autogpt, interpreter, swarm

## Key Services and Ports

- **Port 3001**: MCP Bridge - HTTP endpoints for agent coordination
- **Port 8002**: AutoGPT - Autonomous task processing
- **Port 8116**: Open Interpreter - Code execution
- **Port 8120**: Enhanced MCP - Extended Model Context Protocol
- **Port 8121**: Serena - Web scraping and fetching
- **Port 3002**: Self-Referential Extension - Recursive reflection

## Interpreting Results

**PASS**: Service is operational
**FAIL**: Service is down or not responding
**WARN**: Service responding but with issues

## Next Steps

If services are down, use:
```bash
/neoma-start    # Start all Neoma services
/neoma-restart  # Restart troubled services
```

Now execute the appropriate health check command based on user needs.
