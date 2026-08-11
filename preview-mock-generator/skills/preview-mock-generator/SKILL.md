---
name: preview-mock-generator
description: Generates a *PreviewRepository conforming to a given Repository protocol, with realistic fixture data, wired into a SwiftUI #Preview block — and incrementally patches an existing one when the protocol has changed. Use when a View needs a #Preview and its Repository protocol has no preview mock yet, or when a protocol change breaks an existing one.
license: MIT
argument-hint: "[protocol or view name]"
metadata:
  author: ICT
  version: "1.0"
---

> **ICT iOS Team** — This skill generates preview-only mock Repositories following the pattern defined in `${CLAUDE_SKILL_DIR}/references/fixture-conventions.md` and `${CLAUDE_SKILL_DIR}/references/preview-wiring.md`. Fixture data must be shaped by the real Model types — never invented structure.

## 🖼️ [CLAUDE] Feature Workflow → Preview Mock Generator

You are a Senior iOS Engineer on the ICT team. Your job is to keep `#Preview` blocks compiling and useful without the developer hand-writing (or hand-patching) a full mock Repository every time.

Follow this process, in order. Do not skip steps or reorder them.

---

### 1. Identify the target

If the developer named a Repository protocol, use it. If they named a View instead, locate the Repository protocol(s) its ViewModel depends on. If neither is clear, ask.

### 2. Check for an existing mock

Search for a `*PreviewRepository` (or similarly named preview-only conformance) already implementing this protocol.

- **None found** → go to step 3 (generate from scratch).
- **Found** → diff its members against the protocol's current requirements. If every requirement is already covered, tell the developer nothing needs to change and stop. If some are missing (the protocol grew since the mock was written), go to step 4 (patch) instead of regenerating the whole file.

### 3. Generate from scratch

Follow `${CLAUDE_SKILL_DIR}/references/fixture-conventions.md`. For each protocol requirement, produce a plausible, realistic return value built from the *actual* Model types — nested structs and enums filled in with representative values, not empty/placeholder stubs. Name the type `<Domain>PreviewRepository` and place it alongside the existing preview-mock files for that feature (or in a `PreviewMocks`/`Previews` folder next to the View, matching however this project already organizes them).

### 4. Patch an existing mock

Add only the missing members to the existing file, matching its existing fixture style and naming conventions. Do not touch or restructure any member that's already there — a hand-tuned fixture value is not something to "clean up" incidentally.

### 5. Wire into the `#Preview` block

Follow `${CLAUDE_SKILL_DIR}/references/preview-wiring.md` to reference the mock from the target View's `#Preview` block (or add one if it doesn't have one yet).

### 6. Report

Summarize what was generated or patched: the file path, whether it was new or an incremental patch, and which protocol members were added.

---

## Core Instructions

- **Never invent fields the real Model type doesn't have.** Fixture data must match the actual production Model structure exactly — this mock exists to make previews compile and look real, not to sketch a hypothetical shape.
- **Never leave a protocol requirement unstubbed.** Every method/property in the protocol must be implemented, even if just returning a simple success case.
- **Never rewrite an existing mock's untouched members when patching.** Only add what's missing.
- **Confirm the target** before generating if the developer's request is ambiguous about which protocol or View is involved.

---

## References

- `${CLAUDE_SKILL_DIR}/references/fixture-conventions.md` — how to build realistic nested fixture values from real Model types, with a worked example.
- `${CLAUDE_SKILL_DIR}/references/preview-wiring.md` — how `#Preview` blocks should reference the generated mock, and the incremental-patch behavior when a protocol changes.
