# Detection Rules

## What to flag

1. **`Text`/`Label`/etc. fed a `String`-typed variable instead of a literal.**
   SwiftUI's `Text(_ content: some StringProtocol)` overload renders plain `String` values **verbatim** — it does not localize them. This looks identical to the safe case at a glance, so it's the most commonly missed gap.

   ```swift
   // Flag this — displays verbatim, never localized
   let title: String = "Welcome back"
   Text(title)
   ```

   Suggested fix: wrap the source value in `String(localized:)` where it's defined, not at the call site:
   ```swift
   let title: String = String(localized: "Welcome back")
   Text(title)
   ```

2. **User-facing `String` text outside the View layer.**
   ViewModels, Models, thrown error descriptions, and alert message strings are plain `String` — they need explicit `String(localized:)` to become localizable and get extracted into the String Catalog.

   ```swift
   // Flag this
   var errorMessage: String { "Something went wrong. Please try again." }
   ```
   ```swift
   // Suggested fix
   var errorMessage: String { String(localized: "Something went wrong. Please try again.") }
   ```

3. **Legacy `NSLocalizedString(...)` calls.**
   Still functional and still extractable by Xcode, so this is a modernization nudge rather than a hard gap. Suggest migrating to `String(localized:)` for consistency with String Catalog tooling.

4. **Manual string concatenation/interpolation for user-facing text.**
   Breaks grammar and pluralization in languages with different word order or plural rules.

   ```swift
   // Flag this
   Text("You have " + String(count) + " items")
   ```
   ```swift
   // Suggested fix — let the String Catalog handle the variation
   Text("^[\(count) item](inflect: true)")
   ```

5. **Stray `.strings` / `.stringsdict` files coexisting with a `.xcstrings` String Catalog.**
   Two sources of truth let keys silently diverge. Recommend consolidating into the String Catalog.

## What NOT to flag

- **Direct string literals passed straight into `Text("...")`, `Label("...", systemImage:)`, `Button("...")`, etc.** These already resolve to `LocalizedStringKey` (via `ExpressibleByStringLiteral`), and Xcode's build-time extraction picks them up into the String Catalog automatically. Flagging these produces pure noise.
- Non-user-facing strings: accessibility identifiers used for UI testing, analytics event names, log/debug strings (`Logger`, `print`), URLs, hex colors, font names, dictionary/JSON keys.
