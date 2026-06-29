# Common Unit Testing Rules (Framework-Agnostic)

These rules apply regardless of whether you use XCTest or Swift Testing. For framework-specific syntax, see `xctest-rules.md` or `swift-testing-rules.md`.

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
// (assert invocations, result values, state changes)
```

Do not merge arrange/assert into the act step.

---

## makeSUT()

- Always create the system under test via a `makeSUT()` private helper — never inline construction in each test.
- Place `makeSUT()` at the bottom of the test type under a `// MARK: - Helpers` section.
- `makeSUT()` must call `trackForMemoryLeak(sut, file: file, line: line)` before returning.
- Accept all dependencies as parameters with sensible defaults so each test only provides what it needs.

```swift
private func makeSUT(
    /* dependencies with defaults */
    file: StaticString = #filePath,
    line: UInt = #line
) -> SomeType {
    let sut = SomeType(/* dependencies */)
    trackForMemoryLeak(sut, file: file, line: line)
    return sut
}
```

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
- `testLogin_callsRemoteWithCorrectRequest`
- `testLogin_success_returnsLoadedResponse`
- `testLogin_whenThrowsErrorResponse_returnsErrorState`
- `testGetItems_withNoItems_returnsEmptyList`

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
