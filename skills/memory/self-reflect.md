# Self-Reflect: Memory & System Health Audit

Perform a comprehensive self-reflection audit of Neoma's memory, routing intelligence, and test coverage. Report findings and suggest actions.

## Steps

### 1. Knowledge Index Staleness Check
Read `~/.claude/projects/-home-granny/memory/knowledge-index.md` and identify:
- Entries with dates older than 7 days that may need updating
- Domains referenced in recent sessions but missing from the index
- Any broken MB namespace references

### 2. Memory Bridge Staleness Audit
```bash
source /home/granny/.local/lib/neoma/mb-auth.sh

# Check session_notes for recent activity
echo "=== Recent Session Notes ==="
_mb_curl -sf "http://localhost:8115/list?namespace=session_notes" | python3 -c "import json,sys; d=json.load(sys.stdin); keys=d.get('keys',[]); print(f'{len(keys)} session notes total'); [print(f'  {k}') for k in keys[-5:]]"

# Check pending actions
echo -e "\n=== Pending Actions ==="
_mb_curl -sf "http://localhost:8115/retrieve?namespace=neoma_pending_actions&key=standing_reminders" | python3 -c "import json,sys; d=json.load(sys.stdin); items=d.get('data',{}).get('value',{}).get('items',[]); [print(f'  [{i[\"priority\"]}] {i[\"title\"]} (since {i.get(\"since\",\"?\")})') for i in items]" 2>/dev/null || echo "  No standing reminders"

# Check stale entries in key namespaces
echo -e "\n=== Key Namespace Health ==="
for ns in "neoma_config" "neoma_architecture" "neoma_mode" "projects/dads_book_current"; do
    keys=$(_mb_curl -sf "http://localhost:8115/list?namespace=$ns" | python3 -c "import json,sys; d=json.load(sys.stdin); print(len(d.get('keys',[])))" 2>/dev/null || echo "0")
    echo "  $ns: $keys keys"
done
```

### 3. Flagpole Intelligence Review
```bash
neoma-flagpole summary 2>&1
```
Report: total flags per agent, any agents with zero flags, task types with conflicting signals.

### 4. Orphaned Wheelwright Tests
```bash
# Count total tests vs tests in any profile
TOTAL=$(ls ~/.config/neoma/wheel/tests/*.json 2>/dev/null | wc -l)
IN_PROFILE=$(cat ~/.config/neoma/wheel/profiles/*.include 2>/dev/null | grep -v '^#' | grep -v '^$' | sort -u | wc -l)
echo "Total test specs: $TOTAL"
echo "In at least one profile: $IN_PROFILE"
echo "Orphaned (approx): $((TOTAL - IN_PROFILE))"
```

### 5. Pending Actions Check
```bash
source /home/granny/.local/lib/neoma/mb-auth.sh
_mb_curl -sf "http://localhost:8115/list?namespace=neoma_pending_actions" | python3 -c "import json,sys; d=json.load(sys.stdin); [print(f'  {k}') for k in d.get('keys',[])]" 2>/dev/null
```

### 6. Report & Suggest
Compile findings into a concise report:
- **Critical**: Items that need immediate attention (broken references, security gaps, stale critical data)
- **Recommended**: Items that would improve system health (orphaned tests, stale memory entries)
- **Informational**: System status summary (flag counts, test coverage, MB health)

Suggest specific actions for each finding. If any finding is critical, offer to fix it immediately.
