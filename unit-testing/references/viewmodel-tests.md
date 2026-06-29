# ViewModel Unit Test Patterns

## What to Test

- Test the **ViewModel** (`DomainViewModel`) in isolation.
- Inject mock repositories via `makeSUT()` — never use real repositories.
- Assert ViewModel `@Published` property changes using Combine `.sink` + `XCTestExpectation`.

---

## Required Setup

Every ViewModel test file must import:

```swift
@testable import HRIS
import XCTest
import Combine
```

---

## @MainActor

- Every `func test*` in a ViewModel test class must be annotated with `@MainActor`.
- The `makeSUT()` helper must also be annotated with `@MainActor`.

```swift
@MainActor
func testOnLoadSubjects_whenError_showsError() async { ... }

@MainActor
private func makeSUT(...) -> DomainViewModel { ... }
```

---

## Observing @Published Properties with Combine

To assert that a `@Published` property changes, use this exact pattern:

1. Declare a `var received* = [Type]()` array to collect emitted values.
2. Create an `expectation(description:)` and set `assertForOverFulfill = false`.
3. Subscribe with `.sink`, appending to the received array and calling `exp.fulfill()`.
4. Call the async method under test with `await`.
5. Await the expectation with `await fulfillment(of: [exp], timeout: 0.1)`.
6. Assert with `XCTAssertEqual`.
7. Call `cancellable.cancel()` at the end.

```swift
@MainActor
func testOnLoadSubjects_whenError_showsError() async {
    let anyError = NSError(domain: "", code: -1)
    let subjectRepository = SubjectMockRepository(result: .error(anyError))
    let sut = makeSUT(subjectRepo: subjectRepository)
    var receivedErrors = [Bool]()
    let exp = expectation(description: "wait for subscription")
    exp.assertForOverFulfill = false
    let cancellable = sut.$isError
        .dropFirst(2)
        .sink { isError in
            receivedErrors.append(isError)
            exp.fulfill()
        }

    await sut.loadSubjects()
    await fulfillment(of: [exp], timeout: 0.1)

    XCTAssertEqual(subjectRepository.invocations, [.getSubjects])
    XCTAssertEqual(receivedErrors, [true])
    cancellable.cancel()
}
```

### dropFirst rules

- Use `.dropFirst()` (skip 1) when the property starts at a meaningful default and you only care about changes after the action.
- Use `.dropFirst(2)` when the ViewModel resets then sets the property (e.g., sets `isError = false` before `isError = true`).
- Check the ViewModel's implementation to determine the correct `dropFirst` count.

---

## Mock Repository Structure

Mock repositories track invocations and return configurable results:

```swift
final class SubjectMockRepository: SubjectRepository {

    enum Invocation: Equatable {
        case getSubjects
    }

    private(set) var invocations: [Invocation] = []
    private let result: RequestState<GeneralResponse<[Subject.Response.Subject]>>

    init(result: RequestState<GeneralResponse<[Subject.Response.Subject]>> = .idle) {
        self.result = result
    }

    func getSubjects() async throws -> RequestState<GeneralResponse<[Subject.Response.Subject]>> {
        invocations.append(.getSubjects)
        return result
    }
}
```

---

## Test Examples

### Error state

```swift
@MainActor
func testLoadProfile_whenError_showsError() async {
    let anyError = NSError(domain: "", code: -1)
    let authRepository = AuthenticationMockRepository(getProfileResult: .error(anyError))
    let sut = makeSUT(authRepo: authRepository)
    var receivedErrors = [Bool]()
    let exp = expectation(description: "wait for subscription")
    exp.assertForOverFulfill = false
    let cancellable = sut.$isError
        .dropFirst(2)
        .sink { isError in
            receivedErrors.append(isError)
            exp.fulfill()
        }

    await sut.loadProfile()
    await fulfillment(of: [exp], timeout: 0.1)

    XCTAssertEqual(authRepository.invocations, [.getProfile])
    XCTAssertEqual(receivedErrors, [true])
    cancellable.cancel()
}
```

### Success state

```swift
@MainActor
func testLoadProfile_whenSuccess_setsEmployeeProfile() async {
    let expectedEmployee = Authentication.Response.Employee(
        id: 1,
        name: "John Doe",
        email: "john.doe@example.com",
        employeeId: "EMP001",
        status: "active",
        departmentId: 5,
        headedDepartments: nil,
        managerId: nil,
        roleId: 2,
        profile: nil,
        contract: nil,
        documents: nil
    )
    let response = GeneralResponse(
        success: true,
        statusCode: 200,
        message: "Success",
        data: expectedEmployee
    )
    let authRepository = AuthenticationMockRepository(getProfileResult: .loaded(response))
    let sut = makeSUT(authRepo: authRepository)
    var receivedProfile: Authentication.Response.Employee?
    let exp = expectation(description: "wait for subscription")
    exp.assertForOverFulfill = false
    let cancellable = sut.$employeeProfile
        .dropFirst()
        .sink { profile in
            receivedProfile = profile
            exp.fulfill()
        }

    await sut.loadProfile()
    await fulfillment(of: [exp], timeout: 0.1)

    XCTAssertEqual(authRepository.invocations, [.getProfile])
    XCTAssertEqual(receivedProfile, expectedEmployee)
    cancellable.cancel()
}
```

### Synchronous property tests (no Combine needed)

For pure computed properties or synchronous state changes, skip the Combine pattern:

```swift
@MainActor
func test_isInputDataValid_emptySubjectAndMessage_returnsFalse() {
    let sut = makeSUT()

    sut.subject = ""
    sut.message = ""

    XCTAssertFalse(sut.isInputDataValid())
}
```

---

## makeSUT for ViewModel Tests

```swift
// MARK: - Helpers

@MainActor
private func makeSUT(
    subjectRepo: SubjectMockRepository = SubjectMockRepository(),
    authRepo: AuthenticationMockRepository = AuthenticationMockRepository(),
    file: StaticString = #filePath,
    line: UInt = #line
) -> ContactUsViewModel {
    let sut = ContactUsViewModel(
        subjectRepository: subjectRepo,
        authenticationRepository: authRepo
    )
    trackForMemoryLeak(sut, file: file, line: line)
    return sut
}
```

---

## Coverage Checklist

For each ViewModel method under test, cover:

- [ ] Error state — repository returns `.error`, ViewModel sets `isError = true`
- [ ] Success state — repository returns `.loaded`, ViewModel sets the expected property
- [ ] Edge cases — missing required data, invalid input, guard clause failures
- [ ] Synchronous helpers — `isInputDataValid()`, `resetErrorState()`, etc. (no Combine needed)

---

## Reference Existing Tests

If patterns are unclear, reference any existing `ViewModelTests` file in the project for the canonical implementation.
