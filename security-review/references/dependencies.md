# Dependency & Third-Party SDK Security

## Dependency Inventory

- Every third-party dependency must have a documented justification. Flag any package without a clear use case.
- Check all dependencies against known vulnerability databases. Flag packages with unpatched CVEs.
- Prefer Apple-native frameworks over third-party equivalents where feature-equivalent (e.g., `CryptoKit` over a custom crypto library).

## Package Integrity

- Use Swift Package Manager with exact version pins (`from: "x.y.z"` or `.exact("x.y.z")`). Flag `.upToNextMajor` or unversioned dependencies in security-sensitive apps.
- Verify that `Package.resolved` is committed and reviewed in PRs — it is the lock file and must not be ignored.
- For CocoaPods, `Podfile.lock` must be committed. Flag repos where it is gitignored.
- Carthage `Cartfile.resolved` must also be committed.

## SDK Permissions & Data Access

- Audit each third-party SDK for the permissions it requires and data it transmits.
- Flag analytics, crash reporting, and ad SDKs that access the address book, location, or device identifiers beyond what the app itself uses.
- SDKs with network access must be evaluated for their data-sharing practices — flag SDKs that send data to unknown third-party endpoints.

## Supply Chain Risk

- Flag dependencies with very few maintainers, abandoned repos (no commits in >12 months), or unusual ownership changes.
- Do not use forks of popular libraries unless the fork is actively maintained and the reason for forking is documented.
- Verify the package URL matches the expected canonical repository (e.g., `github.com/apple/...` not a lookalike).

## Cryptography

- Never use third-party cryptography libraries for new code — use `CryptoKit` (Swift) or `CommonCrypto` (C-level).
- Flag MD5 or SHA-1 usage for security-sensitive hashing (e.g., password storage, integrity verification). Use SHA-256 or better.
- Flag `SecRandomCopyBytes` alternatives that use `arc4random` or `rand()` for security-sensitive randomness.
