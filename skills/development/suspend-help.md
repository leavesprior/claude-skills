---
description: Show usage examples and help for the ssuspend command
---

# Suspend Command Help

Display comprehensive help and examples for the `ssuspend` command.

Run the following command to show usage information:

```bash
cat << 'EOF'
╔═══════════════════════════════════════════════════════════════╗
║           SSUSPEND - Smart System Suspend Command            ║
╚═══════════════════════════════════════════════════════════════╝

USAGE:
  ssuspend              Suspend system immediately
  ssuspend <time>       Schedule suspend for specific time

TIME FORMATS:
  23:00                 24-hour format with colon
  @ 23:00               With @ prefix
  2300                  Military time (HHMM)
  9:30                  Single digit hour supported
  930                   Short format (HMM)

FEATURES:
  ✓ Deep sleep mode (CPU + GPU + monitors fully suspended)
  ✓ Pre-suspend health checks for Neoma services
  ✓ Tapeproof snapshot before suspend
  ✓ USB wake source management
  ✓ Network wake-on-LAN disabled
  ✓ 3-second cancellation window
  ✓ Background timer with saved PID

EXAMPLES:
  $ ssuspend
    → Suspends immediately after 3-second countdown

  $ ssuspend 23:00
    ⏰ Suspend scheduled for 23:00
       Time until suspend: 2h 15m
       Current time: 20:45
    ✓ Background timer started (PID: 123456)

  $ ssuspend @ 9:30
    → Schedules suspend for 9:30 (AM next day if past)

CANCELLATION:
  Method 1: kill $(cat /tmp/ssuspend.pid)
  Method 2: kill <PID>    (use PID shown when scheduled)

CHECK STATUS:
  $ ps aux | grep system-suspend
  $ cat /tmp/ssuspend.pid

TECHNICAL DETAILS:
  Command chain:
    ssuspend → system-suspend → prepare_suspend.sh → systemctl suspend

  Sleep mode: deep (via /sys/power/mem_sleep)
  Pre-suspend: Health checks, snapshots, wake source management
  Resume: System wakes with all services restored

FILES:
  /home/granny/bin/ssuspend          Main command
  /home/granny/bin/system-suspend    Suspend handler
  /home/granny/bin/prepare_suspend.sh Pre-suspend tasks
  /tmp/ssuspend.pid                  Active timer PID

LOGS:
  Pre-suspend logs: ~/logs/pre_suspend_*.log
  Tapeproof tapes:  ~/logs/tape_*.tape

╔═══════════════════════════════════════════════════════════════╗
║  For immediate help, just ask Claude: "suspend at 11pm"      ║
╚═══════════════════════════════════════════════════════════════╝
EOF
```
