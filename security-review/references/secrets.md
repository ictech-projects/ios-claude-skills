# Secrets Management

- Never hardcode API keys, tokens, passwords, or private keys as string literals in Swift source files.
- Never commit `.xcconfig` files containing secrets to version control. Use environment variables or a secrets manager at build time.
- Flag any string that matches common secret patterns: `sk-`, `pk_`, `Bearer `, `Basic `, hex strings >32 chars, or strings labeled `key`, `secret`, `password`, `token` assigned to a literal.
- Secrets must not appear in `Info.plist` values, even under obfuscated key names — `Info.plist` is trivially readable from the app bundle.
- Do not store secrets in asset catalogs, JSON config files, or any resource bundled with the app binary.
- Use `.gitignore` and `.xcodeignore` to prevent secrets files from being tracked. Flag repos missing these for secrets-bearing files.

## Acceptable Patterns

```swift
// Wrong: hardcoded literal
let apiKey = "sk-abc123hardcodedkey"

// Wrong: plist-backed key (readable from bundle)
let apiKey = Bundle.main.infoDictionary?["API_KEY"] as? String

// Correct: injected at build time via .xcconfig (file is gitignored)
// In .xcconfig: API_KEY = $(API_KEY_ENV_VAR)
// In Info.plist: <key>API_KEY</key><string>$(API_KEY)</string>
// At runtime, the value is baked in at build, never committed
```

## Runtime Obfuscation

- Obfuscation (e.g., splitting strings, XOR encoding) is not a substitute for proper secrets management — flag it as insufficient.
- If secrets must be embedded, document the threat model and compensating controls (e.g., server-side key rotation, short-lived tokens).
- Prefer secrets that are short-lived and scoped (e.g., ephemeral OAuth tokens) over long-lived static API keys.

## Binary Analysis Risk

- Embedded string literals are trivially extracted with `strings` or a disassembler. Treat the app binary as public.
- Flag any key with admin or write-level permissions that is embedded in the binary — read-only, rate-limited keys are lower risk but still not ideal.
