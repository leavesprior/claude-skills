# Claude Skills

**Neoma self-learning repository for Claude Code skills, commands, and operational knowledge.**

Part of the Neoma cognitive architecture -- a cognitive prosthetic system built on Claude Code.

## Purpose

This repository captures the skills, commands, and operational patterns that Neoma Claude Code agents learn during real work. Instead of being lost between sessions or scattered across local dotfiles, skills are versioned, categorized, and available for reuse.

Skills are discovered organically during work, reviewed for quality, and committed here as reusable building blocks.

## Directory Structure

    skills/
      memory/        Memory agent skills (recall, remember, memory bridge)
      automation/    Automation skills (safe-auto, scheduling, wind-down)
      security/      Security audit and safety skills
      development/   Development workflow skills (visualization, analysis, debugging)
      neoma/         Neoma-specific operational skills (health, agents, tower)

## Skill Categories

### Memory (skills/memory/)
Skills for persistent memory operations -- storing, retrieving, and managing knowledge across sessions. Includes the Memory Bridge integration, recall/remember commands, and self-reflection audits.

### Automation (skills/automation/)
Skills for autonomous and semi-autonomous operation -- safe-auto mode with kernel safety guardrails, multi-agent team coordination, task delegation, wind-down orchestration, and scheduling.

### Security (skills/security/)
Security audit skills, safety constraints, and the kernel safety gate patterns that prevent destructive operations during autonomous mode.

### Development (skills/development/)
Development workflow skills -- diagram generation (Excalidraw, Mermaid, Obsidian Canvas), deep analysis, web automation, browser tools, and MCP troubleshooting.

### Neoma (skills/neoma/)
Neoma-specific operational skills -- system health checks, agent management, tower control, startup/shutdown procedures, debug tools, and the Clawdbot messaging gateway.

## How Skills Get Added

1. **Discovery**: During real work, a new pattern, workflow, or capability emerges
2. **Extraction**: The skill is extracted into a standalone markdown file with frontmatter metadata
3. **Categorization**: Placed in the appropriate category directory
4. **Review**: Verified to work independently and documented with usage instructions
5. **Commit**: Added to this repository with a descriptive commit message

Skills follow the Claude Code skill format:
- Commands (*.md with description frontmatter) -- invoked via /command-name
- Skills (SKILL.md in a directory) -- automatically activated by Claude based on context
- Reference docs -- supplementary knowledge files

## Related Systems

- **Flagpole System**: Tracks per-agent task performance (green/red flags) for intelligent routing
- **Memory Bridge**: Persistent key-value store across sessions (port 8115)
- **Knowledge Consolidation**: Weekly automated staleness detection and pattern surfacing
- **Terminal Restoration**: Session recovery across suspend/resume cycles

## License

Private -- part of the Neoma system.
