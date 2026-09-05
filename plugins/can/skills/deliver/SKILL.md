---
description: Deliver every ticket in one column of a Can board — take them, fan out one worktree subagent per ticket (Sonnet for well-specified work, Opus for refactors), review and merge each PR, turn findings into new tickets, and summarize. Use when asked to "deliver / clear / work through everything in To Do". Invoke as /can:deliver [BOARD] [column].
---

# /can:deliver

Manage the delivery of every ticket in one column of a Can board, end to end:
**take the tickets, implement each on its own branch through a subagent, review
and merge each PR yourself, file what you learn as new tickets, and report.**
You are the delivery manager, not the implementer — the subagents write the
code; you own the board, the review, the merge order and the summary.

Arguments (`$ARGUMENTS`): `[BOARD] [column]`. `BOARD` is the 3–5 letter board
key (`CAN`); if omitted, infer it from the ticket keys in recent commits
(`git log --oneline -30 | grep -oE ' [A-Z]{3,5}-[0-9]+' | head -1`). `column`
defaults to `To Do` and is matched case-insensitively by name.

## Preconditions — fail loudly if missing

- **The Can MCP server (`can`) is authenticated.** If its tools aren't
  available, tell the user to run `/mcp` and complete the Can sign-in, then
  retry. Never fabricate ticket data.
- **A git repo for the board's project, on its default branch, with a clean
  tree, and an authenticated GitHub CLI** (`gh auth status`). Merged PRs are the
  deliverable, so if `gh` isn't authed, stop.
- **Read the repo's `CLAUDE.md` first.** It owns the conventions the subagents
  must follow (lint/typecheck/test commands, worktree location, private
  registry auth, env-file decryption, commit trailers). Everything below defers
  to it.

## 1. Inventory before you touch anything

1. `list_tickets` with `columnName` for the target column, and `list_columns`
   once for the `in-progress` and `done` column ids.
2. **Read every ticket's comments** (`list_comments`). Unblock notes ("0.1.5 is
   published now"), corrections and prior attempts live there, not in the
   description.
3. **Verify external prerequisites the tickets assert.** A ticket that says a
   package version "is published" or another repo's PR "has merged" is a claim
   — check the registry (`pnpm view <pkg> versions`), the sibling repo on
   disk, the referenced file. A ticket whose prerequisite is genuinely missing
   stays in the column with a comment saying why; don't start it.
4. **Map the tickets onto each other.** Which ones touch the same files
   (`package.json`, lockfiles, `CLAUDE.md`, a shared route)? Which one's
   acceptance references another's output (a heading, a script, a constant)?
   Decide the **merge order** now: small, self-contained PRs first; the one
   with the broadest footprint last, rebased onto everything else.
5. **Pick a model per ticket.**
   - **Sonnet** when the ticket is well specified and mechanical: it hands you
     the exact code, the exact commands, or a scaffold to copy; the judgment is
     in the ticket, not the work.
   - **Opus** when the work is a refactor, a migration onto a library, a
     cross-repo adoption, anything with a visible-output diff to explain, or
     anything where the ticket says "check it covers this; if not, say why".
   - When in doubt, Opus. A wrong Sonnet costs a rerun; a wrong Opus costs
     tokens.

## 2. Take every ticket, in one pass

For each ticket: `update_ticket` with `agentAssigneeId` set to your agent slug
(`claude`), then `assign_self` (records the signed-in human). Then one
`bulk_status_tickets` call moving all of them to the `in-progress` column.
If a ticket already has a *different* `agentAssigneeId`, leave it alone and say
so in the summary.

## 3. Fan out — one subagent per ticket, in parallel

Launch all of them in **one message**, each with `isolation: "worktree"` and
the `model` you chose. **Do not implement any ticket inline yourself**, even a
small one — a serial loop in one tree tangles unrelated changes onto one
branch, and inline work stops you from reviewing.

Each brief must carry, explicitly:

- **Worktree setup.** A fresh worktree has none of the git-ignored files: the
  registry token (`.npmrc`), the decrypted env files, `node_modules`, built
  functions. Name the exact commands from `CLAUDE.md` (e.g. `bash
  scripts/ar-auth.sh`, `pnpm install` per install root, `pnpm env:decrypt`),
  and which install roots the ticket needs.
- **The branch name** (`feat/<key-lower>-<slug>`) and that the worktree is
  already cut from the default branch.
- **The ticket verbatim, plus its comments and your prerequisite findings**
  (versions confirmed, files that exist, decisions already made). Don't make
  the agent rediscover what you verified.
- **The workflow to follow:** call `get_help` with `topic: "ticket-to-pr"` and
  follow it, *except* for the board steps and the merge — the orchestrator
  owns those. State it plainly: **do not touch the ticket on the board, do not
  merge, do not deploy.**
- **The checks to run** (from `CLAUDE.md`) and that results must be reported
  honestly, including anything red or not run.
- **Cross-ticket coordination** where your dependency map found any: "a
  sibling PR edits the same `CLAUDE.md` section — keep your edit to one or two
  sentences"; "the heading you must reference already exists today, don't
  depend on the sibling's addition".
- **Baselines** when acceptance asks for a before/after comparison: build the
  untouched tree first and stash the output in the scratchpad before changing
  anything.
- **Commit trailer and PR footer** exactly as the session requires, the
  `<KEY>: …` commit-subject convention, and a PR body with approach, key
  decisions and testing.
- **The report format:** PR URL, branch, worktree path, what was verified,
  what could *not* be verified, and **findings outside the ticket's scope as a
  separate list** so you can file tickets.

Tell each agent it may fix an incidental breakage its change causes (a test
that relied on the old behavior), but must call it out; anything else
out-of-scope is a finding, not a fix.

## 4. While they run — do the orchestrator's homework

Anything a ticket asks of "the deployer" or "before the first deploy" that
doesn't need the code: confirm the production env holds the keys a
fail-closed change will demand, check the live endpoint's current state,
confirm a referenced doc heading exists. Record what you found; it goes in the
ticket comment and the summary. Don't touch the subagents' files.

## 5. As each PR lands — review, then merge

Never merge on the agent's report alone. For each PR, in order:

1. **Read the diff yourself** (`gh pr diff`). Look specifically for:
   - **User-visible copy or composition changes the ticket did not ask for.**
     A voice/lint test can push an agent into rewriting a tagline or inventing
     marketing copy. Product copy is "keep as is" unless the ticket says
     otherwise — restore it and make the test accommodate it transparently
     (and file a ticket for the proper fix). **Fact-check any new user-facing
     sentence** against the code: does onboarding really do what the blurb
     claims?
   - **Acceptance items claimed but not run.** If an item was skipped because
     it needed infrastructure (emulators, seed, a dev server), run it yourself
     now when you can — that is what the worktree is still there for. Look at
     the output, not just the exit code (open regenerated images; diff
     regenerated files against the committed ones).
   - **Incidental fixes** and whether they belong in this PR.
   - **Findings the agent buried** in the PR body that need a ticket.
2. **Rebase onto the current default branch** if another PR merged since the
   branch was cut (`git rebase main` in the agent's worktree), and **re-run the
   checks on the rebased result** — typecheck, tests, build — before merging.
3. **Make small corrections directly in the worktree** and commit them with the
   session's trailers; re-spawn or message the agent only for substantial
   rework. Update the PR body with a short "review" section explaining what
   you changed and why.
4. **Squash-merge** with the subject `<KEY>: <title> (#<PR>)`, then
   `git pull --ff-only` on the main checkout.
5. **Comment on the ticket**: PR URL + merge sha, what changed, what was
   verified, what was deliberately not done, the follow-up tickets filed.
6. **Move the ticket to `done`** — merged is accepted. If acceptance includes a
   post-deploy check, say in the comment that it is verified on the next
   release.
7. **Clean up:** `git worktree remove --force <path>`, delete the local and
   remote branch, `git worktree prune`.

Small PRs merge as they land; the broad one waits for the others and rebases.

## 6. Findings become tickets, not silence

Every out-of-scope finding — from an agent's report, from your review, from
your own verification — is either **incorporated** (small, in scope, and the
ticket's acceptance is better for it) or **filed**:

- Product/app follow-ups go on **the same board**, in the column you are
  clearing, so the next run picks them up.
- Gaps in a shared package or platform go on **that project's board** (for the
  Random Fact family: `PLAT`).
- Each ticket: one paragraph of context that names where it was found, a
  **Do** section, an **Acceptance** section, and a **Related** line back to
  the ticket that surfaced it.
- **A regression the batch introduces is never left implicit.** File it, name
  it as a regression in the ticket title, and flag it in the summary as a
  decision the user must make before the next release.

## 7. Never deploy

Merging ships nothing in these repos; `pnpm release` from a developer machine
is the user's call. Do the pre-deploy verification the tickets ask for, then
stop. Say in the summary exactly what the next release will verify.

## 8. Confirm the board, then summarize

`list_tickets` the target column once more (it should hold only the follow-ups
you filed) and `get_ticket` each delivered key to confirm `done`. Then write
one summary that stands alone:

- A table: ticket, model, PR, one line on what it does; then the state of the
  default branch after your post-merge install/typecheck/test run.
- **Review calls the user should eyeball** — every judgment you made on their
  behalf (copy restored, copy rewritten, an incidental fix kept).
- **Findings → tickets**, grouped by board, the regression first.
- **Verified but not committed** — anything you ran and deliberately threw
  away, and why.
- **Next step for the user** — the release, and any ticket that should land
  before it.
