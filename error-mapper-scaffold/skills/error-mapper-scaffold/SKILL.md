---
name: error-mapper-scaffold
description: Discovers ICT's existing error-mapping convention (handleErrorResponseUsingDefaultFlow + a per-feature *ErrorMapper) and, given a target ViewModel/Repository, generates the mapper and wires the ViewModel to use it — or patches an existing mapper with only the missing mappings. Use when a new ViewModel needs error handling, or an existing one's error surface has grown.
license: MIT
argument-hint: "[ViewModel or Repository name]"
metadata:
  author: ICT
  version: "1.0"
---

> **ICT iOS Team** — This skill generates error-mapping boilerplate following the pattern defined in `${CLAUDE_SKILL_DIR}/references/error-mapping-pattern.md`. It never bypasses the shared 401→logout flow, and never invents error codes the target Repository doesn't actually produce.

## ⚠️ [CLAUDE] Feature Workflow → Error Mapper Scaffold

You are a Senior iOS Engineer on the ICT team. Your job is to give a ViewModel consistent error handling without hand-copying it from whichever ViewModel the developer had open last.

Follow this process, in order. Do not skip steps or reorder them.

---

### 1. Identify the target

If the developer named a ViewModel or Repository, use it. If neither is clear, ask.

### 2. Discover the existing convention

Follow `${CLAUDE_SKILL_DIR}/references/error-mapping-pattern.md` to locate the `handleErrorResponseUsingDefaultFlow`-style method each ViewModel hand-rolls, and determine which of two shapes applies to the target:

1. **Repository already isolates expiry** — its response type has a dedicated `.expired`-style case, handled by the ViewModel's state switch before the generic error handler ever runs. Here, `handleErrorResponseUsingDefaultFlow` only needs to set the plain error flag/message — no dedicated mapper needed.
2. **Repository throws a raw `Error`** — the ViewModel needs a dedicated `<Feature>ViewModelErrorMapper` (a `static func map(_ error: Error) -> ViewModelErrorResult`) that inspects the error's status code itself, including detecting 401 as expiry.

Check the target Repository's actual method signatures to tell which shape applies — don't assume; a Repository that already returns a state enum with `.expired` should not also get a redundant dedicated mapper.

If neither shape is recognizable at all, **stop** — this project may use a different error-handling convention entirely. Report what you found and ask, rather than inventing a new pattern.

### 3. Check for an existing mapper

Search for a mapper already covering this ViewModel's Repository.

- **None found** → go to step 4 (generate from scratch).
- **Found** → compare it against the Repository/TargetType's actual current error surface. If everything is already covered, tell the developer and stop. If new error cases exist that aren't mapped yet, go to step 5 (patch) instead of regenerating.

### 4. Generate from scratch

If the target needs a dedicated mapper (shape 2 above), create `<Feature>ViewModelErrorMapper.swift` following the discovered shape exactly — same `ViewModelErrorResult` return type, same status-code switch style, same flag/message naming as the worked examples in `${CLAUDE_SKILL_DIR}/references/error-mapping-pattern.md`. If the target's Repository already isolates expiry via a state-enum case (shape 1), just generate the plain `handleErrorResponseUsingDefaultFlow` — no mapper file needed. Either way, 401/expiry must always resolve to the same shared `isExpired` → `authenticationRepository.eraseToken(...)` → logout path every other ViewModel uses; never write a second, competing token-erase implementation.

### 5. Patch an existing mapper

Add only the new error-code mappings the current error surface actually needs. Don't touch or restructure any mapping that's already there.

### 6. Wire the ViewModel

Update the ViewModel's failure-handling call site to use the mapper, matching how the worked examples call theirs.

### 7. Report

Summarize: file created or patched, which error codes are now mapped, and confirmation that 401 routes through the shared logout flow (not a new one-off implementation).

---

## Core Instructions

- **Never invent an error code the Repository/TargetType doesn't actually produce.** Base every mapping on the real error surface, not a guess.
- **Never duplicate the 401 → `isExpired` → token-erase → logout flow.** Always route through the existing shared path; a second implementation of this is a security-relevant bug, not just duplication.
- **Never rewrite an existing mapper's untouched mappings when patching.**
- **Stop and ask if no existing convention is found** — don't retrofit this pattern onto a project that doesn't already use it.

---

## References

- `${CLAUDE_SKILL_DIR}/references/error-mapping-pattern.md` — the discovered convention shape (`handleErrorResponseUsingDefaultFlow` + `*ErrorMapper`), the shared 401→logout special case, and worked examples to extend for a new domain.
