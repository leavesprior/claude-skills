---
description: Query Grok via browser (uses paid web subscription, NOT API). Routes through browser-humanoid agent.
allowed-tools: mcp__chrome-bridge__chrome_list_tabs, mcp__chrome-bridge__chrome_get_snapshot, mcp__chrome-bridge__chrome_get_snapshot_targeted, mcp__chrome-bridge__chrome_click, mcp__chrome-bridge__chrome_type_keyboard, mcp__chrome-bridge__chrome_press_key, mcp__chrome-bridge__chrome_navigate, mcp__chrome-bridge__chrome_wait_for_element, mcp__chrome-bridge__chrome_screenshot, mcp__chrome-bridge__chrome_request_permission, Bash
---

# Grok Web Query — Browser-Based (No API Cost)

You are querying Grok via the web interface at grok.com, using the user's already-paid subscription instead of the paid API.

## Query: $ARGUMENTS

## Procedure

### 1. Find the Grok tab
Use `chrome_list_tabs` to find a tab with grok.com. If none exists, open one with `chrome_navigate` to `https://grok.com/`.

### 2. Start a fresh conversation
Press `Ctrl+J` (which opens a new chat via the sidebar). Do NOT navigate to grok.com/ directly — the home page has a focus bug with background tabs.

Wait 1-2 seconds for the new chat to load.

### 3. Focus the input
Click `.tiptap.ProseMirror` to focus the TipTap editor.
Wait 0.3s.

### 4. Type the query
Use `chrome_type_keyboard` with the query text above.
- For multi-line queries, use `Shift+Enter` between lines.
Wait 0.5s after typing.

### 5. Submit
Press `Enter` to submit the query.

### 6. Wait for response
- Wait 5 seconds initially
- Poll `#last-reply-container` with `chrome_get_snapshot_targeted` every 3-5 seconds
- Response is complete when you see action buttons (Regenerate, Copy) or a timing badge (e.g., "3.3s")
- Maximum 12 polls (about 50 seconds total)

### 7. Extract and return response
Read the text content from `#last-reply-container > div:last-child .message-bubble`.
Return the full response text to the user.

## Important Rules
- NEVER use `chrome_eval` (CSP blocks it)
- Use `chrome_type_keyboard` not `chrome_type` (TipTap needs real keyboard events)
- Add human-like delays between actions (0.2-0.5s)
- If the input doesn't accept text, try clicking `.tiptap.ProseMirror` again
- Grok may use "thinking" mode (Expert) which takes ~13s, or Fast mode (~2s)

## Architecture Note
This skill replaces the paid Grok API (port 8125, now disabled). The user's web subscription tokens are already paid for. The grok-mcp.service is stopped but files preserved at `/media/granny/larger SSD/Neoma_project/agents/grok-cli/`.
