# Navigation Pattern

## The pattern

ICT iOS apps route through a single flat, `Hashable` enum of destinations pushed
onto a `NavigationPath`/rendered by a `NavigationStack`, turned into Views by one
central `.navigationDestination(for:)` switch, with a `NavigationManager`-style
`ObservableObject` exposing convenience jumpers for common multi-screen resets
(e.g. `toHome()`, `toLogin()` — each rebuilds the path from scratch rather than
pushing a single step).

There is no Coordinator protocol layer and no DI container involved in routing —
navigation is driven entirely by mutating the shared `NavigationPath`(s) held by
the navigation manager.

## How to locate it in a given project

1. Search for a `Hashable` enum with many cases named like a screen list
   (`NavigationPage`, `Route`, `Screen`, `Destination`) — this is the route enum.
   Some cases carry associated values (request/response payloads, IDs); most don't.
2. Search for `.navigationDestination(for: <RouteEnumName>.self)`. The `switch`
   immediately inside it is the central render switch — in practice this often
   lives in the app's root `App` file, not next to the enum itself.
3. Search for an `ObservableObject` holding one or more `NavigationPath`
   properties (e.g. `path`, `innerPath`) — this is the navigation manager. Its
   methods are the convenience jumpers; note their naming style before adding
   a new one.
4. **If none of the above are found**, stop. The project likely uses a different
   architecture (a Coordinator protocol, or per-view `.navigationDestination`
   calls with no shared enum). Report what you found instead and ask how
   routing works here — don't retrofit the enum pattern onto a project that
   doesn't use it.

## Worked example (hris-ios)

- Route enum: `HRIS/Common/Enum/NavigationPage.swift` — e.g. `case contactUs`,
  `case resetPassword(showHeaderImage: Bool)`, `case verifyResetCode(email: String, expires: Int)`.
- Central render switch: `HRIS/Common/Base/HRISApp.swift` —
  `case .contactUs: ContactUsView()`.
- Navigation manager: `HRIS/Common/Helper/NavigationHelper/NavigationManager.swift`
  (class `NavigationHelper`) — `toHome()`, `toNotification()`, each doing
  `path = NavigationPath(); path.append(.login); path.append(.home)`-style resets.
- Entry point for a menu-launched screen: a menu enum/view (e.g. `SettingsMenu`)
  calling `navigation.navigate(to: .contactUs)` from its tap handler.
