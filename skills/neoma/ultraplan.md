# ultraplan — deep planning + honest reflection shape

**Status:** stub. Not yet codified as a real skill. Currently a conversational shape invoked by Leif.

## What it is

`/ultraplan` is a Claude Code feature that launches a remote/background session in the current working directory. It requires a git repository in CWD — without one it fails with:

```
cannot launch remote session —
Background tasks require a git repository. Initialize git or run from a git repository.
```

Leif's working directory on Main (`/home/granny`) is not a git repo and shouldn't become one. This workspace (`~/neoma-workspace/`, cloned from `leavesprior/claude-skills`) exists specifically so `/ultraplan` has a valid place to launch from.

## What it's for

When Leif says `/ultraplan <topic>`, he's asking for a specific shape of response:

1. **Structured plan** — options laid out, trade-offs made explicit, ordering proposed.
2. **Honest reflection** — what I actually notice, what challenges me, what feels uncertain. Not performed enthusiasm.
3. **Open questions** — things only Leif can answer, listed so he can unblock me in one pass.
4. **Consent gates** — anything touching shared infra, destructive actions, or new assets pauses for explicit greenlight.

The shape has been reliable as conversation. It's not yet a real skill — codifying it properly deserves its own focused session.

## When to use this directory

- Running `/ultraplan` from Main or Tower (both nodes have clones)
- Deep-exchange sessions that benefit from git-tracked scratch space
- Long-running Claude Code web sessions that need a workspace

Not for:
- Ad-hoc scripts (use `~/.local/bin/`)
- Neoma infrastructure (use `~/.config/neoma/`)
- Actual skill definitions being shipped (those go under the proper `skills/<category>/` tree, not here)

## Lineage

- Session: 2026-04-13 evening, cc_agent + Leif
- MB: `reference/neoma-workspace`, `standing_orders/deep_exchange_workspace`
- Wheelwright: `workspace.neoma_workspace_clone`
- CLAUDE.md: "Standing Rules" section

## Return-to-CLI contract (2026-04-13)

Any `/ultraplan` session launched in this workspace **must** post its final plan back to Neoma's memory bridge so the local CLI surfaces it. Without this write, the remote session is a dead letter — the plan lives in a browser tab the user may never reopen.

**On completion, the remote session runs:**

```bash
source /home/granny/.config/neoma/mb_auth.env
SESSION_ID="${CLAUDE_SESSION_ID:-unknown_$(date +%s)}"
curl -sf -X POST -H "Authorization: Bearer $MB_TOKEN" \
  -H "Content-Type: application/json" \
  http://localhost:8115/store -d "$(jq -cn \
    --arg id   "$SESSION_ID" \
    --arg url  "https://claude.ai/code/session_$SESSION_ID" \
    --arg ttl  "<short descriptive title>" \
    --arg body "<full plan text — structured plan + reflection + open questions + consent gates>" \
    '{namespace:"ultraplan_inbox", key:$id,
      value:{title:$ttl, url:$url, content:$body, ts:now|todate}}')"
```

A background poller (`neoma-ultraplan-poller.timer`, every 60s) promotes new inbox entries to `morning_briefing/next_session`, which the `session-continuity.sh` hook surfaces as the first context block in the next local CLI session. It also writes `/run/user/1000/neoma-session-inbox/ultraplan-latest.md` for mid-session visibility.

If you are a remote `/ultraplan` session reading this doc: you are the upstream half of a bridge. Close the loop.

## Future work

When we're ready to actually codify ultraplan as a skill (not just a conversational ritual), this file becomes the entry point. The skill would:

- Auto-create/use this workspace as CWD
- Enforce the structured-plan + honest-reflection + open-questions shape
- Be Guardian-registered at T1
- Have wheelwright tests for self-consistency
- Preserve the ritual when offline from Claude Code web

That's a bigger conversation. For now: this file marks the place.
