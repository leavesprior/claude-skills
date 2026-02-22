---
description: Debug Neoma issues with diagnostic commands and troubleshooting
---

# Neoma Debugging & Troubleshooting

Comprehensive debugging tools for diagnosing and resolving Neoma system issues.

## Quick Diagnostics

Run comprehensive verification:
```bash
neoma-verify
```

Check system status:
```bash
neoma status
```

## Port Checks

Verify which ports are listening:
```bash
ss -ltnp | grep -E '8120|8115|8121|8002|8116|3001|3002|9121'
```

Or use Neoma's port checker:
```bash
for p in 8120 8115 8121 8002 8116 3001 3002; do
  printf "Port %s: " "$p"
  curl -fsS --max-time 2 "http://127.0.0.1:$p/health" && echo "✓ UP" || echo "✗ DOWN"
done
```

## Service Logs

Check systemd service logs:
```bash
# Memory bridge logs
journalctl --user -u neoma-memory-bridge.service -n 100 --no-pager

# Serena service logs
journalctl --user -u serena.service -n 100 --no-pager

# All Neoma services
journalctl --user -u 'neoma-*' -n 50 --no-pager
```

## Agent-Specific Debugging

```bash
# Get detailed agent info
neoma-agents info <agent-name>

# Test specific agent with verbose output
FAST=0 neoma-agents test <agent-name> --deep

# Follow agent logs in real-time
neoma-agents logs <agent-name> --follow
```

## Common Issues & Solutions

### Issue: Services Won't Start

**Check:**
```bash
systemctl --user status neoma-memory-bridge.service
systemctl --user status serena.service
```

**Fix:**
```bash
systemctl --user restart neoma-memory-bridge.service
systemctl --user restart serena.service
```

### Issue: Port Already in Use

**Check what's using the port:**
```bash
ss -ltnp | grep :8121
lsof -i :8121
```

**Kill the process:**
```bash
kill $(lsof -t -i:8121)
```

### Issue: MCP Bridge Not Responding

**Check MCP health:**
```bash
curl -v http://127.0.0.1:8120/health
curl -v http://127.0.0.1:9121/tools/call
```

**Restart MCP services:**
```bash
neoma stop
sleep 2
neoma start
```

### Issue: Ollama Not Responding

**Check Ollama:**
```bash
curl http://127.0.0.1:11434/api/tags
ollama list
```

**Restart Ollama:**
```bash
systemctl --user restart ollama
```

## Health Check Scripts

Run smoke tests:
```bash
neoma smoke              # Quick smoke test
neoma-smoke-v2           # Enhanced smoke test
neoma-daily-smoke        # Comprehensive daily test
```

## Memory & Performance

Check memory usage:
```bash
ps aux | grep -E 'neoma|serena|ollama' | grep -v grep
```

## Network Diagnostics

Check connectivity:
```bash
# Test local endpoints
for port in 8120 8115 8121; do
  curl -I http://127.0.0.1:$port/health 2>&1 | head -1
done
```

## Configuration Issues

Check configuration files:
```bash
# MCP configuration
cat ~/.config/claude/mcp-servers.json

# Neoma environment
cat ~/.config/neoma/ports.env
```

## Emergency Reset

If all else fails, complete reset:
```bash
neoma stop
sleep 5
killall -9 node python3 ollama || true
sleep 2
neoma start
sleep 5
neoma-verify
```

## Get Help

After trying diagnostics, provide this information:
1. Output of `neoma-verify`
2. Output of `neoma status`
3. Relevant logs from `journalctl --user -u neoma-*`
4. Port status from `ss -ltnp`

Now proceed to help debug the specific issue.
