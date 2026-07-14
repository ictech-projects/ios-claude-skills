# 📄 [CLAUDE] MR Template (Overview + Ticket Link)

## PR Body Template

Every PR body must contain at minimum these three sections, in this order:

```markdown
## Overview

<1-3 sentence summary of what changed and why, derived from the commits/diff>

## Ticket

<ticket URL the developer provided>

## Checklist

- [ ] Pull Request title follows SOP (`[TICKET]<TAG>Title`)
- [ ] First commit follows SOP (`[TICKET]<TAG>Title`)
- [ ] Build succeeded
- [ ] Unit tests pass

---
> ⚠️ Please **squash merge** this PR to keep a single commit on the development branch.
```

- **Overview** — summarize the actual change, not the commit list verbatim. Base it on the diff and commit messages between the base branch and the current branch.
- **Ticket** — a bare link (or `[TICKET-ID](url)` if a ticket ID is available) to the URL the developer pasted. Never fabricate a ticket URL — if the developer says there is none, use `NO-BTS` conventions (see below) and state "No ticket — see commit message" instead of a link.
- **Checklist** — always these four items, in this order:
  - **Pull Request title follows SOP** and **First commit follows SOP** — check these off automatically only once the workflow has actually validated the title/first commit against the format below. Never check them off without validating.
  - **Build succeeded** and **Unit tests pass** — the skill cannot verify these itself. Ask the developer whether the build succeeded and unit tests pass, and only check off what they confirm. Leave unchecked (`[ ]`) if they haven't verified yet — don't assume a pass.
- The squash-merge note is always appended, regardless of what else is in the body.

Additional sections (e.g. `## Screenshots`, `## Testing`) may be appended after these three, but Overview, Ticket, and Checklist must always be present and in this order.

---

## Commit Message Format (PR Title Source)

The **first commit ahead of the base branch** becomes the PR title verbatim. It must match:

```
[<TICKET>]<<TAG>><Title>
```

- `<TICKET>` — either a ticket key like `HA-777`, `HA-778`, `DEVT-664` (project prefix + dash + number), or the literal `NO-BTS` when there is no backlog ticket.
- `<TAG>` — an uppercase word describing the type of change, e.g. `FEATURE`, `FIX`. Other uppercase tags (e.g. `CHORE`, `REFACTOR`, `DOCS`) are also accepted — the format only requires it to be uppercase, immediately following the ticket brackets with no space.
- `<Title>` — a short human-readable description of the change.

**Valid examples:**

```
[HA-777]<FEATURE>As a user, I want to show login screen.
[HA-778]<FIX>Fix issue on login screen.
[NO-BTS]<FEATURE>As a user, I want to show login screen.
[NO-BTS]<FIX>Fix issue on login screen.
```

**Regex:** `^\[(NO-BTS|[A-Z][A-Z0-9]*-\d+)\]<[A-Z]+>.+`

If the first commit does not match this format, it must be corrected (see the reword procedure in the parent `SKILL.md`) before the PR is opened, since the PR title is derived directly from it.
