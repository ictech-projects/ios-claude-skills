# 🌿 [CLAUDE] Branch Naming Convention (feature/module-name/HA-100)

The current branch (the one the PR is opened **from**) must match:

```
<type>/<module-name>/<TICKET>
```

- `<type>` — `feature` for new functionality, `bugfix` for fixes. This should agree with the `<TAG>` used in the first commit (see `references/mr-template.md`): `feature` ↔ `FEATURE`, `bugfix` ↔ `FIX`.
- `<module-name>` — lowercase, kebab-case identifier for the area of the app being touched (e.g. `login`, `checkout`, `skills`).
- `<TICKET>` — either a ticket key like `HA-100`, `DEVT-664` (project prefix + dash + number), or the literal `NO-BTS` when there is no backlog ticket. Same rule as the commit format.

**Regex:** `^(feature|bugfix)/[a-z0-9]+(-[a-z0-9]+)*/(NO-BTS|[A-Z][A-Z0-9]*-\d+)$`

**Valid examples:**

```
feature/module-name/HA-100
bugfix/module-name/HA-101
feature/login/NO-BTS
bugfix/checkout/DEVT-42
```

**Invalid examples:**

```
feature/HA-100                     (missing module-name)
feature/module-name/create-thing   (ticket segment isn't NO-BTS or a ticket key)
HA-100-fix-login                   (no type/module-name structure at all)
```

If the current branch doesn't match, it must be corrected (see the rename procedure in the parent `SKILL.md`) before the PR is opened, since the branch name is part of ICT's MR SOP alongside the commit format.
