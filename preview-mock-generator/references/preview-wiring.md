# Preview Wiring

## Wiring a new mock into `#Preview`

The `#Preview` block constructs the View's ViewModel using the mock in place
of the real Repository, exactly the same way the View's default initializer
would take the real one — no special preview-only initializer path needed if
the ViewModel already takes its Repository as a default-arg constructor
parameter (ICT's standard DI pattern).

```swift
#Preview {
    ProfileView(viewModel: ProfileViewModel(repository: AuthenticationPreviewRepository()))
}
```

If the View has no `#Preview` block yet, add one using this shape rather than
inventing a different preview-construction style.

## Incremental patching

When a Repository protocol gains a new requirement and an existing
`*PreviewRepository` no longer conforms:

1. Add only the new method/property to the existing file, in the same style
   (naming, fixture realism) as what's already there.
2. Do not reorder or reformat existing members while you're in the file — the
   diff should show only the addition.
3. Do not touch the `#Preview` block itself unless the View's initializer
   signature changed too — patching the mock's conformance and patching the
   View's preview call site are two independent changes; only do the second
   if it's actually needed.

## Multiple Repositories per ViewModel

If a ViewModel depends on more than one Repository, generate/patch each
mock independently, then wire all of them into the same `#Preview` block's
ViewModel construction — don't skip previewing a screen just because it has
more than one dependency.
