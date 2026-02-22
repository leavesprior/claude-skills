---
description: Wind Down - Graceful session-end orchestration with process awareness
---

# Wind Down - Graceful Session-End Orchestrator

Waits for running agents to complete, saves session state, and suspends the system.

**Tip:** For a quick save-and-sleep without process checking, use `/good-night` instead.

## Instructions

Execute the following phases in order. If any phase fails critically, report to the user and offer to skip or retry.

### Phase 0: Pre-flight

Check for double-invocation and set up:

```bash
# Source MB auth for all memory bridge calls in this session
source /home/granny/.local/lib/neoma/mb-auth.sh

# Check if wind-down is already active
EXISTING=$(_mb_curl -sf "http://localhost:8115/retrieve?namespace=neoma_mode&key=wind_down" 2>/dev/null)
```

If `EXISTING` contains `"active":true` and was set less than 10 minutes ago, tell the user wind-down is already in progress and stop.

```bash
# Mark wind-down as active
_mb_curl -sf -X POST http://localhost:8115/store -H "Content-Type: application/json" -d '{
  "namespace": "neoma_mode",
  "key": "wind_down",
  "value": {"active": true, "phase": "starting", "started_at": "'$(date -Iseconds)'"}
}'
```

If memory bridge is unreachable, warn but continue (degrade gracefully).

### Phase 1: Deactivate Safe-Auto (if active)

```bash
SAFE_AUTO=$(_mb_curl -sf "http://localhost:8115/retrieve?namespace=neoma_mode&key=safe_auto" 2>/dev/null)
```

If safe-auto is active (check `expires_at` too — if expired, note it was already expired):
```bash
_mb_curl -sf -X POST http://localhost:8115/store -H "Content-Type: application/json" -d '{
  "namespace": "neoma_mode",
  "key": "safe_auto",
  "value": {"active": false, "deactivated_by": "wind-down", "at": "'$(date -Iseconds)'"}
}'
```

Note: Tell the user if safe-auto was deactivated.

### Phase 2: Process Completion Check

```bash
neoma-process-check --json
```

**If no processes running:** Proceed immediately to Phase 3.

**If processes are running:**
1. Show the user what's running (names, types)
2. Poll every 15 seconds for up to 5 minutes:

```bash
neoma-process-check --wait 300 --json
```

3. **On timeout** (processes still running after 5 min): Ask the user:
   - **Wait longer** - extend by another 5 minutes
   - **Force proceed** - continue anyway, leaving agents running
   - **Cancel** - abort wind-down entirely

### Phase 3: Neoma's Pending Work

Check for any queued work items:

```bash
_mb_curl -sf "http://localhost:8115/retrieve?namespace=neoma_pending_work&key=queue" 2>/dev/null
```

If there are pending items, execute them with a 2-minute time-box. If the time-box expires, note what was skipped.

If no pending work namespace exists or it's empty, skip this phase silently.

### Phase 4: Good Night Workflow (Inline)

Execute the same steps as the `/good-night` skill, in quick mode:

#### 4a. Gather Session Context
- Infer topics from conversation (don't prompt the user unless nothing is clear)
- If the user provided tags or notes in their wind-down message, use those

#### 4b. Save Session Notes to ALL THREE Namespaces

```bash
DATE=$(date +%Y-%m-%d)
TIME=$(date +%H:%M:%S)
```

Build session note JSON with: `date`, `from_session: "wind-down"`, `priority`, `tags`, `completed` (what was done), `steps` (carry-forward items), `metrics`, `reference`.

Store to all three locations:
```bash
# 1. Canonical store
_mb_curl -s -X POST http://localhost:8115/store -H 'Content-Type: application/json' -d '{
  "namespace": "session_notes", "key": "'${DATE}'_winddown", "value": {SESSION_DATA}
}'
_mb_curl -s -X POST http://localhost:8115/store -H 'Content-Type: application/json' -d '{
  "namespace": "session_notes", "key": "latest", "value": {SESSION_DATA}
}'

# 2. Session continuity
_mb_curl -s -X POST http://localhost:8115/store -H 'Content-Type: application/json' -d '{
  "namespace": "session_continuity", "key": "latest", "value": {SESSION_DATA}
}'
_mb_curl -s -X POST http://localhost:8115/store -H 'Content-Type: application/json' -d '{
  "namespace": "session_continuity", "key": "next_session", "value": {SESSION_DATA}
}'

# 3. Morning briefing
_mb_curl -s -X POST http://localhost:8115/store -H 'Content-Type: application/json' -d '{
  "namespace": "morning_briefing", "key": "next_session", "value": {SESSION_DATA}
}'
```

#### 4c. Memory Review (Quick)
Review auto-memory files for today's learnings:
```
/home/granny/.claude/projects/-home-granny/memory/MEMORY.md
/home/granny/.claude/projects/-home-granny/memory/debugging.md
/home/granny/.claude/projects/-home-granny/memory/granny-services.md
/home/granny/.claude/projects/-home-granny/memory/mcp-debugging.md
```

Quick scan: Did I learn something today that belongs in these files? Is anything now wrong or outdated? Update if yes, skip if nothing to add.

#### 4d. Pre-Suspend Health Snapshot

```bash
echo "=== Pre-Suspend Health Snapshot ==="
for port in 8115 8117 8118 8119 8121 8122 8131; do
  curl -s --max-time 2 http://localhost:$port/health >/dev/null 2>&1 && echo "OK: port $port" || echo "DOWN: port $port"
done

echo ""
echo "=== Core Services ==="
GRANNY_UID=$(id -u granny)
for svc in neoma-wheeld neoma-memory-bridge neoma-guardian; do
  systemctl --user is-active $svc 2>/dev/null && echo "OK: $svc" || echo "DOWN: $svc"
done
```

### Phase 5: Suspend

```bash
# Mark wind-down as complete
_mb_curl -sf -X POST http://localhost:8115/store -H "Content-Type: application/json" -d '{
  "namespace": "neoma_mode",
  "key": "wind_down",
  "value": {"active": false, "completed_at": "'$(date -Iseconds)'"}
}'

# Suspend system
ssuspend
```

### Phase 6: Summary

Display a concise summary:

```
Wind-Down Complete
  Safe-auto: [deactivated / was not active]
  Processes: [waited for N agents / none running]
  Session: saved to 3 namespaces
  Health: [snapshot results]
  System: suspending

  Cancel suspend: kill $(cat /tmp/ssuspend.pid)
```

## Companion Commands

- `/good-night` - Quick save-and-sleep (no process awareness)
- `/suspend` - Just suspend without session save
- `/safe-auto` - Toggle safe automation mode
- `/neoma-health` - Full system health check
