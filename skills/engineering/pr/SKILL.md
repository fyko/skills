---
name: pr
description: Use when creating a pull request — fills the PR body, investigates the "why" before writing, and runs /humanizer before submitting
---

# Create PR

### 1. Gather context

Run these in parallel:

- `git branch --show-current` — current branch name
- `git log main..HEAD --oneline` — all commits on this branch
- `git diff main...HEAD --stat` — files changed summary
- `git diff main...HEAD` — full diff for understanding changes

### 2. Detect base branch

- Default base: `main`
- If the branch was cut off another feature branch (check `git log --oneline main..HEAD` for merge commits or `git merge-base`), use that instead
- Confirm with user if ambiguous

### 3. Investigate the "why" — do this BEFORE writing

The body is only as good as the motivation you can surface. Before drafting, hunt for the real reason this PR exists. Check in this order:

1. **Linear ticket** — if the branch or commits reference a ticket ID (e.g. `ENG-1234`), fetch it with the Linear MCP `get_issue` tool. Pull the ticket description, acceptance criteria, and any comments that describe the user-visible symptom or business driver.
2. **Referenced Sentry issue** — if commits mention Sentry or an error name, search with the Sentry MCP `search_issues` tool. Copy the raw Sentry URL (keep the `?referrer=...&notification_uuid=...` querystring — that's the signature).
3. **Prior PR references** — if the diff looks like a fixup, a revert, or a follow-up, find the earlier PR with `gh pr list` / `gh pr view` and link it (`Missed in #NNNN`, `Fixes a regression from #NNNN`, `Re-applys changes from #NNNN`).
4. **Slack conversations** — if the user mentions one or a recent commit does, include the permalink. Don't fabricate.
5. **Affected customer / stakeholder** — grep the diff and commit messages for customer or stakeholder names. Surface them by name in the body.
6. **The code itself** — if none of the above surfaces a driver, read the diff and reason backward: what was the previous behavior, what breaks without this change, who notices?
7. **Ask the user** — if after steps 1–6 the motivation is still unclear (no ticket, no Sentry, no Slack thread, no prior PR, and the diff alone doesn't tell the story), **stop and ask**. Prompt with something concrete: _"I can see what this changes but not why — is this driven by a specific customer, incident, ticket, or conversation? A one-line answer is fine."_ Do not invent a why.

If the why is genuinely "self-evident from title" (bump, typo, one-line config), that's fine — short body is correct. But do the investigation first; don't skip and then write "cleanup."

### 4. Generate PR title

- Conventional commit: `<type>(<scope>): <Description>`
- With a relevant Linear ticket: `<type>(<scope>): <TEAM>-#### | <Description>` (e.g. `fix: ENG-1234 | Fix staging env vars`, `feat(api): ENG-412 | Add partner router`)
- Common types: `feat`, `fix`, `chore`, `refactor`, `perf`, `revert`, `ci`, `docs`
- Scopes: reuse the ones the repo already uses (check `git log --oneline -50`). Combined scopes like `be/fe` or `infra/be` are fine when genuinely cross-cutting.
- **Capitalize the first word after the colon**. Lowercase rest. No trailing period.
- Keep under 70 characters.
- **Linear ticket in title**: if step 3 surfaced a relevant Linear ticket (branch name contains it, commits reference it, or ticket is the clear driver), include it in the title as `<TEAM>-#### | ` right after the colon. Use whatever team prefix Linear uses for that ticket. Then drop the `Fixes <TEAM>-####` line from the body — don't duplicate. If no ticket surfaced, skip the prefix entirely.

### 5. Fill PR body (in Carter's voice)

Use this template:

```markdown
## What

## Why

### Additional Information
```

The template has `## What` / `## Why` / `### Additional Information`. Keep all three section headers — leave Additional Information's placeholder comment if there's nothing load-bearing. Predictable structure makes the template easy for cursor agents and humans to template against later.

#### Length is the first decision. Pick a mode before writing.

**Default mode — use ≥90% of PRs. Body ≤80 words total.**

- `## What`: one sentence describing the diff.
- `## Why`: one sentence naming the trigger (customer, person, ticket, prior PR, incident).
- `### Additional Information`: placeholder comment, or one link if genuinely useful.

**Expanded mode — only if at least one is true:**

- Real merge-order dependency on other PRs (then 1-2 terse lines, not a numbered playbook).
- Real prod risk worth a `> [!CAUTION]` callout an on-call would actually need.
- Feature PR >500 LOC where the diff alone can't carry the why.
- User explicitly asks for a long body.

In expanded mode: one extra paragraph per section, max. No sub-bullets unless there are genuinely >3 independent items.

**Hard cap: 200 words.** If you hit it, stop and ask the user: _"is this worth >200 words or am I performing?"_ Default answer is cut.

#### Mirror test — match the vibe of these real short PRs

> **feat(infra): Auth for Argo staging**
>
> ## What
>
> Adds Cloudflare Access as an OIDC provider for ArgoCD.
>
> ## Why
>
> Everyone shouldn't have the admin password. Also using Cloudflare Access means we don't have to define a new Google app for every instance and we have one centralized place for auth. I'm gonna use Cloudflare Access for flipt and n8n.

> **feat(agents): Add Telnyx inference**
>
> ## What
>
> Adds Telnyx inference to our available stack.
>
> ## Why
>
> They offer frontier models that we wanna play with. They also do prompt caching, which AWS Bedrock doesn't do, so this should have lower latency. I've been using their GLM 5.1 model with OpenCode and its super fast!

> **feat(be): Partner router for external consumers**
>
> ## What
>
> This PR creates a new `/partner/v1` router to be used by external customers.
>
> ## Why
>
> We already have this endpoint, but its not versioned and we frequently introduce breaking changes to it. One partner, in this example, needs an endpoint to get conversations by call id. This first pass is pretty lean.

If your draft doesn't sound like these, rewrite.

#### `## What`

- One sentence. Present-tense, third-person: _"This PR adds X"_, _"Adds Y"_, _"Switches Z to W"_.
- Opener verbs: `adds`, `creates`, `fixes`, `implements`, `updates`, `switches`, `moves`, `enables`, `unblocks`, `wires up`, `bootstraps`.
- Inline backticks for symbols/paths only when load-bearing.
- **Do NOT enumerate every file changed.** Reviewers read the stat. If there are genuinely independent pieces, max 3 bullets.

#### `## Why`

Lead with the trigger in reader-replayable terms. One sentence is the bar.

- ✅ _"A customer was passing Zendesk ticket IDs as `source_id` when there could be multiple tickets per user."_
- ❌ _"Fixes a bug in source_id handling."_
- ❌ _"Tech debt cleanup."_ — find the actual trigger or don't open the PR today.

Pick the ONE thing that actually motivated this PR. Add a Linear/Sentry/Slack/prior-PR link if it makes the why click. In default mode you do NOT need a checklist of mechanism+blast-radius+evidence+risk — that's expanded-mode territory only.

#### Banned moves (turn a good PR into slop)

- Per-file enumeration of changes (the diff stat shows it).
- "Decisions worth flagging:" sub-sections (belongs in code comments or ADRs).
- Before/After paragraphs paraphrasing the diff.
- `### Additional Information` filled with stats ("22 commits, 71 files, +5,329 LoC") — GitHub already shows this.
- "Verified via `helm template`" / "Tested by `kubectl get`..." narration unless explicitly asked.
- yaml/json code blocks duplicating the diff.
- Multi-step rollout bash blocks (link a runbook or put it in the commit message).
- Corporate voice ("The organization needs to grant..."). If it doesn't sound like Slack, rewrite.

Merge-order notes are fine but terse: _"Merge after #123 applies."_ — not a 5-step numbered playbook.

#### Voice rules (match Carter's tone)

- **Opener:** ~half of PRs start with `"This PR <verb>..."`. Rotate the verb. Alternates: `Resolves ENG-####`, `Missed in #NNNN`, `Per <name>'s request`, `See the title`.
- **Present-tense third person** for describing the diff. First-person only when narrating reasoning or mistakes (_"I removed the retry because..."_, _"read as: oops"_).
- **Softeners** like `just` welcome (_"This PR just switches..."_).
- **Own screw-ups dryly**: `oops`, `read as: oops`, `Missed in #NNNN`, `I panicked`. Short.
- **Casual punctuation** allowed sparingly: lowercase `i`, `idk`, `b/c`, `gonna`, trailing `...`, `™`, `(accepting feedback)`. Not every PR.
- **Emphasis:** `**bold**` or `__CAPS__` (e.g. `**NEVER**`, `__NOT__`). Avoid shouting full sentences.
- **Em-dash `—`** for asides. Slack-style `:eyes:` over unicode emoji. `> [!CAUTION]` only for genuinely important warnings.
- **Vivid verbs** sparingly: `yeets`, `unskellys`, `bootstraps` — one per PR max.
- **Self-reference in third person** is fine occasionally (_"Carter's just helping us while our API previews are busted"_).

### 6. Scope guard

Check if the diff touches multiple unrelated domains (e.g., `api/` + `infra/` + `frontend/`). If yes, warn the user that scope might be too broad. Let them decide whether to split.

### 7. Codegen check

If the repo generates code (OpenAPI spec, API clients, schema types) and the diff touches its inputs (routers, schemas), verify the output is up to date:

- Run the repo's codegen command (check `justfile`, `package.json`, `Makefile`, or AGENTS.md)
- If it produces changes, warn the user and stage them.

### 8. Humanize the description

Run `/humanizer` on the PR body text. **Mandatory — never skip.**

### 9. Create the PR

- Default: ready (not draft)
- Draft if: user passes `--draft`, branch name contains `wip`, or user explicitly asks
- Push with `-u` flag if the branch has no upstream
- Use `gh pr create --title <title> --body "$(cat <<'EOF' ... EOF)"` — HEREDOC to preserve formatting

### 10. Open the PR in the browser

Run `gh pr view --web` immediately after creation. This grabs Carter's attention back — he's been heads-down while you worked, and the browser tab popping up is the handoff signal. **Mandatory — never skip.**

### 11. Arm post-creation monitors, then continue working

After the PR is open, set up two `Monitor` invocations and immediately move on to the next task. Do NOT block your turn polling. **No `sleep` in this house** — the Monitor tool is the only correct way to watch async PR state. Bare `sleep` loops in your own turn are banned.

#### CI/CD monitor (`persistent: true`)

Watches GitHub checks; emits one line per check as it resolves; runs for the session. Description: `"CI checks for PR #<N>"`.

**Do NOT exit on `all(.bucket!="pending")`** — `gh pr checks` returns the checks GitHub has _registered so far_. Early in a workflow run, only the fast checks (labelers, title lint) have been registered, all with terminal status; the slow ones (lint, test, review bots) appear later as pending. An early exit declares victory while the real work hasn't even started. The monitor must run for the session and let TaskStop / session-end clean it up.

Reference shape (adapt PR number; `<owner>/<repo>` comes from `gh repo view --json nameWithOwner -q .nameWithOwner`):

```bash
prev=""
while true; do
  s=$(gh pr checks <PR#> --json name,bucket,link 2>/dev/null || echo '[]')
  cur=$(jq -r '.[] | select(.bucket!="pending") | "\(.bucket | ascii_upcase) \(.name) \(.link)"' <<<"$s" | sort)
  comm -13 <(echo "$prev") <(echo "$cur")
  prev=$cur
  sleep 30
done
```

**On failure event:** stop, investigate, fix.

- Pull the failing job log: `gh run view <run-id> --log-failed`
- Diagnose the actual root cause — not a workaround
- Fix in a new commit and push
- Monitor will rearm against the new SHA automatically since the loop re-queries each iteration
- If the failure looks like a known flake (e.g. transient network), surface it and ask before rerunning — don't silently rerun

#### Review-comment monitor (`timeout_ms: 600000` — 10 min)

Polls both review-comment (inline diff) and issue-comment (general PR conversation) endpoints. Description: `"new comments on PR #<N>"`. **Emit the comment ID in each event** so the handler can react and reply.

```bash
since=$(date -u +%Y-%m-%dT%H:%M:%SZ)
OWNER_REPO=$(gh repo view --json nameWithOwner -q .nameWithOwner)
while true; do
  gh api "repos/$OWNER_REPO/pulls/<PR#>/comments?since=$since" \
    --jq '.[] | "REVIEW id=\(.id) \(.user.login) @ \(.path):\(.line // .original_line // 0): \(.body)"' 2>/dev/null || true
  gh api "repos/$OWNER_REPO/issues/<PR#>/comments?since=$since" \
    --jq '.[] | "ISSUE id=\(.id) \(.user.login): \(.body)"' 2>/dev/null || true
  since=$(date -u +%Y-%m-%dT%H:%M:%SZ)
  sleep 30
done
```

#### Comment triage — focus on actual bugs and problems

When a comment event fires, classify before acting:

- **Act now**: bug, wrong behavior, security issue, broken contract, missing test for a real edge case, factual error in the PR description
- **Hold for Carter**: nits, style preferences, naming bikeshed, "have you considered X" musings, requests for refactors out of scope, approvals/lgtm
- If unclear which bucket: leave it. Carter will weigh in.

#### Reaction-and-reply protocol

For every comment in the **Act now** bucket:

**1. React with 👀 immediately** so the author sees it's being worked on. Pick the endpoint that matches the comment type — `pulls/comments` for inline-diff (REVIEW events), `issues/comments` for general PR comments (ISSUE events):

```bash
# Inline review comment (REVIEW event)
gh api -X POST "repos/$OWNER_REPO/pulls/comments/<comment_id>/reactions" \
  -H "Accept: application/vnd.github+json" -f content=eyes

# General PR comment (ISSUE event)
gh api -X POST "repos/$OWNER_REPO/issues/comments/<comment_id>/reactions" \
  -H "Accept: application/vnd.github+json" -f content=eyes
```

**2. Fix it.** New commit, push.

**3. Reply on the thread with the fix.** The reply body MUST start with this frontmatter so reviewers can identify AI-generated responses at a glance:

```
_This is an AI generated response by the `/pr` skill_

----

<content>
```

**Blank lines around `----` are mandatory.** Without them, GitHub markdown parses the line above + `----` as a setext H2 heading, blowing up the rendered comment. The blank lines force it to render as `<hr>`.

Write `<content>` in Carter's technical writing voice — verb-first, fragment-heavy, em-dash asides, own the screwup dryly when applicable, no corporate voice. Same rules as the PR description body. Keep it tight: what was wrong, what you changed, link the fix commit.

For threaded replies on inline review comments:

```bash
gh api -X POST "repos/$OWNER_REPO/pulls/<PR#>/comments/<comment_id>/replies" \
  -H "Accept: application/vnd.github+json" \
  -f body="$(cat /tmp/reply.md)"
```

For replies to general PR comments (no native threading — post a new issue-comment that quotes/references the original):

```bash
gh pr comment <PR#> --body-file /tmp/reply.md
```

Always write the reply body to a file and pass via `-f body=@file` / `--body-file` — never inline heredoc; same shell-quoting reason as PR creation.

For comments in the **Hold for Carter** bucket: do NOT react with 👀 (that signals "I'm fixing this"). Leave them untouched.

#### Continuing work

Both monitors run alongside whatever's next on the plan. Don't gate the next task on them — that's the whole point. If the CI monitor or review monitor fires while you're mid-task, finish the current atomic step, then pivot.

### 12. Output

- Return the PR URL
- One-line summary: title, base branch, Linear ticket (if linked), draft status
- Note that monitors are armed: "watching CI + review comments (10 min)"

### Rules

- NEVER skip the humanizer, browser-open, or monitor-arming steps
- NEVER use bare `sleep` loops in your own turn to poll PR state — that's what `Monitor` is for
- NEVER auto-address nit comments or bikeshed — focus monitor reactions on real bugs and broken behavior
- NEVER reply to a review comment without the `_This is an AI generated response by the `/pr` skill_` frontmatter — reviewers need to spot AI replies at a glance
- NEVER skip the 👀 reaction on actionable comments — silence reads as "ignored"; the reaction is the "on it" signal
- NEVER create a PR without reading the full diff first
- NEVER guess at the "why" — derive it from Linear, Sentry, Slack, prior PRs, or the diff itself
- NEVER write "tech debt" or "cleanup" as the why without a real trigger
- NEVER exceed 200 words in the body without asking — default short, escalate only if a clear trigger applies
- If anything looks off (wrong branch, no commits ahead of base, uncommitted changes), stop and ask
