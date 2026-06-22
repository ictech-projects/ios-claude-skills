# Build & Release Security

## Build Configuration

- Debug builds must never ship to production. Flag release schemes that include `DEBUG` preprocessor macros.
- Strip debug symbols from release builds (`STRIP_INSTALLED_PRODUCT = YES`, `COPY_PHASE_STRIP = YES`).
- Enable bitcode where supported to allow App Store re-optimization without re-submission.
- Dead code stripping must be enabled (`DEAD_CODE_STRIPPING = YES`) to reduce the attack surface in the binary.

## Compiler Flags

- Enable stack protector: `OTHER_CFLAGS = -fstack-protector-all`.
- Enable full ASLR: `OTHER_CFLAGS += -fPIE`. Verify the binary has PIE enabled using `otool -hv`.
- Disable Objective-C associated objects in release if not used: reduces runtime attack surface.
- Enable hardened runtime settings where available.

## Entitlements

- Review the `.entitlements` file for every target. Flag entitlements that are not required by any feature.
- `com.apple.security.get-task-allow` must be `false` in release builds — it allows debugger attachment.
- Flag `keychain-access-groups` that include groups not owned by the app.
- Flag `com.apple.developer.associated-domains` entries for domains the app does not own.

## App Store & Distribution

- All provisioning profiles used for distribution must use App Store or Enterprise distribution — never Development profiles for production releases.
- Ensure `NSAppTransportSecurity` exceptions are removed or minimized before App Store submission.
- Review `.xcconfig` files used in release schemes — ensure no debug endpoints, staging keys, or verbose logging flags are active.

## Reverse Engineering Protection

- Flag debug detection logic that is the sole protection against tampering — it can be bypassed by patching a single branch.
- Jailbreak/root detection is a compensating control, not a security boundary. Document it as such.
- Do not use string literals for sensitive logic checks (e.g., `if environment == "production"`) — these are trivially patchable.
- Class and method names in the binary are readable. Flag sensitive logic placed in obviously named methods (e.g., `isPremiumUser`, `isAdminAccount`).
