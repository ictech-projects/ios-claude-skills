# Swift Testing-Specific Rules

These rules apply when the chosen framework is **Swift Testing** (Xcode 16+). Always combine with `common-rules.md`.

---

## Imports

```swift
@testable import HRIS
import Testing
import Combine
```

---

## Test Type Structure

- Use a `struct` (preferred) or `final class` annotated with `@Suite` instead of subclassing `XCTestCase`.
- No `setUp`/`tearDown`. Use `makeSUT()` as defined in `common-rules.md`.
- Each test function is annotated with `@Test` — no `test` prefix required, but follow the naming convention from `common-rules.md` for consistency.

```swift
@Suite("AuthenticationDefaultRepository")
struct AuthenticationDefaultRepositoryTests {

    // MARK: - Tests

    @Test func login_callsRemoteWithCorrectRequest() async throws { ... }

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

| Intent | Swift Testing |
|---|---|
| Equality | `#expect(a == b)` |
| Truth | `#expect(x)` / `#expect(!x)` |
| Nil | `#expect(x == nil)` / `#expect(x != nil)` |
| Unwrap (throws if nil) | `let value = try #require(optionalValue)` |
| Force fail | `Issue.record("message")` |
| Throws | `#expect(throws: SomeError.self) { try expr }` |

---

## Async Observation of @Published Properties (Combine + confirmation)

Swift Testing has no `XCTestExpectation`. Use `withCheckedContinuation` combined with Swift Testing's `confirmation` API to observe Combine publishers.

**Pattern:**

1. Create a `confirmation("description", expectedCount: N)` inside a `withCheckedContinuation` or directly in an `await confirmation { confirm in ... }` block.
2. Subscribe with `.sink`, call `confirm()` when the expected value arrives.
3. Call the async method under test with `await` inside the confirmation block.
4. Assert collected values after the block.

```swift
@Test
@MainActor
func loadSubjects_whenError_showsError() async {
    let anyError = NSError(domain: "", code: -1)
    let subjectRepository = SubjectMockRepository(result: .error(anyError))
    let sut = makeSUT(subjectRepo: subjectRepository)
    var receivedErrors = [Bool]()

    await confirmation("isError emits true") { confirm in
        let cancellable = sut.$isError
            .dropFirst(2)
            .sink { isError in
                receivedErrors.append(isError)
                confirm()
            }

        await sut.loadSubjects()
        cancellable.cancel()
    }

    #expect(subjectRepository.invocations == [.getSubjects])
    #expect(receivedErrors == [true])
}
```

### dropFirst rules

Same as XCTest — check the ViewModel implementation:
- `.dropFirst()` — skip 1 initial emission.
- `.dropFirst(2)` — skip 2 (ViewModel resets then sets the property).

---

## @MainActor for ViewModel Tests

- Annotate each `@Test` function with `@MainActor` when testing a ViewModel.
- `makeSUT()` must also be `@MainActor`.

```swift
@Test
@MainActor
func loadProfile_whenError_showsError() async { ... }

@MainActor
private func makeSUT(...) -> DomainViewModel { ... }
```

---

## Handling switch over RequestState

Use `#expect` with a boolean expression, or `try #require` to unwrap the associated value:

```swift
// Option 1 — inline boolean
if case .loaded(let response) = result {
    #expect(response.statusCode == 200)
} else {
    Issue.record("Expected .loaded but got \(result)")
}

// Option 2 — guard + Issue.record
guard case .loaded(let response) = result else {
    Issue.record("Expected .loaded")
    return
}
#expect(response.statusCode == 200)
```

---

## Parameterised Tests

Swift Testing supports parameterised tests natively. Use `@Test(arguments:)` for triangulation instead of writing three separate test functions:

```swift
@Test(arguments: [[], [item1], [item1, item2]])
func getItems_returnsCorrectCount(items: [Item]) async throws {
    let remote = ItemMockRemoteDataSource(items: items)
    let sut = makeSUT(remote: remote)

    let result = try await sut.getItems()

    if case .loaded(let response) = result {
        #expect(response.data?.count == items.count)
    } else {
        Issue.record("Expected .loaded")
    }
}
```

Only use parameterised tests when the same behaviour is expected for all inputs. When different outcomes are expected per case, write separate `@Test` functions.

---

## Key Differences from XCTest at a Glance

| Concern | XCTest | Swift Testing |
|---|---|---|
| Test class | `final class … : XCTestCase` | `@Suite struct …` |
| Test function | `func test*()` | `@Test func *()` |
| Equality | `XCTAssertEqual(a, b)` | `#expect(a == b)` |
| Unwrap | manual + `XCTAssertNotNil` | `try #require(optional)` |
| Fail | `XCTFail("msg")` | `Issue.record("msg")` |
| Async wait | `XCTestExpectation` + `fulfillment` | `confirmation` |
| Parameterised | three separate functions | `@Test(arguments:)` |
