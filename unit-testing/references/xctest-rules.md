# XCTest-Specific Rules

These rules apply when the chosen framework is **XCTest**. Always combine with `common-rules.md`.

---

## Imports

```swift
@testable import HRIS
import XCTest
import Combine
```

---

## Test Class Structure

- Every test class must be `final class DomainTests: XCTestCase`.
- Never override `setUp()` or `tearDown()`. Use `makeSUT()` instead.

```swift
final class AuthenticationDefaultRepositoryTests: XCTestCase {

    // MARK: - Tests

    func testLogin_callsRemoteWithCorrectRequest() async throws { ... }

    // MARK: - Helpers

    private func makeSUT(
        remote: AuthenticationMockRemoteDataSource = AuthenticationMockRemoteDataSource(),
        file: StaticString = #filePath,
        line: UInt = #line
    ) -> AuthenticationDefaultRepository {
        let sut = AuthenticationDefaultRepository(remoteDataSource: remote)
        trackForMemoryLeak(sut, file: file, line: line)
        return sut
    }
}
```

---

## Assertions

| Intent | XCTest |
|---|---|
| Equality | `XCTAssertEqual(a, b)` |
| Truth | `XCTAssertTrue(x)` / `XCTAssertFalse(x)` |
| Nil | `XCTAssertNil(x)` / `XCTAssertNotNil(x)` |
| Force fail | `XCTFail("message")` |
| Throws | `XCTAssertThrowsError(try expr)` |

---

## Async Observation of @Published Properties (Combine + XCTestExpectation)

To assert that a `@Published` property changes, use this exact pattern:

1. Declare `var received* = [Type]()` to collect emitted values.
2. Create `expectation(description:)` and set `assertForOverFulfill = false`.
3. Subscribe with `.sink`, append to the received array, call `exp.fulfill()`.
4. Call the async method under test with `await`.
5. Await with `await fulfillment(of: [exp], timeout: 0.1)`.
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

- `.dropFirst()` — skip 1 initial emission (property starts at a meaningful default).
- `.dropFirst(2)` — skip 2 emissions (ViewModel resets then sets, e.g., `isError = false` then `isError = true`).
- Check the ViewModel implementation to confirm the correct count.

---

## @MainActor for ViewModel Tests

- Every `func test*` in a ViewModel test class must be annotated with `@MainActor`.
- The `makeSUT()` helper must also be annotated with `@MainActor`.

```swift
@MainActor
func testLoadProfile_whenError_showsError() async { ... }

@MainActor
private func makeSUT(...) -> DomainViewModel { ... }
```

---

## Handling switch over RequestState

Use a `switch` + `default: XCTFail(...)` pattern to assert on a specific case:

```swift
switch result {
case .loaded(let response):
    XCTAssertEqual(response.statusCode, 200)
default:
    XCTFail("Expected .loaded but got \(result)")
}
```
