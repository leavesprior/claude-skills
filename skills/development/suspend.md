---
description: Schedule system suspend with deep sleep (CPU+GPU+monitors)
---

# Suspend System Command

You are helping the user schedule a system suspend using the enhanced `ssuspend` command.

## Available Actions:

1. **Immediate Suspend**: Run `ssuspend` without arguments
2. **Scheduled Suspend**: Run `ssuspend <time>` where time can be:
   - `HH:MM` format (e.g., `23:00`, `9:30`)
   - `HHMM` format (e.g., `2300`, `930`)
   - With `@` prefix (e.g., `@ 23:00`)

## Features:
- Deep sleep mode (suspends CPU, GPU, and monitors)
- Pre-suspend health checks for Neoma services
- Tapeproof snapshot before suspend
- USB and network wake source management
- 3-second cancellation window
- PID saved to `/tmp/ssuspend.pid` for easy cancellation

## Instructions:

1. Ask the user what time they want to suspend (if not already specified)
2. Run the appropriate `ssuspend` command
3. Confirm the scheduled time and provide the PID for cancellation
4. If the user wants to cancel, show them: `kill $(cat /tmp/ssuspend.pid)`

## Examples:

```bash
# Suspend now
ssuspend

# Suspend at 11:00 PM
ssuspend 23:00

# Suspend at 9:30 AM tomorrow
ssuspend 9:30

# Suspend using military time
ssuspend 2300
```

## Cancellation:

To cancel a scheduled suspend:
```bash
kill $(cat /tmp/ssuspend.pid)
```

## Technical Details:

The command chain:
1. `ssuspend` → calls `system-suspend` at scheduled time
2. `system-suspend` → runs `prepare_suspend.sh` for pre-suspend tasks
3. Sets deep sleep mode via `/sys/power/mem_sleep`
4. Executes `sudo systemctl suspend`

Now proceed to help the user schedule their system suspend.
