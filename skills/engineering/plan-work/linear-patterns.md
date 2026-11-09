# Linear Patterns Reference

## Ticket Body Template

Use this template for ALL tickets created by /plan-work.

```markdown
## Why this is needed
[A short paragraph capturing who feels this, what they can't do today, and the cost of not doing it. Name a concrete user/workflow/team and the consequence of inaction — "tech debt" or "would be nice" alone is not enough.]

## Context
[Background: link to parent project, prior art, or relevant decisions]

## Implementation
- Architecture decisions and approach
- Key file paths to modify/create
- Patterns and conventions to follow (reference AGENTS.md)
- Integration points with other services/packages

## Acceptance Criteria
- [ ] Specific, testable criteria
- [ ] Each criterion is independently verifiable
- [ ] Include negative cases where relevant

## Verification Strategy
- Existing test commands and direct runtime checks
- Manual browser verification if UI is involved
- High-value automated tests and the observable behavior they exercise
- Screenshot requirements for UI changes

## Dependencies
- Blocks: [ticket IDs this unblocks when complete]
- Blocked by: [ticket IDs that must complete first]
- External: [APIs, services, or config changes needed]
```

## Where grilled decisions go

The Phase 2/3 grilling produces decisions, sharpened terminology, rejected alternatives, and the scenarios that pinned them down. **All of it lands in the ticket body** — this skill does not split plan decisions into ADRs or a CONTEXT.md.

- **Why** a chosen approach beat the alternatives → `Implementation` (lead with the decision, name the rejected option and why).
- **Canonical terms** sharpened during grilling (which entity "account"/"user"/"deal" actually means here) → `Context`, stated once so the implementer inherits the precise language.
- **Scenarios** that exposed an edge case → fold the resulting requirement into `Acceptance Criteria` and its verification into `Verification Strategy`.
- **Constraints not visible in code** (compliance, SLO, partner contract) → `Context`.

A future reader should be able to reconstruct *why this looks the way it does* from the ticket alone, without hunting for a separate decision record.

## Multi-Ticket Additions

When a ticket is part of a multi-PR sequence, prepend this to the Context section:

```markdown
**PR {N} of {total}** in [{project name}]({project URL})

**Previous PR delivers:** [what the prior ticket/PR provides that this builds on]
**This PR enables:** [what the next ticket/PR needs from this one]
```

## Project Structure (Mode C)

When creating a new Linear project:

- **Name:** concise, descriptive (e.g. "User onboarding redesign", not "Work on onboarding stuff")
- **Description:** problem statement, then goal, then PR sequence:
  ```
  **Problem:** [what is broken/missing/inadequate]
  **Goal:** [what this project delivers]

  PR sequence:
  1. [Ticket title] - [one-line summary]
  2. [Ticket title] - [one-line summary]
  ...
  ```
- **Team:** the team chosen in Phase 5 setup
- **Status:** Backlog (unless user specifies otherwise)

## Ticket Ordering

Tickets in a project must be ordered by dependency:
1. Foundation/schema changes first
2. Core logic/API second
3. UI/integration third
4. Test work that requires the full stack last

Set blocking relationships between tickets so the dependency graph is explicit.

## Labels

Query `list_issue_labels` to find existing labels. Common patterns:
- Scope labels: `backend`, `frontend`, `infra`
- Type labels: `bug`, `feature`, `tech-debt`
- Don't invent labels — only use what already exists in Linear

## Assignee

- Default: leave unassigned (cloud agents or team lead will assign)
- If user specifies an assignee during Q&A, set it
- For mode B (existing project): match the project's existing assignment patterns
