# 🌿 [CLAUDE] Release Branch Naming & Version Verification

## Branch naming convention

The release branch must match:

```
release/v<MAJOR>.<MINOR>.<PATCH>
```

**Regex:** `^release/v\d+\.\d+\.\d+$`

**Valid examples:**

```
release/v1.1.7
release/v2.0.0
```

**Invalid examples:**

```
release/1.1.7          (missing the v)
release/v1.1            (not three components)
feature/release/v1.1.7  (wrong prefix — this is the historical outlier, not the convention)
```

This is a flat naming convention — there is no ticket segment and no ICT commit-tag validation (unlike `create-pull-request-against-development`'s branch convention). Release branches are not tied to a single backlog ticket.

## Deriving the version

Ask the developer for the version number in `X.Y.Z` form (e.g. `1.1.7`). Validate it against `^\d+\.\d+\.\d+$` before doing anything else — if it doesn't match, ask again rather than guessing what they meant.

The branch name is always `release/v<version>`.

## Pre-flight checks before creating the branch

1. `git fetch origin` and confirm the local `main` is up to date with `origin/main`. If it's behind, pull it (fast-forward only) before branching — never branch from a stale `main`.
2. Confirm `release/v<version>` does not already exist locally or on `origin`. If it does, stop and tell the developer — do not overwrite or reuse it.

## Version verification (verify only — never edit)

This skill **never modifies** `project.pbxproj`. Version bumps are a separate, earlier step owned by the developer on `main`. This skill's job is only to catch a mismatch before a release branch gets cut from the wrong version.

1. Find the app's `.xcodeproj/project.pbxproj`.
2. Identify the **main app target** — the one whose product type is `com.apple.product-type.application` and whose name matches the project's schemes (e.g. a project with `HRIS` / `HRIS-Dev` schemes has one underlying app target, possibly with per-scheme configurations). Ignore test targets, extensions, and widget targets unless the developer says those should be checked too.
3. For that target's `Debug` and `Release` build configurations, read `MARKETING_VERSION`.
4. Compare it to the version the developer entered.
   - **Matches** → proceed to branch creation.
   - **Doesn't match** → stop. Tell the developer the current `MARKETING_VERSION` you found (with file/line), the version they asked to release, and that they need to bump it on `main` first (via Xcode's target settings or `agvtool new-marketing-version <version>`) before running this skill again. Do not create the branch.
5. `CURRENT_PROJECT_VERSION` (the build number) is informational only — do not validate it. Many CI setups (Xcode Cloud, etc.) stamp this automatically at archive time from a CI-provided build number, so its committed value is expected to be stale between releases.

## After creating the branch

- Push it to `origin` immediately (`git push -u origin release/v<version>`) — confirm with the developer first, since pushing is visible to the rest of the team.
- **Do not open a pull request.** The branch is intentionally left open against `main` for QA/manual testing. Merging or PR-ing it back is a separate, later step outside this skill's scope.
- **Do not create a git tag.** Tagging is not part of this workflow.
