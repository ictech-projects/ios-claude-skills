# Repository Layer

Files:
- Protocol: `Data/Module/Domain/DomainRepository.swift`
- Implementation: `Data/Module/Domain/DomainDefaultRepository.swift`

The repository is the only layer that handles error mapping and wraps results in `RequestState`.

---

## Protocol

```swift
protocol DomainRepository {
    func getItems(
        request: Domain.Request.GetItems
    ) async throws -> RequestState<GeneralResponse<[Domain.Response.Item]>>
}
```

Rules:
- Must mirror the `RemoteDataSource` methods exactly (same function names and parameters)
- Return type must be `RequestState<GeneralResponse<T>>`
- Functions must be `async throws`

---

## Default Implementation

```swift
struct DomainDefaultRepository: DomainRepository {

    private let remote: any DomainRemoteDataSource

    init(remoteDataSource: some DomainRemoteDataSource = DomainDefaultRemoteDataSource()) {
        self.remote = remoteDataSource
    }

    func getItems(
        request: Domain.Request.GetItems
    ) async throws -> RequestState<GeneralResponse<[Domain.Response.Item]>> {
        do {
            let result = try await remote.getItems(request: request)
            return .loaded(result)
        } catch let error as ErrorResponse {
            return .error(error)
        } catch {
            return .error(error)
        }
    }
}
```

Rules:
- Map success result → `.loaded(result)`
- Map `ErrorResponse` failures → `.error(error)`
- Map generic failures → `.error(error)`
- Do not perform UI logic — no `withAnimation`, no `@Published`, no `DispatchQueue`
- Use `struct`, not `class`
- The `init` parameter label is `remoteDataSource:` with a default of `DomainDefaultRemoteDataSource()`
- Use `private let remote: any DomainRemoteDataSource` (existential type)

---

## Local Storage (Optional)

If the domain requires local persistence (e.g., Authentication storing a token), inject a local data source alongside the remote:

```swift
struct DomainDefaultRepository: DomainRepository {

    private let remote: any DomainRemoteDataSource
    private let local: any DomainLocalDataSource

    init(
        remoteDataSource: some DomainRemoteDataSource = DomainDefaultRemoteDataSource(),
        localDataSource: some DomainLocalDataSource = DomainDefaultLocalDataSource()
    ) {
        self.remote = remoteDataSource
        self.local = localDataSource
    }
}
```

Add the Local files under `Data/Module/Domain/Local/`.

---

## Common Mistakes to Flag

- Returning raw `GeneralResponse` instead of `RequestState` (should be wrapped)
- Missing the second `catch` block for generic errors
- Using `class` instead of `struct`
- UI logic inside the repository (animations, state publishing)
- Function name doesn't match Remote/TargetType names
- Not propagating `ErrorResponse` as a distinct catch case
