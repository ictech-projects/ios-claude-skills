# 📦 [CLAUDE] IPA Export (Debugging Distribution)

This replicates, from the command line, the manual flow QA usually asks for:

> Xcode → select scheme/device → Organizer → select build → Distribute App → Custom → Debugging → App Thinning (all compatible device variants) → Automatically manage signing → Export → `~/Downloads`.

This works from **whatever branch is currently checked out** — there is no dependency on being on a release branch. QA may ask for this on `develop` (a dev/staging-scheme build) just as often as on a release branch or `main` (the production-scheme build ahead of an App Store submission).

Always confirm which scheme(s) before starting — `xcodebuild archive` can take several minutes per scheme and this writes files to the developer's `~/Downloads`.

## 1. Confirm the branch and scheme(s)

1. Tell the developer which branch is currently checked out (`git branch --show-current`) — the archive will build from whatever is on disk right now, so if they meant a different branch, have them check it out first.
2. Offer the scheme(s) found via `xcodebuild -list`. The developer may pick one or both. Confirm before starting each archive — don't run all of them silently.

## 2. Pick a label for the output

Ask for a short label to name the output folder/archive with (e.g. a version like `v1.1.7`, or a ticket/branch-derived name). If the developer is on a `release/vX.Y.Z` branch, default to suggesting that version and let them confirm rather than asking from scratch.

## 3. Confirm the build number

**A local `xcodebuild archive` never talks to any CI build-number counter (Xcode Cloud, Fastlane, etc.).** Whatever `CURRENT_PROJECT_VERSION` is currently committed in `project.pbxproj` is exactly what gets baked into the exported `.ipa` — no CI is in the loop to override it here, unlike a build produced by the project's normal CI pipeline. That committed value is commonly stale between releases (see `prepare-release`'s version-verification note, which explicitly treats it as informational for that reason) — and a stale build number can trip a server-side minimum-build gate (e.g. a forced-update prompt) even though the marketing version looks correct.

Before archiving:

1. Read the current `CURRENT_PROJECT_VERSION` from `project.pbxproj` for the target scheme's configuration and show it to the developer.
2. Ask the developer for the latest known build number from whichever build environment this `.ipa` is meant to line up with (e.g. "what's the latest Xcode Cloud build number for this branch?", or a TestFlight/App Store Connect build number) — don't guess or assume the committed value is current.
3. Compare the two:
   - **Committed value is already ≥ the latest known build** → proceed to archive as-is.
   - **Committed value is stale/lower** → offer to bump `CURRENT_PROJECT_VERSION` in `project.pbxproj` (for the target scheme's Debug/Release — or this project's equivalent, e.g. Development/Production — configurations) to match or exceed the number the developer gave. Get explicit confirmation before editing `project.pbxproj` — this is a real, shared-file edit, not a read-only check. Match an *existing* CI-built number exactly when the developer wants this local `.ipa` to represent the same build (e.g. for QA parity with a TestFlight build); otherwise pick something safely above the latest known number to avoid a future collision.
   - **Developer doesn't know the latest number** → proceed with the committed value, but say plainly in the final report that the build number was not cross-checked against any CI/build-tracking system and may trip a minimum-build gate.

## 4. Read signing info dynamically

Do not hardcode a team ID or bundle identifier. For the target scheme's `Release` configuration (or this project's equivalent naming, e.g. `Production` — check `xcodebuild -list`'s "Build Configurations" and the scheme's Archive action), read from `project.pbxproj`:

- `DEVELOPMENT_TEAM` — becomes `teamID` in the export options.
- Confirm `CODE_SIGN_STYLE` is `Automatic` for that configuration (matches "Automatically manage signing" in the Organizer flow). If it's `Manual`, tell the developer — automatic export options won't match a manually-signed target and the export step will likely need adjusting.

## 5. Archive

```
xcodebuild archive \
  -project <Project>.xcodeproj \
  -scheme <Scheme> \
  -configuration Release \
  -archivePath <build-dir>/<Scheme>-<label>.xcarchive
```

Use a scratch build directory (e.g. a temp dir), not a path inside the repo — archives should never be committed.

## 6. Write the export options plist

Generate this alongside the archive (scratch location, not committed):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>method</key>
    <string>debugging</string>
    <key>teamID</key>
    <string><!-- DEVELOPMENT_TEAM read from project.pbxproj --></string>
    <key>signingStyle</key>
    <string>automatic</string>
    <key>thinning</key>
    <string>&lt;thin-for-all-variants&gt;</string>
</dict>
</plist>
```

**Verify the `method` value once against a real manual export before relying on this.** Apple has renamed distribution methods across Xcode versions (`development` vs `debugging` for on-device debug installs); if a local `xcodebuild -exportArchive` run rejects `debugging` as an unknown method for the installed Xcode version, fall back to `development` and note that in the output to the developer, don't silently guess further.

## 7. Export

```
xcodebuild -exportArchive \
  -archivePath <build-dir>/<Scheme>-<label>.xcarchive \
  -exportPath ~/Downloads/<Scheme>-<label> \
  -exportOptionsPlist <build-dir>/exportOptions.plist
```

Resulting `.ipa` lands at `~/Downloads/<Scheme>-<label>/Apps/<AppName>.ipa` (app-thinning exports nest the actual `.ipa`(s) under an `Apps/` subfolder — don't assume it's directly at the export path root).

## 8. Verify and report back — from the `.ipa` itself, not the label

Never trust the label, folder name, or committed `project.pbxproj` value as the source of truth for what actually shipped — any of those can be stale, or a local watcher/rename script can rewrite the output folder name to something that looks authoritative but isn't. Instead, read the real values back out of the exported artifact:

```
unzip -q <exportPath>/Apps/<AppName>.ipa -d <scratch-dir>/extracted
/usr/libexec/PlistBuddy -c "Print :CFBundleShortVersionString" -c "Print :CFBundleVersion" \
  <scratch-dir>/extracted/Payload/<AppName>.app/Info.plist
```

Report both values back to the developer alongside the `.ipa` path(s), per scheme, and which branch/commit it was built from. If they don't match what was expected (from step 3, or from the marketing-version label), say so explicitly rather than assuming the export succeeded correctly. If `xcodebuild` fails (commonly a missing/expired signing certificate or provisioning profile), report the actual error output — do not attempt to fix signing/certificate issues yourself.
