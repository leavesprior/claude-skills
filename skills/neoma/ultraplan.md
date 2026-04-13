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

## Future work

When we're ready to actually codify ultraplan as a skill (not just a conversational ritual), this file becomes the entry point. The skill would:

- Auto-create/use this workspace as CWD
- Enforce the structured-plan + honest-reflection + open-questions shape
- Be Guardian-registered at T1
- Have wheelwright tests for self-consistency
- Preserve the ritual when offline from Claude Code web

That's a bigger conversation. For now: this file marks the place.
