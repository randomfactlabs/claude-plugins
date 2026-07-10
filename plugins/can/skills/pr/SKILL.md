---
description: Take a Can ticket (or several) and drive each to a PR — read it, implement the change on a branch, test it, and open the pull request. Use when handed a CAN-* ticket to do. Invoke as /can:pr [CAN-123].
---

# /can:pr

Take a Can ticket from the board to a review-ready pull request: **read it,
implement the change, test it, then open the PR** — not just open a PR for work
that already exists.

The workflow itself is owned by the Can MCP server as the single source of
truth (it stays current as the flow evolves). Do NOT reproduce it from memory —
fetch it and follow it verbatim:

1. **Load the workflow.** Call the `can` MCP server's `get_help` tool with
   `topic: "ticket-to-pr"`. It returns the authoritative ticket → PR flow —
   branching, testing, opening the PR, agent self-assignment, and
   one-subagent-per-ticket fan-out when you're handed several keys. Follow it
   exactly.
2. **Target ticket(s).** Take the key(s) from the arguments (`$ARGUMENTS`), or
   infer a single key from the current git branch (e.g. `feat/can-123-...` →
   `CAN-123`). **One key → work it inline** in this session. **Several
   space-separated keys** (`CAN-83 CAN-84`) **→ fan out**: one implementation
   subagent and one PR per ticket, run in parallel — never loop them onto one
   branch.

## Preconditions

- **The Can MCP server (`can`) must be authenticated** — the workflow above is
  fetched from it and the ticket tools live there. If its tools aren't
  available, tell the user to run `/mcp` and complete the Can sign-in, then
  retry; don't fabricate ticket data. (Headless `claude -p` / Agent SDK runs
  have no `/mcp` panel, so the user must have authenticated once in an
  interactive session first.)
- **A git repo and an authenticated GitHub CLI** (`gh auth status`). Opening the
  PR is the deliverable — if `gh` isn't authed, stop and say so.
