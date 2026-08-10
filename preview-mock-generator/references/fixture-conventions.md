# Fixture Conventions

## The pattern

A preview mock is a plain `struct`/`class` conforming to a Repository protocol,
compiled only for use inside `#Preview` blocks. Every method returns a
hand-built, realistic instance of the protocol's real return types — never an
empty struct, never `nil` where the type allows it, never placeholder strings
like `"test"` or `"foo"`.

Nested Model types get filled in the same way, recursively: if a `Profile`
Model has a nested `Contract`, `EmergencyContact`, `FinancialInformation`, the
fixture builds all of them with plausible values, not just the top-level
fields.

## Building one

1. Read the protocol's method signatures and their real return types — do not
   guess a shape, resolve the actual Model definitions.
2. For each Model field, pick a plausible value in the same style as
   surrounding fixtures already in the codebase (names, dates, IDs) — reuse
   existing fixture values across files where it makes sense (e.g. the same
   sample employee name) rather than inventing a new one every time.
3. For async/throwing methods, return the success case by default. Only add a
   failure path if the View being previewed specifically needs to preview an
   error state — don't build both paths speculatively.
4. Keep the fixture data static (a `let` constant or literal returned
   directly) — no randomization, so the preview renders identically every time.

## Worked example (hris-ios)

`HRIS/Feature/Profile/PreviewMocks/AuthenticationPreviewRepository.swift` (~170
lines) stubs every `AuthenticationRepository` method (`login`, `logout`,
`changePassword`, `getProfile`, `biometricLogin`, `checkInduction`, …) backed
by one realistic `Employee` fixture with nested `Profile`, `Contract`,
`EmergencyContact`, `FinancialInformation`, `DrivingLicense`, and
`EmployeeDocument` values all filled in — not just the fields the currently
previewed View happens to read.

Other examples following the same shape: `Home/PreviewMocks/AttendancePreviewRepository.swift`,
`Document/Previews/{EmployeePreviewRepository,DocumentPreviewRepository}.swift`,
`Document/ViewModel/DownloadPreviewRepository.swift`,
`Profile/PreviewMocks/EntityPreviewRepository.swift`.
