---
description: Take a Can ticket and drive it to a PR — read it, implement the change on a branch, test it, and open the pull request. Use when handed a CAN-* ticket to do. Invoke as /can:pr [CAN-123].
---

# /can:pr

Take a single Can ticket from the board to a review-ready pull request:
**read it, implement the change, test it, then open the PR** — not just open a
PR for work that already exists. This is the `ticket-to-pr` flow.

## Preconditions

- **The Can MCP server (`can`) must be authenticated.** If its tools aren't
  available, tell the user to run `/mcp` and complete the Can sign-in, then
  retry — do not fabricate ticket data. (In headless `claude -p` / Agent SDK
  runs there is no `/mcp` panel, so the user must have authenticated once in an
  interactive session first.)
- **A git repo and an authenticated GitHub CLI** (`gh auth status`). Opening the
  PR is the deliverable — if `gh` isn't authed, stop and say so.

## Steps

1. **Resolve the ticket.** Take the key from the arguments (`$ARGUMENTS`), or
   infer it from the current git branch (e.g. `feat/can-123-...` → `CAN-123`).
   Call `resolve_ticket` with the key; keep the returned `slug` for later calls.
   Read the title and description with `get_ticket`. Echo the title back so a
   wrong key is caught before any work. If it's already done or owned by a
   different `agentAssigneeId`, stop and ask.
2. **Self-assign and move to in-progress.** `update_ticket` with
   `agentAssigneeId` set to your agent slug (e.g. `claude`); if a human is
   driving you, also set `assigneeId` to them. Then `list_columns` and
   `update_ticket` the `columnId` to an `in-progress` column.
3. **Branch.** Never commit to the default branch. From an up-to-date base, cut
   `feat/<ticket-key-lowercased>-<short-slug>` (e.g.
   `feat/can-123-csv-export`). If the working tree has unrelated uncommitted
   changes, stop and ask.
4. **Implement the change — this is the point of the command.** Make the code
   changes the ticket calls for, scoped to this ticket only (don't fix unrelated
   things — it muddies the PR). Read the repo's `CLAUDE.md`/contributing docs and
   match its conventions. Add tests for the new behavior and run the project's
   checks (lint, types, tests) until green. If a check legitimately can't pass,
   call it out in the PR rather than forcing it.
5. **Open the PR.** Commit referencing the ticket key, `git push -u origin HEAD`,
   then `gh pr create` against the default branch with a body covering the
   approach, key decisions, and testing.
6. **Link the PR back on the ticket.** `create_comment` on the ticket with the PR
   URL so the board carries the link.
7. **Leave the ticket in-progress; report.** Opening a PR is not finishing the
   ticket — it stays in its `in-progress` column while the PR is in review. Tell
   the user the PR URL, a short summary, and the test result, and make clear it's
   *in review*, not done. Move it to a `done` column only once the PR **merges**
   (board mechanics live in the Can `agent-ticket-workflow` help topic).

## More than one ticket?

This command is for **one** ticket, done now. If you were handed several, don't
loop this flow on one branch — implement each in its own branch/PR so reviewers
read one ticket's worth of change per PR.
