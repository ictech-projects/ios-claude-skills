# Privacy & Data Protection

## Logging

- Never log PII, tokens, passwords, or sensitive identifiers — not even in debug builds, since logs can be captured by MDM, Xcode Organizer, or crash reporters.
- Flag any `print()`, `NSLog()`, or `os_log()` call that outputs: email, phone number, name, address, device ID, auth token, or financial data.
- Use `OSLogPrivacy.auto` or `.private` for any log message that may contain user data.
- Crash reporting SDKs (e.g., Firebase Crashlytics, Sentry) must be configured to scrub PII before transmission.

```swift
// Wrong: PII in plain log
print("User logged in: \(user.email)")

// Correct: redacted with os_log privacy
let logger = Logger(subsystem: "com.ict.app", category: "auth")
logger.info("User logged in: \(user.email, privacy: .private)")
```

## Data Minimization

- Collect only the minimum data required for the feature. Flag any field collected but not used within the app.
- PII must not be included in analytics events as raw values — use hashed or anonymized identifiers.
- Device identifiers: never use `UDID` (unavailable) or MAC address. Prefer `identifierForVendor` scoped to your app group, or a server-assigned UUID stored in the Keychain.

## Privacy Manifests

- iOS 17+ requires a `PrivacyInfo.xcprivacy` manifest. Flag apps targeting iOS 17+ without one.
- All required reason APIs (file timestamps, user defaults, disk space, etc.) must be declared in the manifest.
- Third-party SDKs that access required reason APIs must also provide their own privacy manifests.

## App Tracking Transparency

- Any use of data for cross-app tracking requires an ATT prompt (`AppTrackingTransparency`). Flag tracking without the prompt.
- Do not access `ASIdentifierManager.advertisingIdentifier` without checking `ATTrackingManager.trackingAuthorizationStatus == .authorized` first.

## Background UI Obscuring

- When the app enters the background, any screen displaying sensitive data (account details, TOTP codes, payment info, PII) must be obscured to prevent capture in the iOS app switcher.
- Apply a blur overlay or replace the window content with a branded placeholder in `scenePhase == .inactive` or via `UIApplicationDelegate.applicationWillResignActive`.
- Flag any view displaying sensitive content that does not respond to the `scenePhase` environment value or the `UIApplication.willResignActiveNotification` notification.
- The obscuring view must be removed only after the app returns to `scenePhase == .active` — do not remove it on `.background` entry, as the switcher snapshot is taken during `.inactive`.

```swift
// Correct: blur sensitive content when app is inactive
struct SensitiveView: View {
    @Environment(\.scenePhase) private var scenePhase

    var body: some View {
        ZStack {
            SensitiveContentView()
            if scenePhase != .active {
                Rectangle()
                    .fill(.regularMaterial)
                    .ignoresSafeArea()
            }
        }
    }
}
```

## Permissions

- Request permissions only when needed, at the moment of first use — not at app launch.
- Flag apps that request microphone, camera, contacts, or location without a clear in-app justification shown immediately before the system prompt.
- Usage description strings in `Info.plist` must be specific and honest — flag generic strings like "Required for app functionality".
