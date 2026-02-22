# Good Morning - Neoma System Health Check

Run comprehensive morning health checks on system, Neoma, and Wheelwright.

## Instructions

Execute the following health checks in order:

### 1. System Health Check
```bash
# Check disk space (warn if >90%)
df -h | grep -E "^/dev" | awk '{gsub(/%/,"",$5); if($5>90) print "WARNING: "$6" at "$5"%"; else print "OK: "$6" at "$5"%"}'

# Check critical systemd services (must use DBUS env since claude runs as root)
GRANNY_UID=$(id -u granny)
for svc in neoma-wheeld neoma-memory-bridge neoma-morning; do
  sudo -u granny env XDG_RUNTIME_DIR=/run/user/$GRANNY_UID DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$GRANNY_UID/bus" systemctl --user is-active $svc 2>/dev/null && echo "OK: $svc" || echo "DOWN: $svc"
done
```

### 2. Neoma Agent Health Check
Check all spoke agent ports: 8115, 8117, 8118, 8119, 8121, 8122, 8131
```bash
for port in 8115 8117 8118 8119 8121 8122 8131; do
  curl -s --max-time 2 http://localhost:$port/health >/dev/null 2>&1 && echo "OK: port $port" || echo "DOWN: port $port"
done
```

### 3. Wheelwright Health Check
```bash
# Check daemon uptime (via granny's DBUS)
GRANNY_UID=$(id -u granny)
sudo -u granny env XDG_RUNTIME_DIR=/run/user/$GRANNY_UID DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$GRANNY_UID/bus" systemctl --user status neoma-wheeld 2>/dev/null | grep -E "Active:|Main PID:"

# Check tape buffer
curl -s http://localhost:8119/memory/wheel/tape 2>/dev/null | jq -r '.entries | length' 2>/dev/null || echo "Tape: unavailable"
```

### 4. MCP Server Status
```bash
claude mcp list 2>/dev/null | grep -c "Connected" | xargs -I{} echo "MCP servers connected: {}"
```

### 5. Storage Quick Check
```bash
# Check for obvious issues
echo "=== Storage Summary ==="
df -h /home/granny "/media/granny/larger SSD" /media/granny/storage_chest_AI 2>/dev/null | tail -n +2

# Count files needing attention (empty files in key dirs)
find "/media/granny/larger SSD/Neoma_project" -type f -empty 2>/dev/null | wc -l | xargs -I{} echo "Empty files in Neoma: {}"
```

### 5.5 Home Directory Hygiene
```bash
echo "=== Home Directory Hygiene ==="
loose_files=$(ls -p ~ 2>/dev/null | grep -v "/" | grep -v "CLAUDE.md" | wc -l)
old_files=$(find ~ -maxdepth 1 -type f -mtime +7 ! -name '.*' ! -name 'CLAUDE.md' 2>/dev/null | wc -l)
echo "Loose files in ~: $loose_files"
echo "Files older than 7 days: $old_files"

if [[ $old_files -gt 30 ]]; then
    echo "WARNING: Home directory needs tidying. Run /home-tidy"
elif [[ $old_files -gt 20 ]]; then
    echo "NOTICE: Consider running /home-tidy soon"
else
    echo "OK: Home directory is tidy"
fi
```

### 5.8 BOBR Financial Deadlines
```bash
echo "=== BOBR Financial Deadlines ==="
CHECK=$(/home/granny/.local/bin/neoma-bobr-status check 2>/dev/null)
if [ -z "$CHECK" ]; then
    echo "  No financial data available"
else
    FILING=$(echo "$CHECK" | jq -r '.deadlines.filing_days_left // "?"')
    OVERDUE=$(echo "$CHECK" | jq -r '.deadlines.overdue // 0')
    APPROACHING=$(echo "$CHECK" | jq -r '.deadlines.approaching // 0')
    STALE=$(echo "$CHECK" | jq -r '.freshness.stale // false')
    GAP=$(echo "$CHECK" | jq -r '.freshness.data_gap // ""')
    echo "  Tax filing: ${FILING} days (Apr 15, 2026)"
    if [ "$OVERDUE" -gt 0 ]; then
        echo "  WARNING: $OVERDUE overdue items"
        echo "$CHECK" | jq -r '.overdue_items[] | "    * \(.action) (\(-.days_left)d overdue)"'
    fi
    if [ "$APPROACHING" -gt 0 ]; then
        echo "  Approaching: $APPROACHING items within 30 days"
        echo "$CHECK" | jq -r '.approaching_items[] | "    > \(.action) (\(.days_left)d left)"'
    fi
    if [ "$STALE" = "true" ]; then
        echo "  WARNING: Financial data is stale! Run: neoma-bobr-status ingest"
    fi
    if [ -n "$GAP" ] && [ "$GAP" != "" ]; then
        echo "  DATA GAP: $GAP"
    fi
fi
```

### 6. Generate Report
After running checks, summarize findings and store to memory bridge:
```bash
curl -s -X POST http://localhost:8115/store \
  -H "Content-Type: application/json" \
  -d '{"namespace":"morning","key":"'$(date +%Y%m%d)'","value":{"timestamp":"'"$(date -Iseconds)"'","status":"completed","checks":"system,neoma,wheelwright,mcp,storage"}}'
```

## Expected Output
- All services should show "OK" or "Active"
- All ports should respond to health checks
- Disk usage should be below 90%
- MCP servers should be connected
- Report stored to memory bridge

## 7. Morning Briefing (Previous Session Notes)
```bash
# Try all three session note namespaces (session_notes is canonical, others are fallbacks)
for ns in session_notes session_continuity morning_briefing; do
  RESULT=$(curl -s "http://localhost:8115/retrieve?namespace=${ns}&key=latest" 2>/dev/null)
  if echo "$RESULT" | python3 -c "import sys,json; d=json.load(sys.stdin); assert d.get('ok') and d.get('data',{}).get('value')" 2>/dev/null; then
    echo "$RESULT" | python3 -c "
import sys, json
v = json.load(sys.stdin)['data']['value']
print('=== MORNING BRIEFING ===')
print(f'Source: ${ns}')
print(f'Date: {v.get(\"date\", \"today\")}')
print(f'Priority: {v.get(\"priority\", \"none\")}')
print()
if v.get('completed'):
    print('Completed:')
    for i, s in enumerate(v['completed'], 1):
        print(f'  {i}. {s}')
    print()
if v.get('steps'):
    print('Carry-forward:')
    for i, s in enumerate(v['steps'], 1):
        print(f'  {i}. {s}')
    print()
print(f'Metrics: {v.get(\"metrics\", \"n/a\")}')
print(f'Reference: {v.get(\"reference\", \"n/a\")}')
"
    break
  fi
done
```

### 7.5 Diagnostic Results (Last 24h)
```bash
echo "=== DIAGNOSTIC RESULTS (last 24h) ==="
CUTOFF=$(date -d '24 hours ago' -Iseconds 2>/dev/null || date -v-24H -Iseconds 2>/dev/null)
# List all diagnostic result keys and check each
KEYS=$(curl -s "http://localhost:8115/list?namespace=diagnostic_results" 2>/dev/null | python3 -c "
import sys, json
try:
    data = json.load(sys.stdin)
    for k in data.get('keys', []):
        print(k)
except: pass
" 2>/dev/null)

if [ -z "$KEYS" ]; then
    echo "  No diagnostic results found"
else
    FOUND=0
    UNRESOLVED=0
    while IFS= read -r key; do
        [ -z "$key" ] && continue
        RESULT=$(curl -s "http://localhost:8115/retrieve?namespace=diagnostic_results&key=$key" 2>/dev/null)
        python3 -c "
import sys, json
from datetime import datetime, timedelta
try:
    data = json.load(sys.stdin)
    v = data.get('data', {}).get('value') or data.get('value') or {}
    harvested = v.get('harvested_at', '')
    if not harvested:
        sys.exit(1)
    # Check if within last 24h (strip timezone for naive comparison)
    clean = harvested.replace('Z', '')
    if '+' in clean[10:]:
        clean = clean[:clean.rindex('+')]
    elif clean.count('-') > 2:
        clean = clean[:clean.rindex('-')]
    dt = datetime.fromisoformat(clean)
    cutoff = datetime.now() - timedelta(hours=24)
    if dt < cutoff:
        sys.exit(1)
    sev = v.get('severity', 'unknown')
    svc = v.get('service', '?')
    atype = v.get('anomaly_type', '?')
    root = v.get('root_cause', 'Unknown')
    fix = v.get('recommended_fix', 'None')
    status_icon = '!' if sev in ('high', 'critical') else '.'
    print(f'  [{status_icon}] {svc}: {atype} (severity={sev})')
    print(f'      Root cause: {root}')
    print(f'      Recommended: {fix}')
    if sev in ('high', 'critical'):
        print(f'      ** NEEDS ATTENTION **')
except: sys.exit(1)
" <<< "$RESULT" 2>/dev/null && FOUND=$((FOUND + 1))
    done <<< "$KEYS"
    if [ "$FOUND" -eq 0 ]; then
        echo "  No recent diagnostic results (last 24h)"
    fi
fi
```

## If Issues Found
- DOWN services: `sudo -u granny env XDG_RUNTIME_DIR=/run/user/$(id -u granny) DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/$(id -u granny)/bus" systemctl --user restart <service>`
- DOWN ports: Check agent logs in `~/.local/share/neoma/logs/`
- High disk: Run `neoma-storage-audit --quick`
