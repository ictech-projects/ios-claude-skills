# Secure Swift Coding Practices

## Memory & Type Safety

- Avoid `UnsafePointer`, `UnsafeMutablePointer`, and `UnsafeRawBufferPointer` unless interfacing with C APIs. Flag any security-critical code path using unsafe memory access.
- Never use force unwrap (`!`) in security-critical code paths (auth checks, permission validation, crypto). A crash can itself be a denial-of-service.
- Use typed errors (`enum MyError: Error`) rather than stringly typed error handling — untyped errors can mask the true failure mode.

## Deep Links & URL Handling

- All deep link handlers must validate the URL scheme, host, and path before acting on parameters.
- Never execute actions from deep link parameters without authentication checks — assume deep links are attacker-controlled.
- Flag `UIApplicationDelegate.application(_:open:options:)` or `onOpenURL` handlers that act on query parameters without sanitization.

```swift
// Wrong: acting on raw deep link value
func handleDeepLink(_ url: URL) {
    let action = url.queryParameters["action"]
    performAction(action) // attacker-controlled
}

// Correct: validate against an allowlist
func handleDeepLink(_ url: URL) {
    guard let action = url.queryParameters["action"],
          allowedActions.contains(action) else { return }
    performAction(action)
}
```

## WebView Security

- `WKWebView` must not have `allowsArbitraryLoads` set via ATS exceptions.
- Disable `allowsBackForwardNavigationGestures` in WebViews that load authenticated content.
- Use `WKContentRuleList` to block mixed content and restrict navigation to known domains.
- Never inject user-supplied content into `evaluateJavaScript()` — this enables JavaScript injection attacks.
- Disable `WKPreferences.javaScriptEnabled` if the WebView is only displaying local HTML content.

## Input Validation

- Sanitize all user input before use in file paths, URLs, SQL queries, or shell commands.
- Flag `String` interpolation used directly to construct file paths — use `URL(fileURLWithPath:)` with validated components instead.
- Numeric inputs used in security decisions (e.g., retry counts, amounts) must be validated against minimum/maximum bounds.

## Clipboard Security

- Never place sensitive data (TOTP seeds, passwords, private keys, card numbers) on the system clipboard without a short expiry.
- Clear sensitive clipboard content after a maximum of 60 seconds using a `Task.sleep` or a `DispatchWorkItem` scheduled on the main queue.
- Flag any `UIPasteboard.general.string = sensitiveValue` that is not accompanied by a corresponding clear after a timeout.
- Disable the system copy menu on `TextField` and `TextEditor` views that display sensitive values (e.g., OTP seeds, private keys) using `.textSelection(.disabled)`.
- Do not use `UIPasteboard.general` for inter-process data transfer of sensitive values — use Keychain-backed shared containers within the same app group instead.

```swift
// Correct: copy TOTP seed with automatic expiry
func copyWithExpiry(_ value: String, after seconds: Double = 60) {
    UIPasteboard.general.string = value
    Task {
        try? await Task.sleep(for: .seconds(seconds))
        if UIPasteboard.general.string == value {
            UIPasteboard.general.string = ""
        }
    }
}
```

## Concurrency & Race Conditions

- Authentication state changes must be actor-isolated or protected by a serial queue — flag any unsynchronized read/write of auth state across concurrent tasks.
- Token refresh logic must use a lock or actor to prevent multiple simultaneous refresh requests from issuing duplicate tokens.
- Flag `@MainActor`-isolated security checks that could be bypassed via a detached `Task`.
