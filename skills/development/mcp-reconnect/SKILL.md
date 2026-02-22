# MCP Reconnection Troubleshooting

Diagnose and fix failed MCP server connections for Neoma.

## Architecture

Claude Code uses **newline-delimited JSON-RPC** over stdio for MCP servers.
Connection timeout: **30 seconds** (override with `MCP_TIMEOUT` env var).
Claude Code runs as **root** (`HOME=/root`). Wrapper scripts must account for this.

**Config locations (checked in order, user-scoped wins):**
1. **User-scoped (active):** `/root/.claude.json` → `mcpServers` key
2. **Project-scoped (backup):** `/home/granny/.config/claude/mcp-servers.json` → `mcpServers` key

User-scoped overrides project-scoped. Use `claude mcp get <name>` to check which scope a server is in.

## Step 0: Retrieve Known-Good Configs

Before diagnosing anything, check the memory bridge for the last working state:

```bash
curl -s "http://localhost:8115/retrieve?namespace=mcp&key=repair_procedure" | python3 -m json.tool
```

This contains verified-working commands, args, and tool counts for each server.

## Step 1: Read Actual Config

```bash
# User-scoped (takes precedence):
python3 -c "import json; d=json.load(open('/root/.claude.json')); [print(f'{k}: {v.get(\"command\",\"\")}') for k,v in d.get('mcpServers',{}).items()]"

# Project-scoped (fallback):
cat /home/granny/.config/claude/mcp-servers.json | python3 -m json.tool
```

Do NOT hardcode a server list. Always read the config to see what's actually configured.

## Step 2: Check Current MCP Status

```bash
claude mcp list 2>/dev/null
```

Note: this shows the *current session's* connections. Config changes require a session restart.

## Step 3: Trace Each Failed Server

For each failed server, follow this diagnostic chain:

### 3a. Verify the binary exists and is the right thing

```bash
# Does the command exist?
which <command> 2>/dev/null || ls -la <command>

# What IS it? (script? symlink? binary?)
file <command>

# If symlink, where does it point?
readlink -f <command>

# What does it actually do?
<command> --help 2>&1 | head -5
```

**Critical distinction:**
- **Stdio MCP server**: Reads JSON-RPC on stdin, writes responses on stdout
- **HTTP server**: Binds a port, serves HTTP. Cannot be an MCP stdio server.
- **CLI wrapper**: A convenience script (e.g. `serena health`). Not an MCP server.
- **Launcher shim**: Tries uvx/uv/python fallback chain. May fail if none are available.

### 3b. Trace wrapper chains

Many commands are bash wrappers that `exec` into something else. Read the script:

```bash
cat <command>
```

If the wrapper tries multiple paths (uvx -> uv -> python3), test each fallback:

```bash
# Check which tools are available
which uvx uv python3 2>/dev/null

# If wrapper uses a venv, check if it has the right packages
<venv>/bin/python -c "import <expected_module>; print('OK')"
```

### 3c. Test MCP handshake

Only after confirming the binary is correct:

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1"}}}' \
  | timeout 10 <COMMAND> <ARGS> 2>/dev/null \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'{d[\"result\"][\"serverInfo\"][\"name\"]} v{d[\"result\"][\"serverInfo\"][\"version\"]}')"
```

If this produces valid JSON with `serverInfo`, the server works. If it produces nothing or errors, the binary is wrong.

### 3d. Node.js spawn test (how Claude Code actually launches)

```bash
node -e "
const { spawn } = require('child_process');
const p = spawn('<COMMAND>', [<ARGS>], {
  env: Object.assign({}, process.env, {<ENV>}),
  stdio: ['pipe','pipe','pipe'], shell: false
});
p.on('error', e => console.error('SPAWN ERROR:', e.message));
p.on('spawn', () => {
  p.stdin.write(JSON.stringify({jsonrpc:'2.0',id:1,method:'initialize',params:{protocolVersion:'2024-11-05',capabilities:{},clientInfo:{name:'test',version:'1'}}}) + '\n');
  p.stdout.on('data', d => { console.log('OK:', d.toString().trim().slice(0,200)); p.kill(); });
  p.stderr.on('data', d => console.error('STDERR:', d.toString().trim().slice(0,200)));
  setTimeout(() => { console.log('TIMEOUT'); p.kill(); }, 10000);
});
"
```

## MCP Server Reference (7 servers as of 2026-02-05)

### Tier 1: npx-based (reliable, auto-install)

| Server | Wrapper | npx Package | Notes |
|--------|---------|-------------|-------|
| filesystem | `/home/granny/bin/mcp-filesystem.sh` | `@modelcontextprotocol/server-filesystem` | Roots: /home/granny, Neoma_project |
| github | `/home/granny/bin/mcp-github.sh` | `@modelcontextprotocol/server-github` | Loads GITHUB_TOKEN from secrets.env |
| supabase | `/home/granny/bin/mcp-supabase.sh` | `@supabase/mcp-server-supabase@latest` | Loads SUPABASE_ACCESS_TOKEN from secrets.env. Has global fallback at /usr/local/bin/mcp-server-supabase |
| obsidian | `/home/granny/bin/mcp-obsidian.sh` | `mcp-obsidian` | Vault: /media/granny/larger SSD/Neoma_project/memory/obsidian/Obsidian_Vault/Obsidian_memory |

These rarely fail. All wrappers share a common pattern:
1. Resolve nvm node/npx PATH (fallback: scan `~/.nvm/versions/node/*/bin/node`)
2. Load secrets from `$HOME/.config/neoma/secrets.env` and `api_key.env` (github, supabase only)
3. `exec npx -y <package> <args>`

**Common npx wrapper fix:** If node/npx not found, check that nvm is available. The wrappers look for node at `$HOME/.nvm/versions/node/*/bin/node` - since Claude runs as root, `$HOME=/root`, so nvm must have been installed there or the wrapper must hardcode `/home/granny/.nvm/...`. The obsidian wrapper does this correctly (hardcodes `/home/granny`), others use `$HOME` (works because root also has nvm installed).

### Tier 2: Custom servers

#### neoma_memory
- **Binary:** `/home/granny/.local/bin/neoma-mcp-stdio-shim`
- **Config env:** `NEOMA_CLI_ID=oc_agent`
- **Protocol:** Pure Python3, newline-delimited JSON-RPC, v2.0.0
- **Tools (9):** memory_store, memory_retrieve, memory_list, cli_dispatch, cli_broadcast, cli_inbox, cli_respond, cli_poll_response, cli_accept
- **Depends on:** memory bridge (localhost:8115) for tool calls only (init is static/hardcoded)
- **Zero external deps** - uses only stdlib (urllib.request, json, sys)

**Diagnosis:**
```bash
# Handshake test (should return immediately - pure python, no startup delay)
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1"}}}' \
  | timeout 5 env NEOMA_CLI_ID=oc_agent /home/granny/.local/bin/neoma-mcp-stdio-shim

# Check memory bridge health (needed for tool calls, not init)
curl -s http://localhost:8115/health
```

**Fix if broken:** The shim is a standalone python script with no dependencies. If it fails, the file is corrupt or missing. Restore from git: `git -C /home/granny show HEAD:.local/bin/neoma-mcp-stdio-shim > /home/granny/.local/bin/neoma-mcp-stdio-shim && chmod +x /home/granny/.local/bin/neoma-mcp-stdio-shim`

#### serena
- **Binary:** `/home/granny/.local/share/pipx/venvs/serena-agent/bin/python`
- **Args:** `["-c", "import sys; sys.argv = ['serena', 'start-mcp-server', '--transport', 'stdio']; from serena.cli import top_level; top_level()"]`
- **Protocol:** FastMCP, 25+ tools (code navigation, symbol search, memory, editing)
- **Startup:** Slow (~2-3 seconds). Logs to stderr during init.
- **Config:** `/root/.serena/serena_config.yml` (NOT /home/granny - because Claude runs as root)

**Why the invocation is weird:** The `serena` entry point at `~/.local/bin/serena` is a bash HTTP client wrapper, NOT the MCP server. The `serena-mcp-stdio` wrapper tries uvx -> uv -> system python3 fallbacks, but uvx/uv are not installed and system python lacks `sensai-utils`. The only working path is the pipx venv python invoking the click group directly.

**Diagnosis:**
```bash
# Test MCP handshake (allow extra time - serena is slow to start)
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1"}}}' \
  | timeout 15 /home/granny/.local/share/pipx/venvs/serena-agent/bin/python \
    -c "import sys; sys.argv = ['serena', 'start-mcp-server', '--transport', 'stdio']; from serena.cli import top_level; top_level()" \
    2>/dev/null | head -1 | python3 -c "import sys,json; print(json.load(sys.stdin)['result']['serverInfo'])"
```

**Fix if broken:**
```bash
# Reinstall pipx venv
pipx install serena-agent --force
# Or if local dev copy:
/home/granny/.local/share/pipx/venvs/serena-agent/bin/pip install -e "/media/granny/larger SSD/Neoma_project/agents/serena_new"
```

#### chrome-bridge
- **Binary:** `node`
- **Args:** `["/media/granny/larger SSD/Neoma_project/agents/chrome-bridge/mcp-server/server.js"]`
- **Config env:** `MCP_STDIO_ONLY=1`, `CHROME_BRIDGE_PORT=8130`
- **Protocol:** Node.js MCP server, provides browser automation tools (click, type, navigate, screenshot, etc.)
- **Depends on:** Chrome browser with the Neoma bridge extension running on port 8130

**Diagnosis:**
```bash
# Test MCP handshake
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1"}}}' \
  | timeout 10 env MCP_STDIO_ONLY=1 CHROME_BRIDGE_PORT=8130 \
    node "/media/granny/larger SSD/Neoma_project/agents/chrome-bridge/mcp-server/server.js" \
    2>/dev/null | head -1

# Check if Chrome bridge extension is listening
curl -s http://localhost:8130/health 2>/dev/null || echo "Chrome bridge not running on 8130"
```

**Fix if broken:**
- If handshake fails: check node can find the server.js file (path has spaces - must be quoted)
- If tools fail but MCP connects: Chrome browser must be running with the bridge extension loaded
- `MCP_STDIO_ONLY=1` is critical - without it, the server tries to bind an HTTP port instead of stdio

## Common Failure Patterns

### Pattern 1: Wrong binary (most common)

**Symptom:** Handshake test produces no output, usage message, or HTTP bind error.
**Cause:** The configured command is a CLI wrapper, HTTP server, or convenience script - not an MCP stdio server.
**Diagnosis:** Run `<command> --help` and `file <command>`. If it shows a usage message for something other than MCP, the binary is wrong.
**Fix:** Trace the wrapper chain to find the actual MCP server binary. Check memory bridge `mcp/repair_procedure` for known-good paths.

### Pattern 2: Missing runtime dependencies

**Symptom:** Handshake test produces ImportError or ModuleNotFoundError.
**Cause:** The binary is correct but dependencies aren't installed in the runtime environment.
**Diagnosis:** Check which python/node the script uses. Test imports in that environment.
**Fix:** Install deps in the correct venv/environment. For serena, use the pipx venv python.

### Pattern 3: All custom MCPs fail, npx MCPs work

**Cause:** Claude Code startup race condition, or PATH issue for custom servers.
**Fix:** Restart Claude Code. Use full absolute paths in config (never bare command names).

### Pattern 4: Scripts work but Claude Code shows failed

**Cause:** MCP connections are established at session startup. Config changes or fixes don't take effect mid-session.
**Fix:** Exit and restart Claude Code session.

### Pattern 5: Secrets/auth missing

**Cause:** Claude Code runs as root. `$HOME` = `/root`, not `/home/granny`. Wrappers that use `$HOME/.config/neoma/secrets.env` may load from wrong path.
**Fix:** Ensure secrets exist at both `/root/.config/neoma/secrets.env` and `/home/granny/.config/neoma/secrets.env`, or use hardcoded paths in wrappers.

### Pattern 6: Port conflict (HTTP-based servers)

**Cause:** Another instance is already listening on the port.
**Fix:**
```bash
lsof -i :PORT
kill <stale_pid>
```

### Pattern 7: User-scoped config overriding project-scoped

**Symptom:** You fix the project config but nothing changes.
**Cause:** `/root/.claude.json` has an mcpServers entry that takes precedence.
**Fix:** Check both files. Edit the user-scoped one, or remove it to fall through to project-scoped:
```bash
# Check what's actually active
claude mcp get <server-name>

# Remove from user scope if stale
claude mcp remove <server-name> -s user

# Add to user scope with correct config
claude mcp add <server-name> -s user -- <command> <args...>
```

### Pattern 8: Memory bridge connection_refused (neoma_memory tools fail)

**Symptom:** neoma_memory MCP connects fine (init is static) but tool calls return errors.
**Cause:** memory bridge service (port 8115) is down.
**Fix:**
```bash
# Check memory bridge
curl -s http://localhost:8115/health

# Restart if down
sudo -u granny DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus systemctl --user restart neoma-memory-bridge
```

## Config Validation Checklist

Before considering a config "fixed", verify each entry:

- [ ] `command` path exists and is executable
- [ ] `command` is actually an MCP stdio server (not HTTP, not CLI wrapper)
- [ ] MCP handshake test returns valid JSON with `serverInfo`
- [ ] All required `env` vars are set
- [ ] No duplicate functionality (e.g., don't configure both neoma_stdio and neoma_memory)
- [ ] Config is in the correct scope (user-scoped at `/root/.claude.json` takes precedence)

## Quick Repair Script

Test all 7 MCPs in one shot:

```bash
INIT='{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1"}}}'

echo "=== filesystem ===" && echo "$INIT" | timeout 10 /home/granny/bin/mcp-filesystem.sh 2>/dev/null | head -c 100 && echo
echo "=== github ===" && echo "$INIT" | timeout 10 /home/granny/bin/mcp-github.sh 2>/dev/null | head -c 100 && echo
echo "=== supabase ===" && echo "$INIT" | timeout 10 /home/granny/bin/mcp-supabase.sh 2>/dev/null | head -c 100 && echo
echo "=== obsidian ===" && echo "$INIT" | timeout 10 /home/granny/bin/mcp-obsidian.sh 2>/dev/null | head -c 100 && echo
echo "=== neoma_memory ===" && echo "$INIT" | timeout 5 env NEOMA_CLI_ID=oc_agent /home/granny/.local/bin/neoma-mcp-stdio-shim 2>/dev/null | head -c 100 && echo
echo "=== serena ===" && echo "$INIT" | timeout 15 /home/granny/.local/share/pipx/venvs/serena-agent/bin/python -c "import sys; sys.argv = ['serena', 'start-mcp-server', '--transport', 'stdio']; from serena.cli import top_level; top_level()" 2>/dev/null | head -c 100 && echo
echo "=== chrome-bridge ===" && echo "$INIT" | timeout 10 env MCP_STDIO_ONLY=1 CHROME_BRIDGE_PORT=8130 node "/media/granny/larger SSD/Neoma_project/agents/chrome-bridge/mcp-server/server.js" 2>/dev/null | head -c 100 && echo
```

## After Fixing

1. Update memory bridge repair procedure:
```bash
# Store known-good config to mcp/repair_procedure
curl -s -X POST http://localhost:8115/store -H "Content-Type: application/json" \
  -d '{"namespace":"mcp","key":"repair_procedure","value":{...}}'
```

2. Exit and restart Claude Code session.
3. Run `claude mcp list` to verify all servers connected.
4. Test a tool from each MCP to confirm end-to-end.

## History

| Date | Change |
|------|--------|
| 2026-02-05 | Updated to 7 servers: added chrome-bridge and obsidian. Fixed config location docs (user-scoped /root/.claude.json is active, not project-scoped). Added Pattern 7 (scope override) and Pattern 8 (memory bridge down). Added quick repair script. |
| 2026-02-03 | Fixed serena: bare `serena` command -> pipx venv python with click group. Removed neoma_stdio (HTTP server, not stdio MCP). Updated config from 6 to 5 servers. |
| 2026-02-02 | Fixed tldr: dead pyenv shim -> uv tool install at /root/.local/bin/tldr-mcp |
