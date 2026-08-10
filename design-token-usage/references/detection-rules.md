# Detection Rules

## Flag

1. **Named-literal colors** — `Color.red`, `Color.white`, `Color.black`, `Color.gray`,
   `Color.blue`, etc., and their `UIColor` equivalents, including with
   `.opacity(...)` chained on.
   ```swift
   // flag
   .background(Color.gray.opacity(0.1))
   Color.black.opacity(0.5)
   ```
2. **Component-literal colors** — `Color(red:green:blue:)`,
   `UIColor(red:green:blue:alpha:)`, and hex-string initializers.
   ```swift
   // flag
   return Color(red: 0.9, green: 0.71, blue: 0)
   Color(uiColor: UIColor(red: 0.92, green: 0.94, blue: 0.97, alpha: 1))
   ```
3. **Literal font construction** — `.font(.system(size:...))`, and any other
   inline font built from a raw point size instead of a named typography token.
   ```swift
   // flag
   .font(.system(size: 14, weight: .regular))
   .font(.system(size: 24, weight: .bold))
   ```

## Never flag

- A literal already routed through the project's token accessor:
  ```swift
  // correct — do not flag
  Color.brandPrimary
  Color.neutral40
  .font(.baseBody)
  ```
- `Color.clear` and `.opacity(0)` — used for spacer/hit-target purposes, not a
  visible-color decision, so there's no token to match it to.
- Colors/fonts inside `#Preview` blocks or test fixtures — these aren't shipped
  UI and flagging them is noise.

## Worked examples (hris-ios)

Real hits found in a full-project scan, none matched to any of the app's
existing `Color.xcassets` tokens (`BrandPallete`, `Neutral10`–`Neutral100`,
`Danger/Success/Warning/Info` × `Border/Hover/Main/Pressed/Surface`):

- `HRIS/Common/Components/Button/MainClockButton.swift:14` —
  `Color(uiColor: UIColor(red: 0.92, green: 0.94, blue: 0.97, alpha: 1))`
- `HRIS/Common/Components/Alert/BaseAlert.swift` — six separate `Color.white`
  uses across the same file.
- `HRIS/Common/Enum/TeamAttendanceType.swift:47` —
  `Color(red: 0.9, green: 0.71, blue: 0)`
- `HRIS/Feature/Document/View/DocumentDetailView.swift:97,127` —
  `.font(.system(size: 16))`, `.font(.system(size: 16, weight: .semibold))`
  instead of the project's `Font+Base.swift` typography tokens.
