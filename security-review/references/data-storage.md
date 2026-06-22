# Data Storage Security

## Keychain

- Use the Keychain for all sensitive data: tokens, credentials, cryptographic keys, PII.
- Always set `kSecAttrAccessible` to `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` unless there is a specific, documented reason for a weaker policy.
- Never use `kSecAttrAccessibleAlways` or `kSecAttrAccessibleAlwaysThisDeviceOnly` — these allow access when the device is locked.
- Flag Keychain items missing a `kSecAttrAccessGroup` when the app is part of an app group that shares credentials.
- Keychain items must be deleted on logout — do not rely on uninstall to clear them, as Keychain data can persist across reinstalls.

## UserDefaults

- `UserDefaults` is stored as a plain plist — never use it for tokens, passwords, API keys, or any PII.
- Acceptable uses: non-sensitive preferences, UI state, feature flags.
- Flag any key in `UserDefaults` that contains: token, password, secret, key, auth, user_id, email, or similar.

## File System

- Sensitive files must be excluded from iCloud backup using `.isExcludedFromBackupKey = true`.
- Use `.completeFileProtection` (`NSFileProtectionComplete`) for files containing sensitive data.
- Never write sensitive data to the `tmp/` or `Caches/` directory — these are not protected and can be read without authentication.
- SQLite databases storing PII must be encrypted (e.g., SQLCipher). Flag unencrypted databases with sensitive columns.

```swift
// Correct: exclude sensitive file from backup
var url = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
    .appendingPathComponent("sensitive.db")
try (url as NSURL).setResourceValue(true, forKey: .isExcludedFromBackupKey)

// Correct: apply full file protection
try data.write(to: url, options: .completeFileProtection)
```

## Core Data

- Sensitive Core Data stores must use a SQLite file with `.completeFileProtection` and be excluded from backup.
- Flag `NSPersistentStoreDescription` configurations that do not set file protection explicitly when storing PII.
