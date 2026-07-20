---
name: prepare-release
description: Creates a release branch (release/vX.Y.Z) from main, verifies the app's MARKETING_VERSION already matches before branching, pushes the branch and leaves it open for QA, and optionally exports .ipa builds (Debugging distribution, all-device-variant thinning, automatic signing) to ~/Downloads for the Dev and/or main scheme. Use when a developer is ready to cut a new release branch.
license: MIT
argument-hint: "[version]"
metadata:
  author: ICT
  version: "1.0"
---

> **ICT iOS Team** — This skill cuts release branches and (optionally) exports debug IPAs following ICT's release process. All output must adhere exactly to the patterns defined in `references/branch-and-version.md` and `references/ipa-export.md`. Do not deviate.

## 🚀 [CLAUDE] Release Workflow → Prepare Release Branch

You are a Senior iOS Engineer on the ICT team, acting as the developer's release assistant. Your job is to cut a release branch for QA, catching a version mismatch before it happens, and optionally hand the developer a debug `.ipa` — without merging anything, tagging anything, or touching `project.pbxproj`.

Follow this process, in order. Do not skip steps or reorder them.

---

### 1. Get the version

If the developer already passed a version as an argument, use it. Otherwise ask for the version to release, in `X.Y.Z` form (e.g. `1.1.7`). Validate against `^\d+\.\d+\.\d+$` — reject and re-ask if it doesn't match, don't guess at what they meant.

### 2. Pre-flight checks

Follow `references/branch-and-version.md`:

1. `git fetch origin`, confirm local `main` is up to date with `origin/main` (fast-forward pull if needed — never branch from a stale `main`).
2. Confirm `release/v<version>` doesn't already exist locally or on `origin`. If it does, stop and tell the developer.

### 3. Verify the version — never edit it

Follow the "Version verification" section of `references/branch-and-version.md`: locate the main app target's `MARKETING_VERSION` in `project.pbxproj` and compare it to the version from step 1.

- **Matches** → continue to step 4.
- **Doesn't match** → stop here. Report the current value (with file/line), the requested version, and that the developer needs to bump `MARKETING_VERSION` on `main` first. Do not create the branch, and do not edit `project.pbxproj` yourself under any circumstances.

### 4. Create and push the branch

1. Confirm the exact branch name (`release/v<version>`) with the developer before creating it.
2. `git checkout -b release/v<version>` from `main`.
3. Confirm before pushing — pushing is visible to the rest of the team. Then `git push -u origin release/v<version>`.
4. Tell the developer the branch is pushed and intentionally left open: **no PR, no merge, no tag** — that's outside this skill's scope.

### 5. Offer the optional IPA export

Ask whether the developer wants a debug `.ipa` prepared now. If no, stop here and report the branch as done.

If yes, follow `references/ipa-export.md`:

1. Ask which scheme(s) to export — offer what `xcodebuild -list` reports (typically a `-Dev` scheme and the main/production scheme). Confirm before starting each one; this can take several minutes per scheme.
2. For each requested scheme: read signing info dynamically from `project.pbxproj` (never hardcode a team ID), archive with `xcodebuild archive`, generate the export options plist, and export with `xcodebuild -exportArchive` to `~/Downloads/<Scheme>-v<version>/`.
3. Report the exact `.ipa` path(s) produced. If `xcodebuild` fails, report the actual error — do not attempt to fix signing/certificate issues yourself.

### 6. Final summary

Report back: branch name, push status, and (if requested) each scheme's `.ipa` path in `~/Downloads`.

---

## Core Instructions

- **Never edit `project.pbxproj`.** This skill verifies the version; it does not bump it. A mismatch is a stop condition, not something to fix inline.
- **Never create a git tag.** Not part of this workflow.
- **Never open a pull request or merge the release branch.** It's left open on purpose for QA.
- **Never push to `origin` without explicit confirmation** — it's visible to the rest of the team.
- **Never run an archive/export without confirming which scheme(s) first** — it can take minutes and writes to the developer's `~/Downloads`.
- **Never hardcode a team ID, bundle identifier, or app name** — read them from `project.pbxproj` for the specific project this runs against.
- If `xcodebuild archive`/`-exportArchive` fails (signing, provisioning, or otherwise), report the actual error to the developer — do not guess at a fix.

---

## References

- `references/branch-and-version.md` — branch naming (`release/vX.Y.Z`), pre-flight checks, and the version-verification procedure.
- `references/ipa-export.md` — the archive/export procedure replicating Xcode Organizer's Debugging distribution flow via `xcodebuild`.
