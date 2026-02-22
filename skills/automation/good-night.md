---
description: Good Night - Save session, run health snapshot, and suspend system
---

# Good Night - Neoma Session Close & System Suspend

Save session progress, review memory, capture health snapshot, and prepare system for sleep.

> **Tip:** For orchestrated shutdown that waits for running agents/processes to complete, use `/wind-down` instead.

## Instructions

Execute the following steps in order:

### 1. Gather Session Context

Ask the user (if not already provided):
- What topics were covered today?
- Any specific tags to apply?
- Any hints for next session?

If the user just says "good night" without details, infer from the conversation context.

### 2. Save Session Notes to ALL THREE Namespaces

**CRITICAL:** Write to all three so every consumer finds the notes.

```bash
DATE=$(date +%Y-%m-%d)
TIME=$(date +%H:%M:%S)
```

Build the session note JSON with: `date`, `from_session: "goodnight"`, `priority`, `tags`, `completed` (what was done), `steps` (carry-forward items), `metrics`, `reference`.

Then store to:

```bash
# 1. Canonical store
curl -s -X POST http://localhost:8115/store -H 'Content-Type: application/json' -d '{
  "namespace": "session_notes", "key": "'${DATE}'_goodnight", "value": {SESSION_DATA}
}'
# Also update latest pointer
curl -s -X POST http://localhost:8115/store -H 'Content-Type: application/json' -d '{
  "namespace": "session_notes", "key": "latest", "value": {SESSION_DATA}
}'

# 2. Session continuity (startup hook reads this)
curl -s -X POST http://localhost:8115/store -H 'Content-Type: application/json' -d '{
  "namespace": "session_continuity", "key": "latest", "value": {SESSION_DATA}
}'
curl -s -X POST http://localhost:8115/store -H 'Content-Type: application/json' -d '{
  "namespace": "session_continuity", "key": "next_session", "value": {SESSION_DATA}
}'

# 3. Morning briefing (good-morning skill reads this)
curl -s -X POST http://localhost:8115/store -H 'Content-Type: application/json' -d '{
  "namespace": "morning_briefing", "key": "next_session", "value": {SESSION_DATA}
}'
```

### 3. Memory Review (Self-Improvement Step)

Before closing, re-read and update auto-memory files:

```
Files to review:
  /root/.claude/projects/-home-granny/memory/MEMORY.md
  /root/.claude/projects/-home-granny/memory/debugging.md
  /root/.claude/projects/-home-granny/memory/granny-services.md
  /root/.claude/projects/-home-granny/memory/mcp-debugging.md
```

For each file, ask:
- **Did I learn something today that belongs here?** A pattern I rediscovered, a mistake I made, a fix that should be permanent knowledge.
- **Is anything written here now wrong or outdated?** Remove or update it.
- **Is MEMORY.md under 200 lines?** If approaching the limit, move detail to topic files and keep MEMORY.md as an index.
- **Should a new topic file be created?** If today's work revealed a recurring area (e.g., a new subsystem, a new debugging domain), give it its own file and link from MEMORY.md.

This step is what turns individual sessions into accumulated wisdom. Don't skip it.

### 4. Pre-Suspend Health Snapshot

```bash
# Quick health check using proper DBUS env
GRANNY_UID=$(id -u granny)
GRANNY_ENV="XDG_RUNTIME_DIR=/run/user/$GRANNY_UID DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/$GRANNY_UID/bus"

echo "=== Pre-Suspend Health Snapshot ==="
for port in 8115 8117 8118 8119 8121 8122 8131; do
  curl -s --max-time 2 http://localhost:$port/health >/dev/null 2>&1 && echo "OK: port $port" || echo "DOWN: port $port"
done

echo ""
echo "=== Core Services ==="
for svc in neoma-wheeld neoma-memory-bridge neoma-guardian; do
  sudo -u granny env $GRANNY_ENV systemctl --user is-active $svc 2>/dev/null && echo "OK: $svc" || echo "DOWN: $svc"
done
```

### 5. Execute System Suspend

```bash
ssuspend
```

### 6. Confirm to User

Display summary:
- Session saved to all 3 namespaces
- Memory files reviewed and updated
- Health snapshot: captured
- System: suspending

Provide cancellation command: `kill $(cat /tmp/ssuspend.pid)`

## Quick Mode

If user just says "good night" without context:
1. Infer topics from conversation
2. Save session notes (all 3 namespaces)
3. Do a quick memory scan (update if anything obvious, skip if nothing learned)
4. Run health snapshot
5. Suspend

## Companion Commands

- `/good-morning` - Morning health check and briefing retrieval
- `/suspend` - Just suspend without session save
- `/remember` - Save specific information to memory
