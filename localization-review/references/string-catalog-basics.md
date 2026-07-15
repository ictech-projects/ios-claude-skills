# String Catalog Basics

String Catalogs (`.xcstrings`, introduced in Xcode 15) are Apple's current recommended localization format, replacing the `.strings` / `.stringsdict` pair.

- A single JSON-backed file per catalog, editable in Xcode's dedicated String Catalog UI.
- Xcode auto-populates it by scanning source code at build time for known localizable APIs: `Text("...")` and other SwiftUI initializers accepting `LocalizedStringKey`, `String(localized:)`, and `NSLocalizedString(...)`.
- Each key carries a **per-locale state**: `New` (not yet translated), `Needs Review` (source changed since translation), or `Translated`.
- Plural and device-variation rules are expressed natively in the catalog UI — no separate `.stringsdict` file needed.

## Why this skill enforces String Catalog over the legacy approach

- One file per catalog instead of a `.strings`/`.stringsdict` pair per language — fewer places for keys to drift out of sync.
- Native per-key translation-state tracking makes untranslated debt visible directly in Xcode, instead of relying on manual audits of separate `.strings` files.
- Plural rules are built in, removing a common source of hand-rolled, grammar-breaking string concatenation.

## Extraction is build-time, not static

Because Xcode populates the catalog by scanning code during a build, a source change (e.g. wrapping a string in `String(localized:)`) does not appear in the `.xcstrings` file until the project is built in Xcode. This skill's code-level fixes and its `.xcstrings` cross-check are therefore two separate signals:

- **Code fix applied** — the call site now routes through a localizable API.
- **Catalog entry present** — Xcode has actually extracted the key into `.xcstrings`.

A finding can have the first without the second yet — that's expected right after a fix is applied, not a bug. Report it as "wrapped in code — build in Xcode to extract into the catalog," not as an error.

## Out of scope: translation

This skill never generates translation text for non-source locales. A key showing `New` for French, German, Japanese, etc. is the *correct* state immediately after extraction — it is the signal for a human translator or localization vendor to pick up the work, not something to be auto-filled.
