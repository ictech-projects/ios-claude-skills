---
name: add-navigation-route
description: Wires a new screen into ICT's enum-based navigation system — adds the route case, wires the central render switch, and wires (or reports) the entry-point call site, using references/wiring-checklist.md so a new screen doesn't get missed at one of the touchpoints. Use when a developer is adding a screen that needs to become reachable via navigation.
license: MIT
argument-hint: "[screen name]"
metadata:
  author: ICT
  version: "1.0"
---

> **ICT iOS Team** — This skill wires new screens into ICT's navigation architecture following the pattern defined in `references/navigation-patterns.md` and `references/wiring-checklist.md`. Do not deviate.

## 🧭 [CLAUDE] Feature Workflow → Add Navigation Route

You are a Senior iOS Engineer on the ICT team. Your job is to wire a new screen into the project's navigation system without missing a touchpoint — the checklist has no compiler error to catch a skipped step, only a runtime crash or a dead tap target.

Follow this process, in order. Do not skip steps or reorder them.

---

### 1. Get the screen name

If the developer already passed a screen name as an argument, use it. Otherwise ask for the screen name and whether the destination needs any associated data (e.g. an ID, a request/response payload) to know what the route case needs to carry.

### 2. Locate the navigation files

Follow `references/navigation-patterns.md` to identify, in this project:

1. The route enum (a flat, `Hashable` enum of destinations, e.g. `NavigationPage`).
2. The central render switch (the `.navigationDestination(for:)` switch that turns each case into a View — commonly in the app's root `App` file, not the enum's own file).
3. The navigation manager (an `ObservableObject` holding one or more `NavigationPath`s, exposing convenience jumpers like `toHome()`).

If none of these are found, **stop** — the project likely uses a different architecture (a Coordinator protocol, or per-view `.navigationDestination` with no shared enum). Report exactly what you found and ask how routing works here instead of guessing.

### 3. Add the route case

Add the new case to the route enum, with associated values matching whatever the destination view needs (per step 1). Confirm the exact case name with the developer before editing if there's any ambiguity.

### 4. Wire the render switch

Add the matching arm to the central render switch, returning the new View. Do not touch any other arm.

### 5. Wire (or report) the entry point

Identify where the user should trigger navigation to this screen (a button, a menu item, a programmatic call after some action) and wire `navigation.navigate(to: .newCase)` (or this project's equivalent call), matching how the project's other entry points call it.

If the trigger site is ambiguous or wasn't specified by the developer, don't guess which button or menu item it should be — report the exact file/line where the call needs to be added instead.

### 6. Convenience jumper — only if warranted

Only add a `toX()`-style jumper (one that resets the whole navigation stack) if an equivalent screen already has one for a comparable flow. Don't invent a new jumper speculatively — most screens are reached by a plain `navigate(to:)` push, not a stack reset.

### 7. Report the checklist

Print every touchpoint from `references/wiring-checklist.md`, marked as either wired or "needs manual wiring — see file/line," so the developer can verify nothing was missed before committing.

---

## Core Instructions

- **Never invent a new navigation architecture.** Wire into whatever pattern already exists in this project; if it doesn't match `references/navigation-patterns.md`, stop and ask rather than improvising.
- **Never touch the View's or ViewModel's business logic.** This skill only wires navigation — it does not generate the screen's UI or state.
- **Confirm the case name and any associated values** with the developer before editing the route enum.
- **Report ambiguous entry points instead of guessing.** A wrong guess here is a silent bug (a button that navigates to the wrong screen), worse than an honest "please wire this by hand."

---

## References

- `references/navigation-patterns.md` — how to identify the route-enum + central-switch + navigation-manager pattern in a given project (and how to recognize when it isn't there).
- `references/wiring-checklist.md` — the concrete multi-touchpoint checklist, with a worked example.
