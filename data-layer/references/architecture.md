# Architecture

## Layered Structure

Each feature follows this strict layered structure:

```
Model (Request/Response)
        ↓
TargetType (Endpoint definition)
        ↓
RemoteDataSource (Raw network call)
        ↓
Repository (Error mapping + RequestState wrapping)
```

Each layer has a single responsibility and must not leak logic into other layers.

---

## Required File Structure

A "Domain" is a singular PascalCase name matching the BE API documentation domain (e.g., Authentication, ContactMessage, EmployeeProfile).

For a new Domain, files must be created following this structure:

```
APPNAME/
└── Data/
    ├── Model/
    │   └── Domain/
    │       └── Domain.swift
    └── Module/
        └── Domain/
            ├── DomainRepository.swift
            ├── DomainDefaultRepository.swift
            └── Remote/
                ├── DomainRemoteDataSource.swift
                ├── DomainDefaultRemoteDataSource.swift
                └── TargetType/
                    └── DomainTargetType.swift
```

If the domain requires local storage (e.g., Authentication), add:

```
        └── Local/
            ├── DomainLocalDataSource.swift
            └── DomainDefaultLocalDataSource.swift
```

---

## Responsibility Separation

| Layer | Responsibility |
|---|---|
| Model | API contract definition |
| TargetType | Endpoint configuration |
| RemoteDataSource | Execute network request |
| Repository | Error mapping + state wrapping |
| ViewModel | Business logic + UI state |

Never mix responsibilities across layers.

---

## Enforcement

Any new API feature must:
- Follow the file structure above
- Follow naming conventions
- Maintain separation of concerns
- Return correct wrapper types (`GeneralResponse` at Remote level, `RequestState` at Repository level)
- Mirror the backend contract exactly

If a feature does not follow this structure, it must be refactored to comply.
