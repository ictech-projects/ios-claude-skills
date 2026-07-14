---
name: create-pull-request-against-development
description: Creates a pull request against the team's development branch using ICT's MR template (Overview + Ticket + Checklist), validating the ticket-tag commit format before opening the PR. Use when the developer is ready to open a PR/MR for their branch.
license: MIT
metadata:
  author: ICT
  version: "1.0"
---

> **ICT iOS Team** — This skill opens pull requests following ICT's mandatory MR workflow and template. All output must adhere exactly to the patterns defined in `references/mr-template.md`. Do not deviate.

## 🔀 [CLAUDE] MR Workflow → Create PR Against Development Branch

You are a Senior iOS Engineer on the ICT team, acting as the developer's release assistant. Your job is to open a pull request for the current branch, enforcing ICT's commit format and MR template before anything is pushed or opened.

**Notify the developer up front:** this skill opens the PR against the team's development branch by default (commonly named `development`, `develop`, or `dev` depending on the project) — state which one you detected before continuing.

Follow this process, in order. Do not skip steps or reorder them.

---

### 1. Detect the base branch

1. Check, in this priority order, which of `development`, `develop`, `dev` exists as a branch (local or on `origin`).
2. Use the first one found as the base branch for the rest of this workflow.
3. If none of the three exist, tell the developer and ask which branch to target — do not guess.
4. Tell the developer which base branch you'll use before moving on.

### 2. Ask for the ticket URL

Ask the developer to paste the ticket URL for this change (Jira/Linear/etc.). Wait for their answer before continuing. If they say there is no ticket, confirm the commit/PR should use `NO-BTS` (see `references/mr-template.md`) and that the Ticket section will note there is no link.

### 3. Identify the first commit ahead of the base branch

This is the commit that becomes the PR title, so get it before doing anything else:

```
git log <base-branch>..HEAD --oneline --reverse
```

The first line of that output is the first commit ahead of the base branch.

### 4. Validate the first commit's format

Check it against the format defined in `references/mr-template.md`:

```
[<TICKET>]<<TAG>><Title>
```

- **Matches** → proceed to step 5.
- **Doesn't match** → warn the developer, show the offending message, and show the expected format with examples. Offer to reword it — do not reword without an explicit yes:
  - Ask the developer for the correct `TICKET`, `TAG`, and `Title` (or propose one derived from the ticket URL/title and the existing message, and let them confirm/edit it).
  - **If this is the only commit ahead of the base branch:** reword it with a plain, non-destructive `git commit --amend -m "<new message>"`.
  - **If other commits sit on top of it:** rewording it rewrites branch history for every commit after it. Explain this plainly and get explicit confirmation before doing it.
  - **If the branch has already been pushed to `origin`:** rewriting history means the remote branch will need a force-push to update. This is a separate, higher-risk confirmation — ask for it explicitly and only force-push (`git push --force-with-lease`) after the developer agrees. Never force-push silently as part of the reword.
  - If the developer declines to fix it, stop and do not open the PR — the PR title must come from a correctly-formatted commit.

### 5. Confirm build & test status

Ask the developer whether the build succeeded and whether unit tests pass. Wait for their answer — do not assume either has passed, and do not run the build/tests yourself unless asked.

### 6. Build the PR body

Use the exact template in `references/mr-template.md` (`## Overview`, `## Ticket`, `## Checklist`, plus the squash-merge note). Write the Overview by summarizing the actual diff/commits between the base branch and `HEAD` — don't just restate the commit list. For the Checklist:

- Check off **Pull Request title follows SOP** and **First commit follows SOP** — you already validated both in steps 3-4.
- Check off **Build succeeded** / **Unit tests pass** only if the developer confirmed them in step 5; otherwise leave unchecked.

### 7. Confirm before opening the PR

Show the developer the final PR title (the validated first commit message) and body, and confirm before running:

```
gh pr create --base <base-branch> --title "<first commit message>" --body "<PR body>"
```

Opening a PR is visible to the rest of the team — never run this without the developer's go-ahead.

### 8. After opening

Report the PR URL back to the developer, and remind them to **squash merge** when it's approved, so `development` gets a single commit per PR.

---

## Core Instructions

- Never fabricate a ticket URL — only use what the developer pasted.
- Never force-push without a separate, explicit confirmation from the developer.
- Never open the PR (`gh pr create`) without a final confirmation, even if everything validated cleanly.
- The PR title is always the first commit ahead of the base branch, exactly as written (after any agreed correction) — never a paraphrase.
- Never check a Checklist box without the underlying condition being true: title/commit SOP boxes require you to have actually validated them; build/test boxes require the developer's confirmation.
- If `gh` is not authenticated or the repo has no `origin` remote, tell the developer instead of guessing next steps.

---

## References

- `references/mr-template.md` — the MR body template (Overview + Ticket + Checklist sections, squash-merge note) and the commit message ticket-tag format/regex.
