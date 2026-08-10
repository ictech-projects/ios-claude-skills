---
name: design-token-usage
description: Reviews SwiftUI code for raw color/font literals that bypass the project's own design-token system (asset-catalog color sets, typography extensions), and — with approval — mechanically replaces matched literals with the correct token. Use when auditing design-system compliance, before merging new views, or cleaning up accumulated raw-literal drift.
license: MIT
argument-hint: "[scope]"
metadata:
  author: ICT
  version: "1.0"
---

> **ICT iOS Team** — This skill reviews code and, with approval, mechanically replaces raw color/font literals with the project's existing design tokens. It never invents a new token, and never guesses when a literal has no clear match — ambiguous cases are always flagged for design follow-up, not silently resolved.

## 1. Ask for scope

Ask the developer which scope to review, and wait for their answer before scanning anything:

- **Diff scope** — only the current commit / changes not yet merged into the base branch.
- **Full scope** — the entire project.

## 2. Locate the token source

Find the project's actual design-token definitions — don't assume names, discover them:

- Color tokens: an asset catalog (`.xcassets`) with named color sets, typically organized into semantic groups (brand palette, neutral scale, semantic states like danger/success/warning/info, each often with role variants like border/hover/main/pressed/surface).
- Font/typography tokens: a `Font` extension or dedicated typography file defining named text styles, often backed by bundled custom font files.

If neither is found, tell the developer no token system exists yet and stop — there's nothing to enforce compliance against.

## 3. Scan

Scan the chosen scope using the rules in `references/detection-rules.md`. In summary, flag:

1. `Color.red/white/black/gray/...` and any other named-literal `Color` case.
2. `Color(red:green:blue:)` / `UIColor(red:green:blue:alpha:)` and hex-string color initializers.
3. `.font(.system(size:...))` and other literal font constructions that bypass the typography tokens.
4. **Never flag** a literal already wrapped in the project's token accessor (e.g. `Color.brandPrimary`, `.font(.baseBody)`) — that's correct usage, not a violation.

## 4. Map each finding to a token

Follow `references/token-mapping-strategy.md` to propose a replacement for each hit:

1. Match by semantic role first (is this literal acting as a background, a border, a text color, a state indicator?).
2. If no clear role match, match by nearest value (closest RGB distance for colors, closest point size for fonts).
3. If neither produces a confident match, mark the finding **unmatched** — never guess a "close enough" token just to close out the finding.

## 5. Report findings

Present findings as a table:

| File | Line | Literal | Suggested Token | Confidence |
|---|---|---|---|---|

Follow it with a summary line: total files scanned, total findings, matched vs. unmatched count.

## 6. Wait for approval before changing anything

Do not replace any literal until the developer explicitly approves — and approval can be per-finding, not just all-or-nothing, since confidence varies across the table.

## 7. Apply (only after approval)

Replace only the approved literal with its exact matched token accessor. Never rename, restructure, or add new entries to the token source itself — this skill consumes the existing token system, it doesn't extend it.

## 8. Final summary

Always end the run with: what was scanned, how many literals were replaced, and the full list of **unmatched** findings left for a human/design decision — these are not failures, just cases this skill correctly declined to guess on.

---

## Core Instructions

- Never invent a new token or edit the token source (asset catalog, typography file) — this skill only consumes what already exists.
- Never guess a replacement when the mapping is ambiguous — report it as unmatched instead.
- Never apply a fix without explicit approval from the developer.
- Always state the detected scope (diff vs full project) before scanning.
- Always separate matched from unmatched findings in the final summary.

---

## References

- `references/detection-rules.md` — literal patterns to flag, with real code examples of what counts as a violation vs. correct token usage.
- `references/token-mapping-strategy.md` — how to map a raw literal to the nearest token, and when to mark a finding unmatched instead.
