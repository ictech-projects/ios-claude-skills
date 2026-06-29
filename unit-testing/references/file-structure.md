# File Structure Rules

## Test Target Name

- The unit test target is named `<AppName>Tests` — for example, if the app is `HRIS`, the test target is `HRISTests`.
- Always infer the test target name from the app name or from existing test files in the project. Never hardcode `HRISTests` — substitute the actual target name.

---

## Repository (Data Layer) Tests

Repository test files go under `Data/Module/<Domain>/` inside the test target:

```
<AppNameTests>/Data/Module/Authentication/AuthenticationDefaultRepositoryTests.swift
<AppNameTests>/Data/Module/Subject/SubjectDefaultRepositoryTests.swift
```

---

## ViewModel Tests

ViewModel test files go under `Feature/<FeatureName>/ViewModel/` inside the test target. The feature folder name matches the feature folder in the main target:

```
<AppNameTests>/Feature/ForgotPassword/ViewModel/ForgotPasswordEnterVerificationCodeViewModelTests.swift
<AppNameTests>/Feature/ContactUs/ViewModel/ContactUsViewModelTests.swift
```

There is no `Presentation/` folder — do not use it.

---

## Test Doubles / Mocks

- Mocks, stubs, and spy objects must **not** live alongside test files.
- They go under `Data/Mocks/<Domain>/` inside the test target. This folder typically already exists — check the project before creating it.

```
<AppNameTests>/Data/Mocks/Authentication/AuthenticationMockRemoteDataSource.swift
<AppNameTests>/Data/Mocks/Subject/SubjectMockRepository.swift
<AppNameTests>/Data/Mocks/Tenant/TenantMockRepository.swift
```

- One file per mock, named `<Domain>Mock<Type>.swift`.
- If the `Data/Mocks/` folder exists but has a different name, use the existing folder — do not create a new one.

---

## Summary

| File type | Location |
|---|---|
| Repository test | `<AppNameTests>/Data/Module/<Domain>/<Domain>DefaultRepositoryTests.swift` |
| ViewModel test | `<AppNameTests>/Feature/<FeatureName>/ViewModel/<ViewModelName>Tests.swift` |
| Mock / test double | `<AppNameTests>/Data/Mocks/<Domain>/<Domain>Mock<Type>.swift` |
