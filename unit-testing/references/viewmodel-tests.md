# ViewModel Unit Test Patterns

## What to Test

- Test the **ViewModel** (`DomainViewModel`) in isolation.
- Inject mock repositories via `makeSUT()` — never use real repositories.
- Assert ViewModel `@Published` property changes using Combine `.sink` combined with the async waiting mechanism for the chosen framework.

---

## Mock Repository Structure

Mock repositories track invocations and return configurable results. This is the same regardless of framework:

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

## Coverage Checklist

For each ViewModel method under test, cover:

- [ ] Error state — repository returns `.error`, ViewModel sets `isError = true`
- [ ] Success state — repository returns `.loaded`, ViewModel sets the expected property
- [ ] Edge cases — missing required data, invalid input, guard clause failures
- [ ] Synchronous helpers — `isInputDataValid()`, `resetErrorState()`, etc.

---

## XCTest Examples

### @MainActor requirement

Every `func test*` and `makeSUT()` in a ViewModel test must be `@MainActor`.

### Error state (XCTest)

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

### Success state (XCTest)

```swift
@MainActor
func testOnLoadSubjects_whenSuccess_setsSubjects() async {
    let expectedSubjects = [
        Subject.Response.Subject(id: 1, name: "Technical Support"),
        Subject.Response.Subject(id: 2, name: "General Inquiry")
    ]
    let response = GeneralResponse(success: true, statusCode: 200, message: "Success", data: expectedSubjects)
    let subjectRepository = SubjectMockRepository(result: .loaded(response))
    let sut = makeSUT(subjectRepo: subjectRepository)
    var receivedSubjects = [Subject.Response.Subject]()
    let exp = expectation(description: "wait for subscription")
    exp.assertForOverFulfill = false
    let cancellable = sut.$subjects
        .dropFirst()
        .sink { subjects in
            receivedSubjects = subjects
            exp.fulfill()
        }

    await sut.loadSubjects()
    await fulfillment(of: [exp], timeout: 0.1)

    XCTAssertEqual(subjectRepository.invocations, [.getSubjects])
    XCTAssertEqual(receivedSubjects, expectedSubjects)
    cancellable.cancel()
}
```

### Synchronous property test (XCTest)

```swift
@MainActor
func test_isInputDataValid_emptySubjectAndMessage_returnsFalse() {
    let sut = makeSUT()
    sut.subject = ""
    sut.message = ""
    XCTAssertFalse(sut.isInputDataValid())
}
```

### makeSUT (XCTest)

```swift
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

### dropFirst rules (XCTest)

- `.dropFirst()` — skip 1 initial emission.
- `.dropFirst(2)` — skip 2 (ViewModel resets then sets, e.g., `isError = false` then `isError = true`).
- Check the ViewModel implementation to confirm the correct count.

---

## Swift Testing Examples

### @MainActor requirement

Annotate each `@Test` function and `makeSUT()` with `@MainActor`.

### Error state (Swift Testing)

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

### Success state (Swift Testing)

```swift
@Test
@MainActor
func loadSubjects_whenSuccess_setsSubjects() async {
    let expectedSubjects = [
        Subject.Response.Subject(id: 1, name: "Technical Support"),
        Subject.Response.Subject(id: 2, name: "General Inquiry")
    ]
    let response = GeneralResponse(success: true, statusCode: 200, message: "Success", data: expectedSubjects)
    let subjectRepository = SubjectMockRepository(result: .loaded(response))
    let sut = makeSUT(subjectRepo: subjectRepository)
    var receivedSubjects = [Subject.Response.Subject]()

    await confirmation("subjects emits expected value") { confirm in
        let cancellable = sut.$subjects
            .dropFirst()
            .sink { subjects in
                receivedSubjects = subjects
                confirm()
            }

        await sut.loadSubjects()
        cancellable.cancel()
    }

    #expect(subjectRepository.invocations == [.getSubjects])
    #expect(receivedSubjects == expectedSubjects)
}
```

### Synchronous property test (Swift Testing)

```swift
@Test
@MainActor
func isInputDataValid_emptySubjectAndMessage_returnsFalse() {
    let sut = makeSUT()
    sut.subject = ""
    sut.message = ""
    #expect(sut.isInputDataValid() == false)
}
```

### makeSUT (Swift Testing)

```swift
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

### dropFirst rules (Swift Testing)

Same as XCTest — `.dropFirst()` or `.dropFirst(2)` depending on how many times the ViewModel emits before the value you care about.

---

## Reference Existing Tests

If patterns are unclear, reference any existing `ViewModelTests` file in the project for the canonical implementation.
