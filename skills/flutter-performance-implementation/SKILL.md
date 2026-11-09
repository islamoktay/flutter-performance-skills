---
name: flutter-performance-implementation
description: 'Write or refactor Flutter/Dart code following performance best practices, in any Flutter project. Use when: implementing a new screen/widget, refactoring for performance, fixing performance audit items, reducing rebuilds, adding buildWhen/select/Selector, value equality (Equatable/freezed), MediaQuery.sizeOf, ValueNotifier, RepaintBoundary, lazy lists, caching derived data, choosing StatelessWidget vs StatefulWidget, disposing controllers.'
argument-hint: 'What to implement or refactor (feature, file, widget, or audit item)'
---

# Flutter Performance — Implementation Phase

Write or refactor code using the bundled performance rules, adapted to the conventions of whichever project you're in. Then verify the change and record it.

Rules: [performance_rules.md](./references/performance_rules.md) (IDs `R1`…`R24`).

## Procedure

### 1. Detect the project context

Run this once per session and reuse the results.

| What | How to detect | Use |
|---|---|---|
| Flutter command | `.fvmrc` or `.fvm/` exists → `fvm flutter` / `fvm dart`; otherwise `flutter` / `dart` | All commands below |
| State management | Check `pubspec.yaml` for `flutter_bloc`, `provider`, `flutter_riverpod`/`hooks_riverpod`, `get`, `mobx`, `signals` | Choose the matching R1 variant |
| Equality | Check `pubspec.yaml` for `equatable`, `freezed` | How to apply R2 |
| Code generation | `build_runner` in `dev_dependencies` | Regenerate after changing annotated or mocked classes |
| Project rules | Agent instruction files (`AGENTS.md`, `.github/copilot-instructions.md`, `CLAUDE.md`, `*.instructions.md`), architecture docs (`docs/**/architecture*`), a performance guide (`docs/**/*performance*`) | Read them. **Project rules win over the bundled rules** when they conflict. Follow their naming, layering, localization and command conventions (e.g. a required command prefix). |
| Tracker | A performance audit/backlog file (e.g. `docs/**/*audit*.md`, `*performance*.md` with checkboxes) | Update it in step 7 if it exists |
| Tests | `test/` layout that mirrors `lib/` | Choose the related tests in step 6 |

### 2. Clarify

Ask the user short Y/N questions about scope and any gaps: which files or features, whether shared packages may change, and whether to update the tracker.

### 3. Read before you edit

Read the relevant rules, the project docs from step 1, and every file you'll change.

### 4. Confirm the problem exists

Findings from audits or older reviews are often stale. If the issue is already fixed or is a false positive, don't change the code. Record that in step 7 instead.

### 5. Implement

- Apply the rules that fit, starting with the highest-impact group (A → F in the rules file).
- Keep each change small and focused, and keep existing behavior.
- Remove anything the change leaves unused: imports, fields, keys, helpers, widgets, packages (R23).
- Follow the project's architecture and conventions over your own preferences.

### 6. Verify

Never build or run the app unless the user asks.
- `<flutter> analyze` must report no new issues.
- Run the tests related to the changed files, e.g. `<flutter> test test/<mirrored path>/`.
- If you changed generated or mocked classes: `<dart> run build_runner build --delete-conflicting-outputs`, then re-run the tests.
- If a command is missing or fails for environment reasons, tell the user. Don't work around it.

### 7. Record

If the project has a tracker, update it:
- `[x]` for fixed items, with a one-line note about what changed.
- `[-]` for items you're not fixing (stale or false positive), with the reason.
- New rows for new findings.

Don't create a tracker unless the user asks for one.

### 8. Report

List the changes with file links and the rule IDs applied, then the analyze and test results and any follow-ups.

## Quick reference (full details in the rules file)

- **Rebuilds:** R1 rebuild guards, R2 value equality, R3 `MediaQuery.xOf`, R4 small `setState`/`ValueNotifier`, R5 `const`, R6 pass data down.
- **Widget type:** R7 stateless by default, stateful to cache data, own controllers or use lifecycle; R8 widget classes instead of `_buildX()`.
- **`build()`:** R9 cheap and pure, R10 cache derived data, R11 move work out of loops, R12 fetch in `initState`, not `didChangeDependencies`.
- **Work and paint:** R13 lazy lists, R14 isolates/batching, R15 pause hidden tabs, R16 `RepaintBoundary`, R17 no static `Opacity`, R18 no measuring `GlobalKey`, R19 binary search, R20 startup.
- **Memory:** R21 dispose/cancel/close, R22 dispose order.
- **Hygiene:** R23 dead code, R24 redundant wrappers.
