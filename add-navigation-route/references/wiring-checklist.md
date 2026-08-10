# Wiring Checklist

Adding one screen touches every item below. Skipping one either crashes the app
(missing render-switch arm — the enum case has no matching View) or silently
does nothing (missing entry-point wiring — a tap goes nowhere). Neither failure
is caught by the compiler, since the check happens at runtime against a
`NavigationPath` push.

1. **Route enum** — add the case, with associated values for any data the
   destination view needs (an ID, a request/response payload, a flag).
2. **Render switch** — add the matching `case` arm that returns the
   destination View. Nothing else in the switch should change.
3. **Entry point** — wire the button/menu-item/action that calls
   `navigation.navigate(to: .newCase)` (or this project's equivalent). If the
   trigger site is ambiguous, report the exact file/line to add it manually
   instead of guessing which control it should be.
4. **Convenience jumper (optional)** — only if an equivalent screen already has
   a `toX()`-style jumper for a comparable flow; don't add one speculatively.

## Worked example (hris-ios: `ContactUs`)

Adding the `ContactUs` screen touched:

- `HRIS/Common/Enum/NavigationPage.swift:25` — `case contactUs`
- `HRIS/Common/Base/HRISApp.swift:98-99` — `case .contactUs: ContactUsView()`
- `HRIS/Common/Enum/SettingsMenu.swift:41,68` — added a `.contactUs` menu entry
- `HRIS/Feature/Settings/View/SettingsView.swift:176-177` —
  `navigation.navigate(to: .contactUs)` on tap

No `toX()` jumper was added — `ContactUs` is reached by a plain push from
Settings, not a stack reset, so none of the existing `NavigationManager`
jumpers applied.

## What to report back

List all four touchpoints (or this project's equivalents) as a checklist, each
marked either "wired" (with file/line) or "needs manual wiring" (with the exact
file/line where it belongs) — so the developer can verify nothing was missed
before committing.
