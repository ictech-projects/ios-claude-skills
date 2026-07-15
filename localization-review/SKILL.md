---
name: localization-review
description: Reviews Swift/SwiftUI code for missing or incorrect String Catalog (.xcstrings) localization coverage, flags legacy NSLocalizedString/.strings usage, and — with approval — mechanically wraps flagged strings in String(localized:). Use when auditing localization coverage, before merging user-facing string changes, or migrating away from legacy .strings files.
license: MIT
argument-hint: "[scope]"
metadata:
  author: ICT
  version: "1.0"
---

> **ICT iOS Team** — This skill reviews code and, with approval, mechanically converts it to route through Xcode's String Catalog (`.xcstrings`). It is a checker/converter only — it never generates translations for other languages.

## 1. Ask for scope

Ask the developer which scope to review, and wait for their answer before scanning anything:

- **Diff scope** — only the current commit / changes not yet merged into the base branch.
- **Full scope** — the entire project.

## 2. Locate the String Catalog

Find the project's `.xcstrings` file(s). If none exist, tell the developer no String Catalog was found and ask whether to proceed — findings can still surface, but nothing can be cross-checked against a catalog until one exists.

## 3. Scan

Scan the chosen scope using the rules in `references/detection-rules.md`. In summary:

1. `Text`/`Label`/etc. fed a `String`-typed variable instead of a literal — the primary, sneakiest gap (SwiftUI's `Text(_ content: some StringProtocol)` overload renders plain `String` values verbatim, not localized).
2. User-facing `String` text living outside the View layer (ViewModels, Models, error/alert messages) not wrapped in `String(localized:)`.
3. Legacy `NSLocalizedString(...)` calls — flagged as a modernization suggestion, not a hard gap.
4. Manual string concatenation/interpolation used to build user-facing text.
5. Stray `.strings`/`.stringsdict` files coexisting with a `.xcstrings` catalog.
6. **Never flag** string literals passed directly into `Text("...")`, `Label("...", ...)`, etc. — these already resolve to `LocalizedStringKey` and Xcode auto-extracts them. Flagging these is noise.

## 4. Report findings

Present findings as a table:

| File | Line | Issue | Suggested Action |
|---|---|---|---|

Follow it with a summary line: total files scanned, total findings, legacy API count.

## 5. Cross-check against the String Catalog

Read `references/string-catalog-basics.md` to understand how extraction works, then cross-reference flagged and existing localizable strings against the `.xcstrings` file itself:

- Confirm each expected key is actually present with the correct source-language value.
- Report mismatches plainly — e.g. a string wrapped in code but not yet extracted into the catalog needs an Xcode build to pick it up. This is expected right after a fix, not an error.

## 6. Wait for approval before changing anything

Do not wrap or modify any code until the developer explicitly approves. Show the findings table first and wait.

## 7. Apply (only after approval)

Apply only the mechanical fix: wrap the flagged literal/expression in `String(localized:)`, keeping the existing source-language text exactly as-is. Never invent new copy, reword existing copy, or generate translations.

## 8. Final summary

Always end the run with:

- What was scanned (scope) and what was found/fixed.
- The String Catalog cross-check result.
- An explicit disclaimer: this is a review/checker only. It does not produce translations — any new key will show as `New` for every non-source locale in the String Catalog until a human translator or localization vendor fills it in.

---

## Core Instructions

- Never generate or guess translations into other languages — always out of scope.
- Never flag a direct string literal passed straight to a SwiftUI `Text`/`Label` initializer — it's already correctly localizable.
- Never apply a fix without explicit approval from the developer.
- Always state the detected scope (diff vs full project) before scanning.
- Always end with the String Catalog cross-check and the "checker only, not a translator" disclaimer.

---

## References

- `references/detection-rules.md` — full detection rule list with code examples (what to flag, what not to flag).
- `references/string-catalog-basics.md` — how Xcode String Catalogs and build-time extraction work, and why translation generation is out of scope.
