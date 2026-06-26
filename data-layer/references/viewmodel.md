# ViewModel Layer

The ViewModel layer sits above the data layer. It is responsible for business logic and UI state — not for network calls or error mapping.

---

## Pattern

Use async/await MVVM with `ObservableObject`:

```swift
import Foundation
import SwiftUI
import Combine

@MainActor
final class DomainViewModel: ObservableObject {
    @Published var isError: Bool = false
    @Published var errorMessage: String?
    @Published var isExpired = false

    // State for each repository operation
    @Published var getItemsState: RequestState<GeneralResponse<[Domain.Response.Item]>> = .idle

    private let domainRepository: any DomainRepository

    init(domainRepository: some DomainRepository = DomainDefaultRepository()) {
        self.domainRepository = domainRepository
    }

    func loadItems() async {
        isError = false
        errorMessage = nil

        do {
            getItemsState = try await domainRepository.getItems(request: .init())
            switch getItemsState {
            case .idle, .loading, .refresh, .expired, .paginationLoading:
                break
            case .loaded(let response):
                // handle success
                break
            case .error(let errorResponse):
                handleError(errorResponse)
            }
        } catch {
            handleError(error)
        }
    }

    private func handleError(_ error: Error, useBackendErrorMessage: Bool = true) {
        // map error, set isExpired, isError, errorMessage
    }
}
```

---

## Rules

- Annotate the class with `@MainActor`
- Use `final class`, not `struct`
- Use `ObservableObject` with `@Published` properties
- Can import `SwiftUI` for `withAnimation` when toggling UI state smoothly:
  ```swift
  withAnimation { isError = true }
  ```
- Never call UI framework directly (no `UIKit` unless explicitly required)
- Never perform network requests directly — always go through the repository
- Inject repositories via `init` with a default concrete implementation

---

## Unit Testing

Test ViewModel state transitions using Combine + `XCTestExpectation`:

```swift
@MainActor
final class DomainViewModelTests: XCTestCase {

    func test_loadItems_success() async throws {
        let mockRepo = MockDomainRepository()
        let sut = DomainViewModel(domainRepository: mockRepo)

        let exp = expectation(description: "state published")
        var cancellables = Set<AnyCancellable>()

        sut.$getItemsState
            .dropFirst()
            .sink { state in
                if case .loaded = state {
                    exp.fulfill()
                }
            }
            .store(in: &cancellables)

        await sut.loadItems()
        await fulfillment(of: [exp], timeout: 0.1)
    }
}
```

Rules:
- Annotate test class with `@MainActor`
- Use `Combine` `.sink` to observe `@Published` properties
- Use `await fulfillment(of: [exp], timeout: 0.1)` — do NOT use `wait(for:timeout:)`
- Create mock implementations of repository protocols for isolation

---

## Common Mistakes to Flag

- Missing `@MainActor` on the class
- Calling `wait(for:timeout:)` in tests instead of `await fulfillment(of:timeout:)`
- Performing network calls directly in the ViewModel (bypassing repository)
- Missing `@Published` on state properties observed by the View
- Injecting concrete types instead of protocols in `init`
