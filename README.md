# ios-claude-skills

A [Claude Code](https://claude.ai/code) skill marketplace for the ICT iOS team. Provides eleven iOS-specific skills covering the full development lifecycle — data layer generation, SwiftUI views, unit testing, security review, localization, pull request creation, release preparation, IPA builds, navigation wiring, and design-token compliance.

---

## Skills

| Skill | Command | Status | Description |
|---|---|---|---|
| Data Layer | `/data-layer` | Available | Creates or reviews the Data layer for a domain (Model → TargetType → RemoteDataSource → Repository) |
| Security Review | `/security-review` | Available | Comprehensive iOS security assessment against OWASP MASVS and Apple security guidelines |
| SwiftUI Reviewer | `/swiftui-reviewer` | Available | Reviews SwiftUI code for best practices, modern APIs, and performance |
| SwiftUI View Gen | `/swiftui-view-gen` | Available | Generates SwiftUI views from Figma designs, handles state management, animations, and Liquid Glass adoption |
| Unit Testing | `/unit-testing` | Available | Writes or reviews unit tests for Repository and ViewModel layers (XCTest + Swift Testing) |
| Localization Review | `/localization-review` | Available | Reviews code for String Catalog (`.xcstrings`) coverage, flags legacy localization APIs, and mechanically converts flagged strings on approval — never generates translations |
| Create Pull Request | `/create-pull-request-against-development` | Available | Opens a PR against the development branch, validating branch naming and the commit ticket-tag format, using ICT's MR template (Overview + Ticket + Checklist) |
| Prepare Release | `/prepare-release` | Available | Creates a `release/vX.Y.Z` branch from main, verifies the app version before branching, and optionally hands off to `create-ipa` for a production build |
| Create IPA | `/create-ipa` | Available | Builds and exports a debug `.ipa` to `~/Downloads` for a chosen scheme, from whatever branch is currently checked out — usable any time QA asks for a build |
| Add Navigation Route | `/add-navigation-route` | Available | Wires a new screen into ICT's enum-based navigation system — route case, render switch, and entry-point call site |
| Design Token Usage | `/design-token-usage` | Available | Reviews code for raw color/font literals bypassing the project's design tokens, and mechanically replaces matched literals on approval |
| Error-Handling Review | — | Planned | Reviews/standardizes the `isError`/`errorMessage` + per-feature 401→logout ViewModel pattern |
| Accessibility Review | — | Planned | Audits/adds `accessibilityLabel`/`accessibilityIdentifier`/`accessibilityHint` coverage |
| Preview Mock Generator | — | Planned | Generates `*PreviewRepository` stubs with realistic fixture data for `#Preview` blocks |

---

## Planned Skills

Candidates only — not yet built. Originally captured from a brainstorming session
(2026-07-20) grounded in a survey of `hris-ios`; revised in a follow-up session
(2026-08-10) after a second pass over the same codebase. That second pass split
the original **Feature Scaffold** candidate in two: its navigation-wiring half
shipped standalone as `add-navigation-route` (above), and its DI-wiring half was
dropped — it would have duplicated the "not worth a skill" DI note below. The
error-handling half was never part of Feature Scaffold to begin with; it stays
its own candidate. Both **Accessibility Review** and **Design Token Usage** were
re-affirmed as worth building standalone rather than folding into
`swiftui-reviewer`/`swiftui-view-gen` — folding them in would bury them as one
bullet among many in a general review, the same problem the survey found with
accessibility today. **Preview Mock Generator** is new from the second pass.

**High signal** (repeated house-style patterns with no existing skill coverage):

1. **Error-Handling Review** — Every ViewModel hand-rolls an `isError`/`errorMessage` + `withAnimation { isError = true }` pattern (46+ files in `hris-ios`), paired with a per-feature `*ErrorMapper` that special-cases HTTP 401 → `isExpired` → manual token erase → logout, copy-pasted per feature with no shared abstraction or automatic token-refresh/retry. Strongest candidate — most duplicated, most bug-prone.
2. **Accessibility Review** — Zero uses of `accessibilityLabel`/`accessibilityIdentifier`/`accessibilityHint` anywhere in `HRIS/Feature` or `HRIS/Common`. A flat-out gap rather than an inconsistency; high value, low ambiguity.
3. **Preview Mock Generator** — 6+ hand-written `*PreviewRepository` structs (~100-170 lines each) must be manually kept in sync whenever their Repository protocol changes.

**Explicitly not worth a skill (yet):** DI-container abstraction (none exists — the current default-arg protocol init pattern is consistent enough as-is), feature flags/remote config (none in use), deep-linking (single hard-coded push-notification target, no generic router), or CI conventions beyond what `prepare-release`/`create-ipa` now cover.

**Suggested order:** Preview Mock Generator → Accessibility Review → Error-Handling Review.

---

## Installation (one-time per machine)

Register this repository as a marketplace in Claude Code once per developer machine. Run this inside any Claude Code session:

```
/plugin marketplace add https://github.com/ictech-projects/ios-claude-skills
```

This clones the marketplace to `~/.claude/plugins/marketplaces/ios-claude-skills/`. Skills are **not** copied into your project — they are referenced from this shared location.

---

## Enabling Skills in a Project

After installing the marketplace, enable the skills you want for a project by adding them to `.claude/settings.json` at the root of the repo:

```json
{
  "enabledPlugins": {
    "data-layer@ios-claude-skills": true,
    "security-review@ios-claude-skills": true,
    "swiftui-reviewer@ios-claude-skills": true,
    "swiftui-view-gen@ios-claude-skills": true,
    "unit-testing@ios-claude-skills": true,
    "localization-review@ios-claude-skills": true,
    "create-pull-request-against-development@ios-claude-skills": true,
    "prepare-release@ios-claude-skills": true,
    "create-ipa@ios-claude-skills": true,
    "add-navigation-route@ios-claude-skills": true,
    "design-token-usage@ios-claude-skills": true
  }
}
```

Commit this file so all team members get the same skills automatically (each developer still needs to run the one-time marketplace install on their own machine).

---

## Usage

Once installed and enabled, invoke any skill directly from the Claude Code prompt using its slash command:

```
/data-layer
/swiftui-view-gen
/unit-testing
/security-review
/swiftui-reviewer
/localization-review
/create-pull-request-against-development
/prepare-release
/create-ipa
/add-navigation-route
/design-token-usage
```

Each skill guides you through its workflow interactively — no extra configuration needed.

---

## Updating

When this repository is updated, pull the latest version on each developer machine:

```bash
git -C ~/.claude/plugins/marketplaces/ios-claude-skills pull
```

Then restart Claude Code for changes to take effect.

---

## Repository Structure

```
ios-claude-skills/
├── data-layer/
│   ├── SKILL.md          # Skill entry point and instructions
│   ├── references/       # Architecture, naming, and pattern references
│   └── skills/           # Sub-skill scripts
├── security-review/
│   ├── SKILL.md
│   └── references/
├── swiftui-reviewer/
│   ├── SKILL.md
│   └── references/
├── swiftui-view-gen/
│   ├── SKILL.md
│   └── references/       # Latest APIs, state management, layout, animations, etc.
├── unit-testing/
│   ├── SKILL.md
│   └── references/       # XCTest and Swift Testing rules, file structure, mocking
├── localization-review/
│   ├── SKILL.md
│   └── references/       # Detection rules, String Catalog background
├── create-pull-request-against-development/
│   ├── SKILL.md
│   └── references/       # Branch naming convention, MR template (Overview + Ticket + Checklist), commit ticket-tag format
├── prepare-release/
│   ├── SKILL.md
│   └── references/       # release/vX.Y.Z branch naming and version-verification procedure
├── create-ipa/
│   ├── SKILL.md
│   └── references/       # xcodebuild archive/export procedure (Debugging distribution, automatic signing)
├── add-navigation-route/
│   ├── SKILL.md
│   └── references/       # Route-enum/render-switch pattern, wiring checklist
└── design-token-usage/
    ├── SKILL.md
    └── references/       # Literal-detection rules, token-mapping strategy
```

---

## Adding to Your Project's README

Include the following section in your project's `README.md` so teammates know how to get set up:

~~~markdown
## Claude Code Skills

This project uses [Claude Code](https://claude.ai/code) with a set of iOS-specific skills sourced from the [`ios-claude-skills`](https://github.com/ictech-projects/ios-claude-skills) marketplace.

Skills are referenced via `.claude/settings.json` — not copied into the repo. Each developer must register the marketplace once on their machine.

### Available Skills

| Skill | Command | Description |
|---|---|---|
| Data Layer | `/data-layer` | Creates or reviews the Data layer for a domain (Model → TargetType → RemoteDataSource → Repository) |
| Security Review | `/security-review` | Comprehensive iOS security assessment against OWASP MASVS and Apple security guidelines |
| SwiftUI Reviewer | `/swiftui-reviewer` | Reviews SwiftUI code for best practices, modern APIs, and performance |
| SwiftUI View Gen | `/swiftui-view-gen` | Generates SwiftUI views, handles state management, animations, and Liquid Glass adoption |
| Unit Testing | `/unit-testing` | Writes or reviews unit tests for Repository and ViewModel layers (XCTest + Combine) |
| Create Pull Request | `/create-pull-request-against-development` | Opens a PR against the development branch, validating branch naming and the commit ticket-tag format, using ICT's MR template (Overview + Ticket + Checklist) |

### Setup (one-time per machine)

Run this once in Claude Code after cloning the repo:

```
/plugin marketplace add https://github.com/ictech-projects/ios-claude-skills
```

That's it. The skills defined in `.claude/settings.json` activate automatically.

### Updating Skills

When `ios-claude-skills` is updated, pull the latest marketplace on your machine:

```bash
git -C ~/.claude/plugins/marketplaces/ios-claude-skills pull
```

Then restart Claude Code.
~~~

---

## License

MIT
