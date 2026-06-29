# Common Unit Testing Rules

## Frameworks

- Use **XCTest** for all test cases.
- Use **Combine** for observing published properties and async state changes.
- Do not introduce any additional testing frameworks.

---

## Test Structure

- Every test class must be `final class DomainTests: XCTestCase`.
- Never override `setUp()` or `tearDown()`. Use `makeSUT()` instead.
- Use `makeSUT()` as a private helper at the bottom of the test class under a `// MARK: - Helpers` section.
- `makeSUT()` must call `trackForMemoryLeak(sut, file: file, line: line)` before returning.

```swift
private func makeSUT(
    /* dependencies */
    file: StaticString = #filePath,
    line: UInt = #line
) -> SomeType {
    let sut = SomeType(/* dependencies */)
    trackForMemoryLeak(sut, file: file, line: line)
    return sut
}
```

---

## Arrange – Act – Assert (AAA) / Given – When – Then (GWT)

Every test function body must follow this three-part structure:

```swift
// Arrange
let remote = SomeMockDataSource(result: .success(anyResponse()))
let sut = makeSUT(remote: remote)

// Act
let result = try await sut.doSomething()

// Assert
XCTAssertEqual(remote.invocations, [.doSomething])
```

Do not merge arrange/assert into the act step.

---

## Triangulation for Arrays

When a method under test returns or operates on a collection, always provide tests for:

1. **Zero items** — empty array / empty response
2. **Single item** — exactly one element
3. **Multiple items** — two or more elements

```swift
func testGetItems_withNoItems_returnsEmptyList() async throws { ... }
func testGetItems_withSingleItem_returnsOneItem() async throws { ... }
func testGetItems_withMultipleItems_returnsAllItems() async throws { ... }
```

---

## Mock Reuse

- Before creating a new mock, search the project for an existing mock of the same type.
- Reuse the existing mock — do not duplicate it.
- If a mock doesn't exist yet, create it once and reuse it across all test files that need it.

---

## Mock Behaviour Injection

- Always pass mock return values through the **initializer**.
- Never set mock results via property assignment after construction.

```swift
// ✅ Correct
let remote = AuthenticationMockRemoteDataSource(
    loginResult: .success(anyLoginSuccessResponse())
)

// ❌ Wrong
let remote = AuthenticationMockRemoteDataSource()
remote.loginResult = .success(anyLoginSuccessResponse())
```

---

## Test Naming Convention

Test names follow the pattern: `test<MethodName>_<condition>_<expectedOutcome>`

Examples:
- `testLogin_callsRemoteWithCorrectRequest()`
- `testLogin_success_returnsLoadedResponse()`
- `testLogin_whenThrowsErrorResponse_returnsErrorState()`
- `testGetItems_withNoItems_returnsEmptyList()`

---

## Helper Factory Methods

Use `any*()` factory helpers for test data to avoid magic values scattered across tests.

```swift
private func anyLoginRequest() -> Authentication.Request.Login {
    Authentication.Request.Login(username: "user@test.com", password: "password")
}

private func anyLoginSuccessResponse() -> GeneralResponse<Authentication.Response.Login> {
    GeneralResponse(success: true, statusCode: 200, message: "Login successfully", data: anyLoginData())
}
```

---

## Test Scope

- Write **unit tests only**.
- Do not write UI Tests, Snapshot Tests, integration tests, or any other test type.
- Tests must be fast, isolated, and deterministic.
