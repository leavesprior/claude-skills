# Wheelwright Test Discipline

## When to Use
When adding new wheelwright tests, reviewing test tiers, auditing SI-RNA noise, or after discovering a service generates persistent red flags without meaningful repair.

## Rules

### R1: Tier Matches Dependency Position
The test tier determines how much immune system (SI-RNA) attention a failure receives:
- **canary/primary**: Core infrastructure whose failure degrades system function (MB, broker, synapse, guardian, identity tape, agent coordination)
- **bench**: Leaf services that consumers handle gracefully on failure (messaging relays, voice bridges, display UIs, optional tunnels)

**Test before assigning canary**: Does any script fail hard when this service is down? If every consumer returns False/logs/continues, the service is a leaf.

### R2: Assert Mechanisms, Not Outcomes
Tests should verify the machine works, not that the world is pleasant.
- BAD: `all_fresh == true` (passes when scoring is broken)
- GOOD: `score_variance > 0` (fails when scoring produces identical results)
- BAD: `error_count == 0` (passes when error counting is broken)
- GOOD: `error_count >= 0 AND total_checked > 0` (validates counting works)

### R3: Separate Lifecycles, Separate Keys
If two operations produce output at different times, store them under different keys. Never let a batch operation overwrite an interactive operation's result.

## Leaf Service Audit Pattern
```
1. Find canary tests for the suspect service
2. List all scripts that reference the service
3. Check if consumers handle failure gracefully (try/except, return False, fallback)
4. If yes → demote to bench, severity to low
5. Clear stale status files so SI-RNA stops processing old failures
6. Tombstone nerve cluster records
7. Verify SI-RNA observe shows reduced observation count
```

## Example: Clawdbot Demotion
```bash
# Clawdbot had 6 canary tests for a Telegram relay
# 22 scripts reference it — ALL handle failure gracefully
# SI-RNA was processing 3-4 clawdbot failures per cycle
# Nerve cluster couldn't fix it (needs secrets from ntch-unlock)
# Epigenetic override learned to avoid nerve_attempt (0/6 success rate)

# Fix: demote to bench, clear status files
python3 -c "import json; d=json.load(open(f)); d['wheel']='bench'; d['severity']='low'; json.dump(d,open(f,'w'),indent=2)"
rm ~/.local/share/neoma/wheel/clawdbot*.status
rm ~/.local/share/neoma/wheel/clawdbot*.err
```
