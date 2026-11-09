---
name: do
description: Handed a Linear ticket URL or ID, gather everything needed before work starts — ticket details, relevant project memory, related code. Triggers on "/do <linear-url-or-id>", "do this ticket", "spin up context for <TICKET-ID>".
---

Gather full context for a Linear ticket so work can start immediately — no manual round-trip through Linear, memory, and the codebase first.

## Workflow

1. **Fetch the ticket**
   Parse the ID out of the URL (or take it directly, e.g. `ENG-1234`). Call the Linear MCP `get_issue` tool. Pull title, description, status, labels, project, assignee, acceptance criteria, comments, linked/blocking issues.

2. **Check memory**
   Your memory index (`MEMORY.md`) is already loaded in context. Scan it for entries whose description overlaps the ticket's title, project name, or key nouns (e.g. ticket mentions "billing export" → check `project_billing_export_*`). Read the matching memory files. Skip silently if nothing's relevant — don't force a match.

3. **Check the code**
   Grep for the ticket ID itself (sometimes referenced in commit messages, comments, or branch names from prior partial work) and for entities named in the ticket (function/class/table names, file paths). If the ticket points at a folder with an `AGENTS.md`, read it.

4. **Surface, don't decide**
   This skill gathers — it doesn't plan or implement. End with a tight briefing and stop.

## Output format

```
# <TICKET-ID>: <title>
<status> · <project> · <url>

## What it's asking for
<2-4 sentence distillation of description + acceptance criteria>

## Relevant memory
- [[memory-name]]: <one-line why it's relevant>
(or: "Nothing on file" if no match)

## Relevant code
- <file:line> — <why>
(or: "No existing references found")

## Open questions
<anything the ticket leaves ambiguous that'll need a call before/during implementation>
```

## Fallbacks

- **Ticket ID doesn't resolve** — say so, don't guess at a similar-looking ticket.
- **No memory hits** — common case, skip the section content but keep the header so the gap is visible rather than silently omitted.
- **Ticket references a PR or branch already** — note it; that's probably where prior work left off.
