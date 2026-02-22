---
description: Send messages, check status, and manage Clawdbot messaging channels (Discord, Telegram, etc.)
---

# Clawdbot - External Communication Gateway

Clawdbot enables you to communicate with Neoma from anywhere using Discord, Telegram, or other messaging platforms.

## Quick Actions

### Check Status
```bash
source ~/.nvm/nvm.sh && nvm use 22 >/dev/null 2>&1
curl -sf http://localhost:8190/health | python3 -m json.tool 2>/dev/null || echo "Bridge not running"
npx clawdbot gateway health 2>/dev/null || echo "Gateway not running"
```

### Start Services
```bash
source ~/.nvm/nvm.sh && nvm use 22

# Start gateway (background)
npx clawdbot gateway run &

# Wait for gateway, then start bridge
sleep 3
nohup node /home/granny/.local/lib/neoma-clawdbot-bridge/bridge.mjs > /tmp/neoma-clawdbot-bridge.log 2>&1 &
```

### List Configured Channels
```bash
source ~/.nvm/nvm.sh && nvm use 22 && npx clawdbot channels list
```

### Send a Message
```bash
# Via bridge HTTP API
curl -s -X POST http://localhost:8190/send \
  -H 'Content-Type: application/json' \
  -d '{"channel":"discord","to":"CHANNEL_ID","text":"Hello from Neoma!"}'
```

### Test Message Routing
```bash
# Test without needing a real channel
curl -s -X POST http://localhost:8190/test-roundtrip \
  -H 'Content-Type: application/json' \
  -d '{"text":"/search test query"}' | python3 -m json.tool
```

## Channel Management

### Add Discord Channel
```bash
source ~/.nvm/nvm.sh && nvm use 22
npx clawdbot channels add --channel discord --token YOUR_BOT_TOKEN
```

### Add Telegram Channel
```bash
source ~/.nvm/nvm.sh && nvm use 22
npx clawdbot channels add --channel telegram --token YOUR_BOT_TOKEN
```

### Remove Channel
```bash
npx clawdbot channels remove --channel discord
```

## Message Commands (from Discord/Telegram)

When messaging Neoma from your phone:
- `/search <query>` - Web search via Scira
- `/fetch <url>` - Fetch webpage via Serena
- `/remember <note>` - Save a note to memory
- `/recall <topic>` - Retrieve saved notes
- `/status` - Check Neoma health

## Troubleshooting

### Gateway won't start
```bash
ss -tlnp | grep 18789  # Check if port in use
pkill -f "clawdbot gateway"  # Kill existing
```

### Bridge not connecting
```bash
tail -20 /tmp/neoma-clawdbot-bridge.log
curl http://localhost:18789/health  # Gateway must be up first
```

### Messages not routing
```bash
# Check memory bridge (required dependency)
curl http://localhost:8115/list
```

Now help the user with their Clawdbot task. Run status checks or start services as needed.
