# ios-claude-skills

A [Claude Code](https://claude.ai/code) skill marketplace for the ICT iOS team. Provides eight iOS-specific skills covering the full development lifecycle — data layer generation, SwiftUI views, unit testing, security review, localization, pull request creation, and release preparation.

---

## Skills

| Skill | Command | Description |
|---|---|---|
| Data Layer | `/data-layer` | Creates or reviews the Data layer for a domain (Model → TargetType → RemoteDataSource → Repository) |
| Security Review | `/security-review` | Comprehensive iOS security assessment against OWASP MASVS and Apple security guidelines |
| SwiftUI Reviewer | `/swiftui-reviewer` | Reviews SwiftUI code for best practices, modern APIs, and performance |
| SwiftUI View Gen | `/swiftui-view-gen` | Generates SwiftUI views from Figma designs, handles state management, animations, and Liquid Glass adoption |
| Unit Testing | `/unit-testing` | Writes or reviews unit tests for Repository and ViewModel layers (XCTest + Swift Testing) |
| Localization Review | `/localization-review` | Reviews code for String Catalog (`.xcstrings`) coverage, flags legacy localization APIs, and mechanically converts flagged strings on approval — never generates translations |
| Create Pull Request | `/create-pull-request-against-development` | Opens a PR against the development branch, validating branch naming and the commit ticket-tag format, using ICT's MR template (Overview + Ticket + Checklist) |
| Prepare Release | `/prepare-release` | Creates a `release/vX.Y.Z` branch from main, verifies the app version before branching, and optionally exports debug `.ipa` builds to `~/Downloads` |

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
    "prepare-release@ios-claude-skills": true
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
└── prepare-release/
    ├── SKILL.md
    └── references/       # Branch naming/version verification, xcodebuild archive/export procedure
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
