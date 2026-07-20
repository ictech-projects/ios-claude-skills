# 📦 [CLAUDE] IPA Export (Debugging Distribution)

This replicates, from the command line, the manual flow QA usually asks for:

> Xcode → select scheme/device → Organizer → select build → Distribute App → Custom → Debugging → App Thinning (all compatible device variants) → Automatically manage signing → Export → `~/Downloads`.

This is **optional** and only runs if the developer asks for it after the release branch is created. Always confirm which scheme(s) before starting — `xcodebuild archive` can take several minutes per scheme and this writes files to the developer's `~/Downloads`.

## 1. Ask which scheme(s)

Offer the schemes found via `xcodebuild -list` (typically a `-Dev` scheme and the main/production scheme). The developer may pick one or both. Confirm before starting each archive — don't run all of them silently.

## 2. Read signing info dynamically

Do not hardcode a team ID or bundle identifier. For the target scheme's `Release` configuration, read from `project.pbxproj`:

- `DEVELOPMENT_TEAM` — becomes `teamID` in the export options.
- Confirm `CODE_SIGN_STYLE` is `Automatic` for that configuration (matches "Automatically manage signing" in the Organizer flow). If it's `Manual`, tell the developer — automatic export options won't match a manually-signed target and the export step will likely need adjusting.

## 3. Archive

```
xcodebuild archive \
  -project <Project>.xcodeproj \
  -scheme <Scheme> \
  -configuration Release \
  -archivePath <build-dir>/<Scheme>-v<version>.xcarchive
```

Use a scratch build directory (e.g. a temp dir), not a path inside the repo — archives should never be committed.

## 4. Write the export options plist

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

## 5. Export

```
xcodebuild -exportArchive \
  -archivePath <build-dir>/<Scheme>-v<version>.xcarchive \
  -exportPath ~/Downloads/<Scheme>-v<version> \
  -exportOptionsPlist <build-dir>/exportOptions.plist
```

Resulting `.ipa` lands at `~/Downloads/<Scheme>-v<version>/<AppName>.ipa`.

## 6. Report back

Tell the developer the exact `.ipa` path(s) produced, per scheme. If `xcodebuild` fails (commonly a missing/expired signing certificate or provisioning profile), report the actual error output — do not attempt to fix signing/certificate issues yourself.
