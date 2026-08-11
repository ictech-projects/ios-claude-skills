# Identifier Convention

## Detect first

Before proposing anything, search the project for existing
`.accessibilityIdentifier(...)` usage. If any exist, infer the pattern
already in use (e.g. dot-separated scopes, camelCase, a raw string constant
per screen) and match it exactly — don't introduce a second, competing style.

## Propose only if none exists

If the project has zero existing identifiers (as was the case in the hris-ios
survey — a fleet-wide gap, not an isolated one), propose:

```
<Feature>.<Element>.<Role>
```

- `<Feature>` — the screen/feature name (matching the `Feature/<Name>` folder
  or the View's own name minus `View`).
- `<Element>` — what the element is (`submitButton`, `emailField`,
  `errorBanner`).
- `<Role>` — optional, only when the same element type repeats in a list
  (`row.0`, `row.1`) or needs disambiguating from a similar element elsewhere
  on the same screen.

```swift
// e.g. for ContactUsView
submitButton.accessibilityIdentifier("ContactUs.submitButton")
messageField.accessibilityIdentifier("ContactUs.messageField")
```

**Always confirm the proposed convention with the developer before applying
it broadly** — this is a one-time decision that every future screen will
follow, so it's worth a deliberate sign-off rather than a silent default.

## Scope discipline

Only add identifiers to elements a human would plausibly need to reference
from a future UI test (buttons, fields, key status indicators) — don't
blanket every `Text` and `Image` in a screen with an identifier just because
it's technically possible. That's noise, not test infrastructure.
