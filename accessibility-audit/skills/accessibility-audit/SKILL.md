---
name: accessibility-audit
description: A dedicated, scope-selectable sweep for missing VoiceOver labels/traits, Dynamic Type hazards, and accessibilityIdentifier coverage for UI testing — and, with approval, mechanically applies fixes using real adjacent copy. Use for a standalone accessibility pass across a diff or the whole project, not as one bullet inside a general code review.
license: MIT
argument-hint: "[scope]"
metadata:
  author: ICT
  version: "1.0"
---

> **ICT iOS Team** — This skill reviews code for accessibility gaps and, with approval, mechanically applies fixes following `${CLAUDE_SKILL_DIR}/references/audit-rules.md` and `${CLAUDE_SKILL_DIR}/references/identifier-convention.md`. It never invents placeholder copy — every label comes from real, adjacent text already in the View.

## 1. Ask for scope

Ask the developer which scope to review, and wait for their answer before scanning anything:

- **Diff scope** — only the current commit / changes not yet merged into the base branch.
- **Full scope** — the entire project.

Also ask whether to include **accessibilityIdentifier coverage** for future UI-testing support (step 5) — some reviews only want the VoiceOver/Dynamic Type pass.

## 2. Scan

Scan the chosen scope using `${CLAUDE_SKILL_DIR}/references/audit-rules.md`. In summary, flag:

1. Interactive elements with no VoiceOver label — icon-only buttons/menus with no text label, `onTapGesture()` without `.accessibilityAddTraits(.isButton)`.
2. Images with unclear or missing VoiceOver readings — not marked decorative/hidden, and not given an `accessibilityLabel()` when they convey information.
3. Hardcoded font sizes that block Dynamic Type scaling.
4. Color-only differentiation with no `.accessibilityDifferentiateWithoutColor` fallback.
5. If accessibilityIdentifier coverage was requested: interactive elements with no `accessibilityIdentifier` at all, needed for any future XCUITest coverage.

This skill exists because these checks are easy to skip when they're one bullet
among many in a general code review (see `swiftui-reviewer/references/accessibility.md`,
whose checklist this skill's rule 1-4 builds on) — running it standalone means
accessibility gets its own pass, not a footnote.

## 3. Report findings

Present findings as a table:

| File | Line | Element | Issue | Severity |
|---|---|---|---|---|

Severity: **high** = invisible or unreachable via VoiceOver entirely; **medium** = reachable but with a poor/unclear reading; **low** = missing identifier only (doesn't affect real users, only future test coverage).

Follow it with a summary line: total files scanned, total findings by severity.

## 4. Wait for approval before changing anything

Do not modify any code until the developer explicitly approves — approval can be per-finding, not just all-or-nothing.

## 5. Apply (only after approval)

Apply the approved fixes:

- **Labels** — pull the label text from real, already-visible adjacent copy in the same View (e.g. a nearby `Text`, the button's own visual label). Never invent placeholder text like `"Button"` or `"Icon"`.
- **Traits/hidden/differentiate-without-color** — apply the specific modifier `${CLAUDE_SKILL_DIR}/references/audit-rules.md` calls for; don't over-apply beyond what the finding actually needs.
- **Identifiers** — follow `${CLAUDE_SKILL_DIR}/references/identifier-convention.md` to detect an existing naming convention in the project first; only propose a new convention if none exists, and confirm it with the developer before applying it broadly.

## 6. Final summary

Report what was scanned, what was fixed vs. left for developer decision (e.g. an image where it's genuinely unclear whether it's decorative), and — if identifiers were added — the convention used.

---

## Core Instructions

- **Never invent placeholder label text.** Every `accessibilityLabel`/`accessibilityHint` must come from real copy already present in the View or its immediate context.
- **Never apply a fix without explicit approval from the developer.**
- **Detect an existing `accessibilityIdentifier` convention before proposing one.** Don't introduce a second, competing naming scheme.
- **Always state the detected scope and whether identifier coverage was included** before scanning.
- **This is a standalone sweep, not a replacement for `swiftui-reviewer`.** If a review surfaces other, non-accessibility issues (deprecated APIs, data flow), leave those to `swiftui-reviewer` — don't scope-creep.

---

## References

- `${CLAUDE_SKILL_DIR}/references/audit-rules.md` — the full check list (VoiceOver, images, Dynamic Type, color differentiation, identifier coverage), building on `swiftui-reviewer`'s existing accessibility checklist.
- `${CLAUDE_SKILL_DIR}/references/identifier-convention.md` — how to detect an existing `accessibilityIdentifier` naming convention in a project, or propose one when none exists.
