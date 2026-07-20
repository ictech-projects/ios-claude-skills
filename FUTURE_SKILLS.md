# Future Skills — Candidate Ideas

Not implemented yet. Captured from a brainstorming session (2026-07-20) grounded in a survey of `hris-ios`, to inform which skill to build next after `prepare-release`. Revisit and prune as the codebase evolves — these are candidates, not commitments.

## High signal (repeated house-style patterns with no existing skill coverage)

### 1. Error-handling / ViewModel error-mapper review
Every ViewModel hand-rolls an `isError`/`errorMessage` + `withAnimation { isError = true }` pattern (46+ files in `hris-ios`), paired with a per-feature `*ErrorMapper` that special-cases HTTP 401 → `isExpired` → manual token erase → logout. It's copy-pasted per feature with no shared abstraction and no automatic token-refresh/retry. Strongest candidate — most duplicated, most bug-prone (401 handling is inconsistent per feature today).

Evidence: `ProfileViewModelErrorMapper.swift`, `MultiFactorAuthenticationErrorMapper.swift`, `Moya+Request.swift` (status-code → `ErrorResponse`/`MoyaError` mapping).

### 2. Accessibility review
Zero uses of `accessibilityLabel`/`accessibilityIdentifier`/`accessibilityHint` anywhere in `HRIS/Feature` or `HRIS/Common` — a flat-out gap, not a house-style inconsistency. A skill that audits/adds a11y annotations on new or existing SwiftUI views would be high value and low controversy. (Note: `swiftui-view-gen`/`swiftui-reviewer` already have `references/accessibility*.md` — check whether this should be a standalone skill or folded into one of those before building it separately.)

### 3. Feature scaffold (navigation + repository DI wiring)
Adding a screen means: add a `NavigationPage` case + a matching switch arm in the single giant `.navigationDestination` switch in `HRISApp.swift`, plus repeating the default-arg repository DI init pattern (`init(remoteDataSource: some XRemoteDataSource = XDefaultRemoteDataSource())`) in every new Repository/ViewModel. A "scaffold a new feature" skill that wires navigation + DI + basic ViewModel/View stubs together (rather than three separate skills) would reduce multi-step manual wiring that's easy to get subtly wrong.

Evidence: `HRIS/Common/Helper/NavigationHelper/NavigationManager.swift`, `HRISApp.swift`'s `NavigationPage` switch, `CrashReportDefaultRepository.swift`.

## Lower signal / optional

### 4. Design token usage
Enforce use of existing `Color.xcassets` tokens (Success/Danger/Warning/Info/Neutral/Grey/Surface/BrandPallete) and `Font+baseStyle.swift` instead of raw colors/fonts in new views. Overlaps `swiftui-view-gen`/`swiftui-reviewer` — likely folds into those rather than becoming standalone.

## Explicitly not worth a skill (yet)

No strong signal found for: DI-container abstraction (none exists — "poor-man's DI" via default-arg protocol inits is consistent enough as-is), feature flags / remote config (none in use), deep-linking (single hard-coded push-notification target, no generic router), or CI conventions beyond what `prepare-release` now covers (Xcode Cloud only, no fastlane, minimal `ci_scripts/`).

## Suggested order

1. Error-handling / ViewModel error-mapper review — most duplicated, most bug-prone.
2. Accessibility review — pure gap-filler, no ambiguity in scope.
3. Feature scaffold — higher-value but more of a generator than a reviewer; different shape of work, do after the above two are validated.
4. Design token usage — reassess whether this should just be an addition to `swiftui-view-gen`/`swiftui-reviewer` instead of standalone.
