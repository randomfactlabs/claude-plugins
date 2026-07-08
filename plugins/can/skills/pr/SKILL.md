---
description: Opens a PR for the current Can ticket. Use when shipping work tied to a CAN-* ticket. Invoke as /can:pr [CAN-123].
---

# /can:pr

Open a pull request for the current Can ticket.

## Preconditions

The Can MCP server (`can`) must be authenticated. If its tools are not
available, tell the user to run `/mcp` and complete the Can sign-in, then
retry — do not fabricate ticket data. (In headless `claude -p` / Agent SDK
runs there is no `/mcp` panel, so the user must have authenticated once in an
interactive session first.)

## Steps

1. Determine the ticket key from the arguments (`$ARGUMENTS`), or infer it from
   the current git branch name (e.g. `feat/can-123-...` → `CAN-123`).
2. Read the ticket from Can (`resolve_ticket`, then `get_ticket`) for its title
   and description.
3. Gather the diff for the current branch (against the default branch), and
   confirm `gh auth status` — opening the PR is the deliverable.
4. Write a PR title and body that summarize the change and reference the ticket
   key. Push the branch first (`git push -u origin HEAD`).
5. Open the PR with `gh pr create` against the default branch.
6. Link the PR back on the ticket with `create_comment` so the board carries the
   URL.
7. Move the ticket on Can per the team convention: leave it in an `in-progress`
   column while the PR is in review, and move it to a `done` column only once
   the PR **merges**. (Board mechanics live in the Can `agent-ticket-workflow`
   help topic.)
