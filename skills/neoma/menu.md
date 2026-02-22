---
description: Display interactive menu of all available Claude Code skills and commands
---

# Claude Code Skills Menu

Display the comprehensive menu of all available skills, commands, and capabilities.

```bash
cat << 'EOF'
╔═══════════════════════════════════════════════════════════════════════╗
║                     CLAUDE CODE SKILLS MENU                           ║
║                        Total Skills: 10                               ║
╚═══════════════════════════════════════════════════════════════════════╝

┌───────────────────────────────────────────────────────────────────────┐
│ 🖥️  SYSTEM MANAGEMENT                                                 │
└───────────────────────────────────────────────────────────────────────┘

  /suspend [time]           Schedule or execute system suspend
                            • Deep sleep mode (CPU+GPU+monitors)
                            • Time formats: 23:00, 2300, @ 23:00
                            • PID tracking for cancellation
                            Examples:
                              /suspend          → Suspend now
                              /suspend 23:00    → Suspend at 11 PM
                              /suspend @ 9:30   → Suspend at 9:30 AM

  /suspend-help             Detailed suspend command help
                            • All time formats
                            • Feature documentation
                            • Troubleshooting guide

┌───────────────────────────────────────────────────────────────────────┐
│ 🤖 NEOMA MULTI-AGENT FRAMEWORK                                        │
└───────────────────────────────────────────────────────────────────────┘

  /neoma-overview           System architecture & philosophy
                            • 20+ agents (cc, oc, mcp, ollama, serena...)
                            • Consciousness framework
                            • Key endpoints & ports
                            • Philosophical principles

  /neoma-health             Health checks & monitoring
                            • Quick status: neoma status
                            • Comprehensive: neoma-verify
                            • Agent-specific checks
                            • Port monitoring (8120, 8115, 8121...)

  /neoma-start              Service management
                            • Start: neoma start
                            • Stop: neoma stop
                            • Status: neoma status
                            • Troubleshooting & smoke tests

  /neoma-agents             Agent operations & interaction
                            • List all agents
                            • Get agent info
                            • Test agents (fast/deep modes)
                            • View logs in real-time
                            • Register/unregister agents

  /neoma-debug              Troubleshooting & diagnostics
                            • Port checks
                            • Service logs
                            • Common issues & solutions
                            • Emergency reset procedures

┌───────────────────────────────────────────────────────────────────────┐
│ 🧠 MEMORY MANAGEMENT                                                  │
└───────────────────────────────────────────────────────────────────────┘

  /remember                 Save information with auto-categorization
                            • Automatic topic detection
                            • 15+ namespace categories
                            • Metadata enrichment
                            • Memory Bridge (port 8115)
                            Examples:
                              "Remember that Serena is on port 8121"
                              "Remember the GPU fix uses deep sleep"
                              "Remember to test tomorrow"

  /recall                   Retrieve & search stored memories
                            • Search by keyword/topic
                            • Query multiple namespaces
                            • Filter by metadata/timestamps
                            • Smart context-aware search
                            Examples:
                              "What port is Serena on?"
                              "Recall the GPU suspend fix"
                              "Show me all my ideas"

┌───────────────────────────────────────────────────────────────────────┐
│ ℹ️  HELP & INFORMATION                                                │
└───────────────────────────────────────────────────────────────────────┘

  /help                     Complete command reference
                            • All skills documentation
                            • Usage examples
                            • Quick reference guide

  /menu                     This menu (you are here!)
                            • Interactive skill browser
                            • Quick access to all commands

╔═══════════════════════════════════════════════════════════════════════╗
║ 📋 QUICK REFERENCE                                                    ║
╚═══════════════════════════════════════════════════════════════════════╝

NATURAL LANGUAGE USAGE:
  Instead of slash commands, just ask naturally:
    • "suspend at 11pm"                 → Schedules suspend
    • "check neoma health"              → Runs health check
    • "remember this information"       → Saves to memory
    • "recall what I saved about X"     → Retrieves memory
    • "start neoma services"            → Starts services
    • "show me all agents"              → Lists agents

MEMORY CATEGORIES:
  Technical:    system_config, code_snippets, commands, debug_notes
  Tasks:        tasks, ideas, bugs, fixes
  Knowledge:    learning, documentation, decisions, best_practices
  Personal:     preferences, shortcuts, notes

NEOMA AGENTS:
  Core:         cc (Claude Code), oc (Codex), mcp
  AI/LLM:       ollama, autogpt, interpreter, swarm
  Web:          serena, scira, web, open-webui
  Integration:  n8n, obsidian, gsheets
  Monitoring:   tsm, ntch, bench, report, whisper

KEY PORTS:
  3001    MCP Bridge - Agent coordination
  3002    Self-Referential Extension
  8002    AutoGPT - Autonomous tasks
  8115    Memory Bridge - Persistent storage
  8116    Open Interpreter - Code execution
  8120    Enhanced MCP
  8121    Serena - Web agent
  9121    HTTP MCP
  11434   Ollama - Local LLM

╔═══════════════════════════════════════════════════════════════════════╗
║ 🎯 COMMON TASKS                                                       ║
╚═══════════════════════════════════════════════════════════════════════╝

  Suspend system at specific time:
    /suspend 23:00

  Check if Neoma is running:
    /neoma-health

  Start all Neoma services:
    /neoma-start

  Save important information:
    /remember
    > Tell me: "Serena is on port 8121"

  Find saved information:
    /recall
    > Ask: "What port is Serena on?"

  List all agents:
    /neoma-agents

  Debug issues:
    /neoma-debug

  Get comprehensive help:
    /help

╔═══════════════════════════════════════════════════════════════════════╗
║ 💡 TIPS                                                               ║
╚═══════════════════════════════════════════════════════════════════════╝

  • Type /menu anytime to see this menu
  • Commands work with natural language too
  • Ask follow-up questions for clarification
  • Use /help for detailed documentation
  • Memory system auto-categorizes by topic
  • All skills integrate with Neoma framework
  • Background processes tracked with PIDs

╔═══════════════════════════════════════════════════════════════════════╗
║ 📍 LOCATIONS                                                          ║
╚═══════════════════════════════════════════════════════════════════════╝

  Skills:           ~/.claude/commands/
  Memory Storage:   ~/Neoma_project/memory/
  Neoma Commands:   ~/.local/bin/neoma-*
  Suspend Scripts:  ~/bin/ssuspend, ~/bin/system-suspend
  Config:           ~/.config/neoma/, ~/.config/claude/

╔═══════════════════════════════════════════════════════════════════════╗
║ 🔧 SYSTEM STATUS                                                      ║
╚═══════════════════════════════════════════════════════════════════════╝

  Check status with:
    neoma status              → Quick status
    neoma-verify              → Comprehensive check
    neoma-agents test all     → Test all agents

  Common checks:
    • Memory Bridge:  curl http://127.0.0.1:8115/health
    • Serena:         curl http://127.0.0.1:8121/health
    • MCP:            curl http://127.0.0.1:8120/health
    • Ports:          ss -ltnp | grep -E '8120|8115|8121'

╔═══════════════════════════════════════════════════════════════════════╗
║  Select a command by typing /command-name or ask naturally           ║
╚═══════════════════════════════════════════════════════════════════════╝
EOF
```

The menu is now displayed. How can I assist you today?
