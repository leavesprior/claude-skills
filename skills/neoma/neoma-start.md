---
description: Start or restart Neoma system services
---

# Neoma Service Management

Start, stop, or restart Neoma system services and agents.

## Start All Services

```bash
neoma start
```

This will:
- Enable and start neoma-memory-bridge.service
- Restart serena.service and serena-bridge-worker.service
- Start Serena HTTP bridge if not running (port 8121)

## Stop All Services

```bash
neoma stop
```

This will:
- Stop serena-bridge-worker.service and serena.service
- Stop neoma-memory-bridge.service
- Stop neoma-mb-forwarder.service
- Stop neoma-mb-proxy-8119.service

## Verify Services Are Running

After starting, verify with:

```bash
neoma status
```

Or run comprehensive checks:

```bash
neoma-verify
```

## Common Startup Issues

### Port Already in Use

Check if ports are occupied:
```bash
ss -ltnp | grep -E '8120|8115|8121|9121'
```

### Service Won't Start

Check systemd logs:
```bash
journalctl --user -u neoma-memory-bridge.service -n 50
journalctl --user -u serena.service -n 50
```

### Memory Bridge Issues

Restart the memory bridge:
```bash
systemctl --user restart neoma-memory-bridge.service
```

## Agent-Specific Operations

For individual agent management:

```bash
# Check agent status
neoma-agents info <agent-name>

# Test agent
neoma-agents test <agent-name>

# View agent logs
neoma-agents logs <agent-name> -n 50 --follow
```

## Smoke Testing

After starting services, run smoke tests:

```bash
neoma smoke         # Quick validation
neoma-smoke-v2      # Enhanced smoke test
neoma-daily-smoke   # Comprehensive daily validation
```

## Background Services

Neoma uses systemd user services. Check them with:

```bash
systemctl --user list-units 'neoma-*' 'serena-*'
```

Now proceed to execute the appropriate service management command.
