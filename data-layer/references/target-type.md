# TargetType

File: `Data/Module/Domain/Remote/TargetType/DomainTargetType.swift`

Defines endpoint configuration for all cases in a domain.

---

## Enum Case Pattern

```swift
enum DomainTargetType {
    case getItems(Domain.Request.GetItems)
    case getItemDetail
}
```

Rules:
- Case name must match the repository function name exactly
- Prefix with HTTP intention: `get`, `create`, `update`, `delete`
- Use an associated value only if a request body or query parameters exist
- Case names must be camelCase

---

## Required Conformance

```swift
extension DomainTargetType: BaseTargetType, AccessTokenAuthorizable
```

---

## Authorization

- Default: `.bearer`
- Override only if the endpoint is public (unauthenticated)

---

## Headers

- GET requests → `"Accept": "application/json"`
- Requests with a body → also add `"Content-Type": "application/json"`

---

## Parameters

- Use `body.toJSON()` for request models that have an associated value
- Return `[:]` for endpoints with no parameters

---

## Task

Always use:

```swift
.requestParameters(parameters: parameters, encoding: URLEncoding.default)
```

---

## Path

- Must match the backend route exactly
- Use plural resource naming if backend uses plural

```swift
// Examples
"/users"
"/blogs"
"/entities/projects"
```

---

## Method

Explicitly define the HTTP method for each case:
- `.get`
- `.post`
- `.put`
- `.delete`

---

## sampleData

Rules:
- Must return a `GeneralResponse<Model>` structure
- Must mirror the backend contract
- Use realistic dummy values
- Keep the structure accurate to what the real API returns

---

## Common Mistakes to Flag

- Case name doesn't match the corresponding repository/remote function name
- Using associated value on a GET with no query params (should use `[:]`)
- Missing `AccessTokenAuthorizable` conformance on a secured endpoint
- `sampleData` returning an empty or incorrect structure
- Hardcoded path strings that don't match the backend contract
