# Data Layer (Repository) Unit Test Patterns

## What to Test

- Test the **Repository** (`DomainDefaultRepository`) — not the RemoteDataSource directly.
- The Repository depends on a `DomainRemoteDataSource`. Inject a `DomainMockRemoteDataSource` via the initializer.
- Do not test networking, URL construction, or Moya internals.

---

## MockRemoteDataSource Structure

Every `DomainMockRemoteDataSource` must:

1. Accept stub results for each method via its initializer.
2. Record each call in an `invocations` array so tests can assert call order and arguments.
3. Use a `MoyaProvider` with `immediatelyStub` when it needs to drive real Moya stubs.

```swift
final class EntityMockRemoteDataSource: EntityRemoteDataSource {

    private let stubProvider: MoyaProvider<EntityTargetType>

    init(
        stubProvider: MoyaProvider<EntityTargetType> = MoyaProvider<EntityTargetType>(stubClosure: MoyaProvider.immediatelyStub)
    ) {
        self.stubProvider = stubProvider
    }

    func getItems() async throws -> GeneralResponse<[Entity.Response.Item]> {
        try await stubProvider.request(.getItems, model: GeneralResponse<[Entity.Response.Item]>.self)
    }
}
```

For mocks that need to simulate success and failure without a real Moya stub:

```swift
final class AuthenticationMockRemoteDataSource: AuthenticationRemoteDataSource {

    enum Invocation: Equatable {
        case login(Authentication.Request.Login)
        case changePassword(Authentication.Request.ChangePassword)
    }

    private(set) var invocations: [Invocation] = []

    private let loginResult: Result<GeneralResponse<Authentication.Response.Login>, Error>
    private let changePasswordResult: Result<GeneralResponse<GeneralVoid>, Error>

    init(
        loginResult: Result<GeneralResponse<Authentication.Response.Login>, Error> = .success(anyLoginSuccessResponse()),
        changePasswordResult: Result<GeneralResponse<GeneralVoid>, Error> = .success(anySuccessResponse())
    ) {
        self.loginResult = loginResult
        self.changePasswordResult = changePasswordResult
    }

    func login(with request: Authentication.Request.Login) async throws -> GeneralResponse<Authentication.Response.Login> {
        invocations.append(.login(request))
        return try loginResult.get()
    }

    func changePassword(body: Authentication.Request.ChangePassword) async throws -> GeneralResponse<GeneralVoid> {
        invocations.append(.changePassword(body))
        return try changePasswordResult.get()
    }
}
```

---

## Required Test Cases per Repository Method

For every repository method, cover:

| Case | Description |
|---|---|
| Correct invocation | Asserts the mock's `invocations` matches the expected call with the correct arguments |
| Success (200) | Asserts the returned `RequestState` is `.loaded(response)` with the expected data |
| ErrorResponse (400/401/404) | Asserts the returned `RequestState` is `.error(ErrorResponse)` with correct statusCode and message |
| Generic error (NSError) | Asserts the returned `RequestState` is `.error(NSError)` with correct domain and code |

---

## Test Examples

### Invocation assertion

```swift
func testLogin_callsRemoteWithCorrectRequest() async throws {
    let remote = AuthenticationMockRemoteDataSource(
        loginResult: .success(anyLoginSuccessResponse())
    )
    let sut = makeSUT(remote: remote)

    _ = try await sut.login(with: anyLoginRequest())

    XCTAssertEqual(
        remote.invocations,
        [.login(anyLoginRequest())]
    )
}
```

### Success case

```swift
func testLogin_success_returnsLoadedResponse() async throws {
    let remote = AuthenticationMockRemoteDataSource(
        loginResult: .success(anyLoginSuccessResponse())
    )
    let sut = makeSUT(remote: remote)

    let result = try await sut.login(with: anyLoginRequest())

    switch result {
    case .loaded(let response):
        XCTAssertEqual(response.success, true)
        XCTAssertEqual(response.statusCode, 200)
        XCTAssertEqual(response.message, "Login successfully")
        XCTAssertEqual(response.data, anyLoginData())
    default:
        XCTFail("Expected login to succeed")
    }
}
```

### ErrorResponse case

```swift
func testLogin_whenThrowsErrorResponse_returnsErrorState() async throws {
    let expectedError = ErrorResponse(
        success: false,
        statusCode: 400,
        message: "Invalid password",
        errors: nil
    )
    let remote = AuthenticationMockRemoteDataSource(
        loginResult: .failure(expectedError)
    )
    let sut = makeSUT(remote: remote)

    let result = try await sut.login(with: anyLoginRequest())

    switch result {
    case .error(let error as ErrorResponse):
        XCTAssertEqual(error.statusCode, 400)
        XCTAssertEqual(error.message, "Invalid password")
    default:
        XCTFail("Expected login to return ErrorResponse")
    }
}
```

### Generic error case

```swift
func testLogin_whenThrowsGenericError_returnsErrorState() async throws {
    let dummyError = NSError(domain: "TestError", code: 999)
    let remote = AuthenticationMockRemoteDataSource(
        loginResult: .failure(dummyError)
    )
    let sut = makeSUT(remote: remote)

    let result = try await sut.login(with: anyLoginRequest())

    switch result {
    case .error(let error as NSError):
        XCTAssertEqual(error.domain, "TestError")
        XCTAssertEqual(error.code, 999)
    default:
        XCTFail("Expected login to return generic error")
    }
}
```

---

## makeSUT for Repository Tests

```swift
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
```

---

## HTTP Status Coverage

At a minimum, cover these response codes per method:

- `200` — success
- `401` — unauthorized
- `404` — not found
- `500` — server error

Use `ErrorResponse` with the appropriate `statusCode` to simulate each case.

---

## Reference Existing Tests

If patterns are unclear, reference any existing `DefaultRepositoryTests` file in the project for the canonical implementation.
