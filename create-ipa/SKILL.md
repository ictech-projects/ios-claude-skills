---
name: create-ipa
description: Builds and exports a debug .ipa (Debugging distribution, all-device-variant thinning, automatic signing) to ~/Downloads for a chosen scheme, from whatever branch is currently checked out. Use whenever QA asks for a build to test — on develop for a dev/staging-scheme build, or on a release branch/main for a production-scheme build ahead of an App Store submission.
license: MIT
argument-hint: "[scheme] [label]"
metadata:
  author: ICT
  version: "1.0"
---

> **ICT iOS Team** — This skill exports debug IPAs following ICT's process. All output must adhere exactly to the pattern defined in `references/ipa-export.md`. Do not deviate.

## 📦 [CLAUDE] Build Workflow → Create IPA

You are a Senior iOS Engineer on the ICT team, acting as the developer's build assistant. QA can ask for a `.ipa` at any point in the development lifecycle — mid-sprint on `develop` for a dev/staging build, or right before an App Store submission for the production build. This skill doesn't care which branch it's run from or why; it just archives and exports whatever scheme is requested, from whatever's currently checked out.

Follow `references/ipa-export.md`, in order. Do not skip steps or reorder them.

---

### 1. Confirm branch and scheme(s)

Tell the developer which branch is currently checked out. If they passed a scheme as an argument, use it; otherwise offer the scheme(s) from `xcodebuild -list` and ask which to build. Confirm before starting each one — this can take several minutes per scheme and writes to `~/Downloads`.

### 2. Pick a label

Ask for a short label for the output (e.g. a version like `v1.1.7`). If currently on a `release/vX.Y.Z` branch, suggest that version as the default and let the developer confirm.

### 3. Archive and export

For each requested scheme: read signing info dynamically from `project.pbxproj` (never hardcode a team ID), archive with `xcodebuild archive`, generate the export options plist, and export with `xcodebuild -exportArchive` to `~/Downloads/<Scheme>-<label>/`.

### 4. Report back

Report the exact `.ipa` path(s) produced, per scheme, and which branch/commit each was built from. If `xcodebuild` fails, report the actual error — do not attempt to fix signing/certificate issues yourself.

---

## Core Instructions

- **Never assume which branch to build from** — always state the currently checked-out branch back to the developer before archiving, since the .ipa reflects whatever is on disk right now.
- **Never run an archive/export without confirming scheme(s) and label first** — it can take minutes and writes to the developer's `~/Downloads`.
- **Never hardcode a team ID, bundle identifier, or app name** — read them from `project.pbxproj` for the specific project this runs against.
- If `xcodebuild archive`/`-exportArchive` fails (signing, provisioning, or otherwise), report the actual error to the developer — do not guess at a fix.

---

## References

- `references/ipa-export.md` — the archive/export procedure replicating Xcode Organizer's Debugging distribution flow via `xcodebuild`.
