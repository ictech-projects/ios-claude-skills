# Error Mapping Pattern

`handleErrorResponseUsingDefaultFlow` is not a shared base-class method or
protocol extension — it's a `private func` hand-copied into each ViewModel
(12+ occurrences in `hris-ios`: `HomeViewModel`, `ProfileViewModel`,
`ContactUsViewModel`, `LeaveRequestViewModel`, `EditLeaveRequestViewModel`,
`UpdatePasswordViewModel`, `TeamAttendancesViewModel`,
`NotificationSettingViewModel`, `AttendanceHistoryViewModel`,
`MainLeaveViewModel`, `NotificationsViewModel`,
`TeamAttendanceDetailsViewModel`, …). Two distinct shapes coexist — check
which one the target's Repository already follows before generating either.

## Shape 1 — Repository already isolates expiry

The Repository's response type is a state enum with a dedicated `.expired`
case, so the ViewModel's state switch catches 401 *before* the generic error
handler ever runs. `handleErrorResponseUsingDefaultFlow` here only needs to
set the plain flag/message — no dedicated mapper.

```swift
// HomeViewModel.swift
private func handleErrorResponseUsingDefaultFlow(_ response: Error) {
    if let errorResponse = response as? ErrorResponse {
        isError = true
        errorMessage = errorResponse.message
    } else {
        isError = true
        errorMessage = response.localizedDescription
    }
}

func loadNotifications() async {
    // ...
    switch state {
    case .expired:
        isExpired = true
        authenticationRepository.eraseToken(with: KeychainKey.accessToken.key)
        await fcmDeletionIgnoringResult()
    case .error(let response):
        handleErrorResponseUsingDefaultFlow(response)
    }
}
```

## Shape 2 — Repository throws a raw `Error`

The ViewModel needs a dedicated `<Feature>ViewModelErrorMapper` that inspects
the error's status code itself, switching 401 into expiry.

```swift
// ProfileViewModel.swift
private func handleErrorResponseUsingDefaultFlow(_ error: Error) {
    let result = ProfileViewModelErrorMapper.map(error)

    if result.isExpired {
        isExpired = true
        authenticationRepository.eraseToken(with: KeychainKey.accessToken.key)
        return
    }

    if let message = result.message {
        errorMessage = message
        withAnimation { isError = true }
    }
}
```

```swift
// ProfileViewModelErrorMapper.swift
struct ViewModelErrorResult {
    let message: String?
    let isExpired: Bool
    let isValidationError: Bool

    init(message: String?, isExpired: Bool, isValidationError: Bool = false) {
        self.message = message
        self.isExpired = isExpired
        self.isValidationError = isValidationError
    }
}

final class ProfileViewModelErrorMapper {
    static func map(_ error: Error) -> ViewModelErrorResult {
        guard let errorResponse = error as? ErrorResponse else {
            return ViewModelErrorResult(message: "Something went wrong. Please try again.", isExpired: false)
        }

        switch errorResponse.statusCode {
        case 401:
            return ViewModelErrorResult(message: nil, isExpired: true)
        case 422:
            let message = mapValidationErrors(errorResponse)
            return ViewModelErrorResult(message: message ?? errorResponse.message, isExpired: false, isValidationError: true)
        default:
            return ViewModelErrorResult(message: errorResponse.message, isExpired: false)
        }
    }

    private static func mapValidationErrors(_ errorResponse: ErrorResponse) -> String? {
        guard let errors = errorResponse.errors else { return nil }
        let messages = Mirror(reflecting: errors).children
            .compactMap { $0.value as? [String] }
            .flatMap { $0 }
        guard !messages.isEmpty else { return nil }
        return messages.map { "- \($0)" }.joined(separator: "\n")
    }
}
```

`MultiFactorAuthenticationErrorMapper` follows the identical shape, adding a
`404` case alongside `401`/`422` — extend the `switch` with whatever status
codes the target's actual backend endpoint can return, don't copy 401/422/404
speculatively if the endpoint doesn't produce them.

## Which shape to generate

- If the target Repository's method already returns a state enum with an
  `.expired` case → generate shape 1 (no mapper file).
- If the target Repository's method throws a raw `Error` → generate shape 2
  (dedicated `<Feature>ViewModelErrorMapper` + `ViewModelErrorResult`).
- **Never** add a dedicated mapper on top of a Repository that already isolates
  expiry via shape 1 — that would duplicate the 401 handling in two places.

## The one rule that never changes

However 401 is detected, it must always resolve to the exact same sequence:
`isExpired = true` → `authenticationRepository.eraseToken(with: KeychainKey.accessToken.key)`
→ (implicitly) logout. Never write a second implementation of this — it's a
security-relevant path, not a place for per-feature creativity.
