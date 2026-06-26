# Models

File: `Data/Model/Domain/Domain.swift`

---

## Namespace Pattern

Always use a namespaced enum — never define request or response models at the top level:

```swift
enum Domain {
    enum Request {}
    enum Response {}
}
```

---

## Request Models

Rules:
- Must be defined inside `Domain.Request`
- Must conform to `Codable`
- Add `Equatable` only if necessary
- Always map snake_case fields using `CodingKeys`
- Properties must match the backend contract exactly

Naming convention by HTTP method:

| Action | Format |
|---|---|
| GET | GetX |
| POST | CreateX |
| PUT/PATCH | UpdateX |
| DELETE | DeleteX |

Example:

```swift
extension Domain.Request {
    struct GetItems: Codable {
        let userId: Int?

        enum CodingKeys: String, CodingKey {
            case userId = "user_id"
        }
    }
}
```

---

## Response Models

Rules:
- Must be defined inside `Domain.Response`
- Must conform to `Codable`
- Add `Equatable` and `Hashable` if used in UI state
- Map backend keys explicitly using `CodingKeys` if needed
- Model names must be singular

Example:

```swift
extension Domain.Response {
    struct Item: Codable, Equatable, Hashable {
        let id: Int?
        let name: String?
    }
}
```

---

## Common Mistakes to Flag

- Top-level structs not nested inside `Domain.Request` or `Domain.Response`
- Missing `CodingKeys` when backend uses snake_case
- Response models named in plural form (e.g., `Items` instead of `Item`)
- Conformances added without reason (e.g., `Hashable` on a model never used in a Set or as a dictionary key)
