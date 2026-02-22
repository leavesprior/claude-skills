---
name: gnome-session
description: Run commands in the user's GNOME desktop session with proper D-Bus access. Use when gsettings, dconf, notify-send, or other GNOME session commands fail with "dbus-launch" errors. This skill provides the workaround for interacting with the live desktop from CLI sessions.
allowed-tools:
  - Bash
  - Read
---

# GNOME Session Command Skill

Run commands that require access to the user's GNOME desktop session (D-Bus).

## Problem

When running from Claude Code or other CLI sessions, GNOME commands fail:

```
(process:1234): dconf-WARNING **: failed to commit changes to dconf:
Failed to execute child process "dbus-launch" (No such file or directory)
```

This happens because the CLI session doesn't have access to the user's D-Bus session bus.

## Solution

### Quick Method - Use the Wrapper

```bash
gnome-session-cmd <command> [args...]
```

Examples:
```bash
# Get current dock favorites
gnome-session-cmd gsettings get org.gnome.shell favorite-apps

# Set dock favorites
gnome-session-cmd gsettings set org.gnome.shell favorite-apps "['app1.desktop', 'app2.desktop']"

# Change theme
gnome-session-cmd gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'

# Send notification
gnome-session-cmd notify-send "Title" "Message body"
```

### Manual Method

If `gnome-session-cmd` isn't available:

```bash
# 1. Find the D-Bus session address from a running GNOME process
sudo cat /proc/$(pgrep -u granny gnome-shell | head -1)/environ | tr '\0' '\n' | grep DBUS_SESSION

# 2. Use it with your command
sudo -u granny DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/1000/bus" gsettings ...
```

## Common Use Cases

| Task | Command |
|------|---------|
| Add app to dock | `gnome-session-cmd gsettings set org.gnome.shell favorite-apps "[...]"` |
| Get current favorites | `gnome-session-cmd gsettings get org.gnome.shell favorite-apps` |
| Change wallpaper | `gnome-session-cmd gsettings set org.gnome.desktop.background picture-uri "file:///path/to/image"` |
| Toggle dark mode | `gnome-session-cmd gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'` |
| Read dconf value | `gnome-session-cmd dconf read /org/gnome/shell/favorite-apps` |
| Send notification | `gnome-session-cmd notify-send "Title" "Body"` |
| Refresh dock icons | `gnome-session-cmd gsettings set org.gnome.shell favorite-apps "$(gnome-session-cmd gsettings get org.gnome.shell favorite-apps)"` |

## Affected Commands

These commands require D-Bus session access:
- `gsettings`
- `dconf`
- `notify-send`
- `zenity` (for some features)
- `gdbus`
- `gio`
- Any command that talks to GNOME Shell or settings daemon

## Troubleshooting

### gnome-session-cmd not found
```bash
# Check if it exists
ls -la /usr/local/bin/gnome-session-cmd

# If missing, recreate it
sudo tee /usr/local/bin/gnome-session-cmd << 'EOF'
#!/bin/bash
exec sudo -u granny DBUS_SESSION_BUS_ADDRESS="unix:path=/run/user/1000/bus" "$@"
EOF
sudo chmod +x /usr/local/bin/gnome-session-cmd
```

### Permission denied
The command needs sudo access to read the GNOME process environment and run as the desktop user.

### GNOME Shell not running
If `pgrep gnome-shell` returns nothing, the user isn't logged into a GNOME session. The workaround only works when GNOME is actively running.

## Memory Bridge Reference

This pattern is also stored in memory bridge for retrieval:
```bash
curl "http://localhost:8115/retrieve?namespace=cc_agent_patterns&key=gnome_dbus_workaround"
```
