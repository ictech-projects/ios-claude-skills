# Authentication & Authorization

- Never store raw passwords or session tokens in `UserDefaults`. Use the Keychain with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` or stricter.
- Always validate JWTs on the server — do not trust client-side JWT decoding for authorization decisions.
- Tokens must have a defined expiry (`exp` claim). Flag tokens with no expiry or excessively long lifetimes (>24h for access tokens).
- Use `ASWebAuthenticationSession` for OAuth flows, never a custom `WKWebView` which exposes tokens to JavaScript.
- Biometric authentication must use `LAContext` with `localizedReason`. Never fall back to device passcode silently — make the fallback explicit and intentional.
- Do not cache authentication state in memory beyond the session. Re-validate on app foreground where appropriate.
- Flag any auth check that can be bypassed by a return value alone (e.g., `if isAuthenticated { ... }` where `isAuthenticated` is a stored Bool that can be flipped at runtime).
- Multi-factor authentication state must not be stored in `UserDefaults` or any plist — only in memory or the Keychain.
- Deep links must never trigger authenticated actions without re-validating the session state first.
- Implement token refresh logic with a proper lock to prevent race conditions issuing multiple refresh requests concurrently.

## Keychain Usage

```swift
// Correct: store token securely with device-bound protection
let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrAccount as String: "authToken",
    kSecValueData as String: tokenData,
    kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
]
SecItemAdd(query as CFDictionary, nil)

// Wrong: storing tokens in UserDefaults
UserDefaults.standard.set(token, forKey: "authToken")
```
