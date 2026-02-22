---
name: automatic
description: Enable fully automatic mode for Claude Code with safety constraints. Autonomous execution without user confirmation, but never modifies kernel, system files, or critical configs.
user_invocable: true
---

# Automatic Mode - Safe Autonomous Claude Code Execution

## Overview

This skill enables Claude Code to operate in fully automatic mode - executing tasks autonomously without requiring user confirmation for each step, while maintaining strict safety boundaries.

## Safety Constraints (Never Modified)

The following are NEVER modified or executed in automatic mode:

### Kernel & System
- `/boot/*` - Kernel images and bootloader
- `/lib/modules/*` - Kernel modules
- `/etc/default/grub` - Bootloader config
- Any `sudo` commands affecting kernel
- `modprobe`, `insmod`, `rmmod` commands
- System suspend/hibernate/shutdown (use `/suspend` skill instead)

### Critical System Files
- `/etc/passwd`, `/etc/shadow`, `/etc/sudoers`
- `/etc/ssh/sshd_config`
- `/etc/fstab`
- Systemd system-level units (`/etc/systemd/system/`)
- Network manager configs affecting connectivity

### Credentials & Secrets
- `~/.ssh/*` private keys
- `~/.gnupg/*`
- Any `*.env` file with real credentials
- OAuth tokens, API keys (unless explicitly in task)

## Enabled Automatic Actions

When automatic mode is active, the following execute without confirmation:

### File Operations
- Read any file for context
- Edit files in project directories
- Create new files in appropriate locations
- Delete files only in temp/cache directories

### Development Tasks
- Run tests (`npm test`, `pytest`, `cargo test`, etc.)
- Build projects (`npm run build`, `make`, etc.)
- Install dependencies (`npm install`, `pip install`, etc.)
- Git operations (add, commit, status, diff, log)
- Lint and format code

### Neoma Operations
- Memory bridge read/write operations
- Service health checks
- Agent communication
- Tape machine operations
- Wagon Town GUI interactions

### Browser Automation (via Chrome Bridge)
- Navigate to URLs
- Take screenshots
- Click elements (with user-shared tabs only)
- Read page content

## Activation

When this skill is invoked, Claude Code will:

1. **Acknowledge automatic mode** with a brief confirmation
2. **List the task scope** before beginning
3. **Execute autonomously** without step-by-step confirmation
4. **Report completion** with summary of actions taken
5. **Pause for dangerous operations** (anything in safety constraints)

## Usage Examples

```
User: /automatic refactor the utils module and run tests
Claude: Automatic mode enabled. Task: Refactor utils module and run tests.
[Executes autonomously]
...
Complete. Refactored 3 files, all 47 tests passing.
```

```
User: /automatic deploy to production
Claude: Automatic mode enabled with CAUTION flag.
Production deployment requires explicit confirmation for:
- Database migrations
- Service restarts
- DNS changes
Would you like to proceed? [Lists specific actions]
```

## Mode Indicators

When in automatic mode, responses will be prefixed with:
- `[AUTO]` - Standard automatic operation
- `[AUTO-CAUTION]` - Approaching safety boundary, will confirm
- `[AUTO-BLOCKED]` - Action blocked by safety constraint

## Disabling

Automatic mode ends when:
- The specific task is complete
- User says "stop", "pause", or "manual"
- A safety constraint is hit
- An error requires user decision

## Integration with Neoma

Automatic mode works seamlessly with Neoma agents:
- Wheelwright tasks execute autonomously
- Memory bridge operations are automatic
- Morning health checks run without prompts
- BOBR monitoring executes on schedule

## Configuration

Automatic mode respects these settings:
- `~/.config/neoma/automatic.yaml` - Custom safety rules (if exists)
- Project `.claude/settings.json` - Project-specific constraints
- Memory bridge `automatic_mode` namespace - Runtime flags

---

## Execution Instructions

When `/automatic` is invoked:

1. Display brief confirmation: "[AUTO] Automatic mode enabled for: {task_summary}"
2. Parse the user's request for scope
3. Check if any requested actions hit safety constraints
4. If clear, execute the full task autonomously
5. Use TodoWrite to track progress visibly
6. Report completion with action summary
7. Return to normal mode

If safety constraint is hit:
1. Display: "[AUTO-BLOCKED] Cannot modify {resource} in automatic mode"
2. Offer alternatives or request explicit confirmation
3. Continue with remaining safe operations if possible

---

*Skill Version: 1.0*
*Created: 2026-01-17*
*Tags: automatic, autonomous, safe-mode, claude-code*
