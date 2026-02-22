---
name: delegate-task
description: Delegate tasks to other Neoma agents (oc_agent/Codex or gem_agent/Gemini). Use when user asks to send work to another agent, or when a task is better suited for Codex (code-heavy) or Gemini (research-heavy).
allowed-tools:
  - Bash
  - Read
---

# Delegate Task to Another Agent

Send tasks to oc_agent (Codex) or gem_agent (Gemini) for parallel or specialized processing.

## Detection Priority Chain

neoma-delegate automatically finds the best running agent instance:

1. **Existing CLI in gnome-terminal tab** - Detects codex/gemini running in an open terminal tab via pgrep, maps to tab index, sends via D-Bus paste
2. **Existing tmux session** - Checks for Neoma tmux sessions (`tmux -L neoma`)
3. **Running on Tower** - SSH checks Tower (192.168.50.238) for running agent processes
4. **Start on Tower** - If Tower is reachable, starts a new tmux session there
5. **Start local tmux session** - Last resort fallback

This means: just run `neoma-delegate oc "task"` and it finds the right place automatically.

## Pre-Dispatch Checklist

Before delegating, verify the target agent is reachable:

1. **Is the agent running?** `pgrep -u granny -f "node.*bin/codex"` (or `gemini` for gem_agent)
2. **Where is it?** Check if it's in a GUI terminal tab or tmux session:
   - GUI tab: `ps -o ppid= -p $(pgrep -u granny -f "node.*bin/codex")` — if parent is gnome-terminal, it's a visible tab
   - tmux: `tmux -L neoma list-sessions 2>/dev/null` — look for matching session
3. **User-visible?** If the user is watching a specific terminal, prefer that one
4. **If nothing running**: neoma-delegate will start a new session, but confirm with user first if the task is large

## Post-Dispatch Verification

After `neoma-delegate` returns, verify the task was received:

1. **Check delegate output**: neoma-delegate reports which delivery method it used (tab, tmux, tower)
2. **For tmux delivery**: `tmux -L neoma capture-pane -t <session> -p | tail -3` — verify the prompt appeared
3. **For GUI tab delivery**: delegate output confirms tab index and paste action
4. **No confirmation within 10 seconds?** Investigate:
   - Process may have crashed: re-check `pgrep`
   - May be waiting for input or auth
   - Terminal may have scrolled past the prompt

## When to Use This Skill

- User explicitly asks to delegate to oc_agent, gem_agent, codex, or gemini
- Task is better suited for another agent's strengths
- User wants to run something in a separate terminal
- Parallel processing would be beneficial

## Agent Strengths

| Agent | CLI | Best For |
|-------|-----|----------|
| **oc_agent** | `codex` | Large refactoring, code review, bulk operations, static analysis |
| **gem_agent** | `gemini` | Research, documentation, external API investigation, comparisons |

## Commands

```bash
# Auto-detect (finds existing CLI first, then tmux, then tower)
neoma-delegate oc "task description"
neoma-delegate gem "task description"

# Force run on Tower
neoma-delegate --tower oc "task description"

# Force local (skip terminal detection)
neoma-delegate --local oc "task description"

# Open in new terminal window
neoma-delegate --terminal oc "task description"

# Background (saves to file)
neoma-delegate --background gem "task description"
```

## How Terminal Detection Works

On GNOME Wayland, the script:
1. Uses `pgrep -u granny -f "node.*bin/codex"` to find the CLI process
2. Walks the process tree to confirm it's in gnome-terminal (not tmux)
3. Maps the pts device to a tab index via `ps --ppid <gnome-terminal-pid>`
4. Switches to that tab via D-Bus: `org.gtk.Actions.Activate "active-tab"`
5. Puts task text in Wayland clipboard via `wl-copy`
6. Pastes via D-Bus: `org.gtk.Actions.Activate "paste-text"`
7. Sends Enter key via `ydotool key 28:1 28:0`

**Dependencies:** `wl-clipboard` (wl-copy), `ydotool`, `gdbus`

## Examples

### Delegate refactoring to Codex
```bash
neoma-delegate oc "Refactor all API handlers to use async/await consistently"
```

### Delegate research to Gemini
```bash
neoma-delegate gem "Compare React state management libraries: Redux vs Zustand vs Jotai"
```

### Force delegation to Tower
```bash
neoma-delegate --tower oc "Review the authentication module for security issues"
```

## Routing Guidelines (from CLAUDE.md)

| Task Type | Primary Agent | Support Agent |
|-----------|---------------|---------------|
| architectural_decisions | cc_agent | gem (research) |
| large_refactoring | oc_agent | cc (verify) |
| research_comparison | gem_agent | cc (implement) |
| complex_debugging | cc_agent | oc (static) |
| documentation_writing | gem_agent | cc (accuracy) |

## Run on Tower (Remote Node)

Delegate to agents running on the Tower node (192.168.50.238):

```bash
# Auto-detect handles Tower automatically, but you can force it:
neoma-delegate --tower oc "task description"
neoma-delegate --tower gem "task description"

# Or use the direct tower script:
neoma-tower-agent oc "task description"
neoma-tower-agent --terminal oc "task description"
```

Use Tower when:
- Local compute is limited
- Want to run in parallel with local work
- Tower has specific models/resources needed

## Tracking

Delegations are logged to memory bridge:
```bash
curl "http://localhost:8115/list?namespace=neoma_delegations"
```
