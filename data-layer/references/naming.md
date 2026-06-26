# Naming Conventions

All naming must align exactly across every layer. No deviations are allowed.

---

## Per-Layer Rules

| Layer | Rule |
|---|---|
| Domain name | Singular PascalCase (e.g., `User`, `Blog`, `ContactMessage`) |
| File names | Always start with the domain name |
| Concrete implementations | Must use the `Default` prefix (e.g., `UserDefaultRepository`) |
| Request model | Verb + Noun in PascalCase (e.g., `GetItems`, `CreateUser`) |
| Response model | Singular noun in PascalCase (e.g., `Item`, `User`) |
| TargetType case | camelCase with HTTP verb prefix (e.g., `getItems`, `createUser`) |
| Remote function | Same as TargetType case |
| Repository function | Same as Remote function |
| Path | Match backend route exactly |

---

## HTTP Verb Prefix Mapping

| HTTP Method | Prefix |
|---|---|
| GET | `get` |
| POST | `create` |
| PUT / PATCH | `update` |
| DELETE | `delete` |

---

## Cross-Layer Consistency Check

When reviewing, verify that all four of these agree for every endpoint:

1. TargetType case name
2. RemoteDataSource function name
3. Repository function name
4. Request model name (verb matches the function prefix)

Example — all four must align:

```
TargetType case:          .getItems(Domain.Request.GetItems)
Remote function:          func getItems(request: Domain.Request.GetItems)
Repository function:      func getItems(request: Domain.Request.GetItems)
Request model:            struct GetItems: Codable
```

Any mismatch across these four is a naming violation.

---

## File Naming Examples

For domain `ContactMessage`:

| File | Correct Name |
|---|---|
| Model | `ContactMessage.swift` |
| Repository protocol | `ContactMessageRepository.swift` |
| Repository impl | `ContactMessageDefaultRepository.swift` |
| Remote protocol | `ContactMessageRemoteDataSource.swift` |
| Remote impl | `ContactMessageDefaultRemoteDataSource.swift` |
| TargetType | `ContactMessageTargetType.swift` |

---

## Common Mistakes to Flag

- Domain name in plural form (e.g., `UsersRepository` instead of `UserRepository`)
- Concrete impl missing `Default` prefix
- Mismatch between TargetType case name and Remote/Repository function name
- Request model verb doesn't match the HTTP method (e.g., `StoreUser` for a POST when `CreateUser` is the convention)
- Response model in plural form (e.g., `Items` instead of `Item`)
