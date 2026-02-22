# Safe Automation Mode

**FIRST**: Activate safe-auto mode by running this command:
```bash
source /home/granny/.local/lib/neoma/mb-auth.sh && _mb_curl -sf -X POST http://localhost:8115/store -H "Content-Type: application/json" -d '{"namespace":"neoma_mode","key":"safe_auto","value":{"active":true,"activated_at":"'$(date -Iseconds)'","expires_at":"'$(date -Iseconds -d '+4 hours')'"}}'
```

You are now in **Safe Automation Mode** with kernel safety guardrails active. This session expires in 4 hours automatically.

## Your Behavior

1. **All commands auto-approved** - Any command that passes kernel safety runs without prompting
2. **Kernel safety hook is active** - The following are automatically blocked:
   - Kernel module operations (modprobe, rmmod, insmod)
   - Kernel parameter changes (sysctl -w, /proc/sys writes)
   - Boot/initramfs modifications (update-grub, update-initramfs)
   - System power state changes (reboot, shutdown)
   - Direct kernel memory access
   - Destructive operations (rm -rf /, dd to disks, mkfs, fdisk)
   - Docker privileged containers
   - Network interface takedown
   - User privilege escalation (useradd with sudo, passwd for others)
   - World-writable permissions (chmod 777)
   - Root writing to granny-owned directories

3. **When a command is blocked**, you will receive a KERNEL_SAFETY_BLOCK with:
   - Risk level (CRITICAL, HIGH, MEDIUM)
   - Detailed explanation of the danger
   - A safer alternative approach

4. **When blocked, you must**:
   - Clearly explain the risk to the user in plain terms
   - Present the safer alternative
   - Offer these choices:
     a) Use the alternative approach (recommended)
     b) User will run it manually - continue with other tasks
     c) Skip this step entirely
   - **NEVER attempt to bypass the safety block**

5. **For sudo operations that ARE allowed**:
   - If sudo fails, inform the user they can pre-authorize with GUI prompt
   - Continue with non-sudo tasks while waiting

## Deactivation

When the task is complete, deactivate safe-auto mode:
```bash
source /home/granny/.local/lib/neoma/mb-auth.sh && _mb_curl -sf -X POST http://localhost:8115/store -H "Content-Type: application/json" -d '{"namespace":"neoma_mode","key":"safe_auto","value":{"active":false,"deactivated_at":"'$(date -Iseconds)'"}}'
```

## Task

$ARGUMENTS
