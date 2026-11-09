---
name: load
description: Load full PR context into a fresh session — branch, associated PR, description, review comments, CI status, linked Linear ticket, and code changes. Use whenever the user says "load", "/load", "catch me up", "get up to speed", "load context", "what's going on with this branch/PR", or starts a new session on an existing branch and needs orientation. Pairs with the branch-diff skill for the actual code diff.
---

Bring the user up to speed on the current branch's PR when they're starting fresh. The goal is a tight, scannable briefing so they can resume work without manually clicking through GitHub, Linear, and CI dashboards.

## Workflow

Run these in parallel where possible — they're independent reads:

1. **Branch + PR detection**
   ```bash
   git rev-parse --abbrev-ref HEAD
   gh pr view --json number,title,state,isDraft,url,body,headRefName,baseRefName,author,additions,deletions,changedFiles,reviewDecision,mergeable,statusCheckRollup
   ```
   `gh pr view` with no arg uses the current branch's PR. If it errors with "no pull requests found", the branch has no PR yet — see fallback below.

2. **Review comments + discussion**
   ```bash
   gh pr view --json comments,reviews
   ```
   Filter for unresolved threads and review comments that request changes. Past resolved chatter is usually noise; surface what's blocking.

3. **CI status**
   The `statusCheckRollup` from step 1 already has this. Summarize as `X passed, Y failed, Z pending`. List failing checks by name with their target URL.

4. **Linked Linear ticket** (if referenced)
   Scan the branch name and PR description for a Linear ID pattern (e.g., `ENG-1234`). If found, fetch it with the Linear MCP `get_issue` tool. Pull title, status, description, acceptance criteria. Skip if no ID found — don't guess.

5. **Code changes**
   Don't reproduce branch-diff inline — invoke that skill (or tell the user to run `/branch-diff`) for the actual diff. The briefing should mention scope (`+X / -Y across N files` from PR data) but defer detailed code review to branch-diff.

## Output format

Structured briefing, lead with the most-need-to-know. Skip sections that have nothing useful.

```
# PR #<num>: <title>
<state badge: Draft / Open / Merged> · <reviewDecision> · <mergeable>
<url>

## Summary
<1-2 sentence distillation of the PR body — what & why>

## Outstanding review feedback
- <reviewer>: <comment summary> (<file>:<line> if applicable)
- ...
(or: "No outstanding review comments" if clean)

## CI
<X passed, Y failed, Z pending>
Failing: <check name> — <url>
(or: "All checks passing" / "No checks yet")

## Linked ticket
<TICKET-ID>: <title> [<status>]
<1-2 sentence ticket summary + acceptance criteria if present>
(or omit section entirely if no ticket linked)

## Scope
+<additions> / -<deletions> across <N> files
Run /branch-diff for the full code diff.
```

## Fallbacks

- **No PR for current branch** — show branch name, base detection, and `git log <base>..HEAD --oneline` so user sees what's here. Suggest pushing + opening a PR with `gh pr create` if there are commits.
- **Branch is the base branch itself** (on `main`) — explain there's nothing to load, ask if they meant a different branch.
- **Detached HEAD** — bail with the SHA + a note that there's no branch/PR context to load.
- **`gh` not authenticated** — surface the error directly and tell user to run `gh auth login`. Don't try to work around it.
- **No Linear ID in branch/PR** — skip the ticket section silently; this is the common case.

## Why pair with branch-diff

`/load` answers "what is this PR about and what's blocking it?" — the meta layer.
`/branch-diff` answers "what code actually changed?" — the content layer.
Keep them separate so users can pull just what they need without a wall of diff in the briefing.
