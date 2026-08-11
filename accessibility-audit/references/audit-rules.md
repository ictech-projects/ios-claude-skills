# Audit Rules

This builds on `swiftui-reviewer/references/accessibility.md`'s checklist —
running it standalone means these get a dedicated pass with severity ranking
and optional identifier coverage, instead of being one bullet among eight in
a general review.

## Flag

1. **Icon-only interactive elements with no text label.**
   ```swift
   // flag — VoiceOver reads nothing useful
   Button(action: myAction) { Image(systemName: "plus") }

   // correct
   Button("Add", systemImage: "plus", action: myAction)
       .labelStyle(.iconOnly) // visual stays icon-only; VoiceOver still gets "Add"
   ```
2. **`onTapGesture()` without a button trait.**
   ```swift
   // flag
   Text("Tap me").onTapGesture { ... }

   // correct
   Text("Tap me")
       .onTapGesture { ... }
       .accessibilityAddTraits(.isButton)
   ```
3. **Images with unclear or missing VoiceOver readings.** An image conveying
   information needs a label; a purely decorative one needs to be hidden.
   ```swift
   // flag — unclear what "newBanner2026" means to VoiceOver
   Image(.newBanner2026)

   // correct, decorative
   Image(.newBanner2026).accessibilityHidden(true)

   // correct, informative
   Image(.receipt).accessibilityLabel("Receipt")
   ```
4. **Hardcoded font sizes blocking Dynamic Type.**
   ```swift
   // flag
   Text("Total").font(.system(size: 14))

   // correct
   Text("Total").font(.body)
   ```
5. **Color-only differentiation.** If color is the only signal distinguishing
   states, add a non-color fallback for `.accessibilityDifferentiateWithoutColor`.
6. **Missing `accessibilityIdentifier` on interactive elements** — only flag
   when the developer opted into identifier coverage for this run. This
   doesn't affect real users; it only matters for future XCUITest coverage,
   so it's a separate, lower-severity category from the rest.

## Never flag

- Elements that already carry a correct label/trait/hidden modifier.
- Purely internal/debug views never shown to real users (e.g. behind a
  developer-only flag) — still flag if the project ships them to production
  builds, since VoiceOver users on that build are affected regardless.

## Worked example (hris-ios)

A full-project scan found **zero** matches for
`accessibilityIdentifier|accessibilityLabel|accessibilityHint|.accessibility(`
anywhere under `HRIS/` — every one of the checks above applies fleet-wide, not
just to isolated files. There's also no XCUITest target
(`HRISTests/TESTING.md` explicitly says not to bother with UI tests), so
identifier coverage here is purely forward-looking groundwork, not fixing an
existing test suite.
