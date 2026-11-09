---
name: flutter-performance-review
description: 'Review committed Flutter/Dart code for performance problems, in any Flutter project. Use when: reviewing a commit, commit range, PR or branch diff for performance; checking for missing rebuild guards (buildWhen/select/Selector), missing value equality, MediaQuery.of, wide setState, Opacity, GlobalKey misuse, undisposed controllers, eager lists, expensive build methods, wrong StatelessWidget/StatefulWidget choice. Reports findings first and asks before fixing anything.'
argument-hint: 'Commit, range or branch (e.g. HEAD, abc123, main..HEAD). Defaults to the last commit.'
---

# Flutter Performance — Review Phase

Review committed code for performance problems. **Report first and change nothing**, then ask the user what to do.

- Rules: [performance_rules.md](../flutter-performance-implementation/references/performance_rules.md) (IDs `R1`…`R24`). This file lives in the implementation skill, so copy both skill folders together.
- Search patterns: [detection_patterns.md](./references/detection_patterns.md)

## Procedure

### 1. Detect the project context

Same as step 1 of the `flutter-performance-implementation` skill:
- Flutter command (`fvm` or not)
- State-management library
- Equality approach (`equatable`/`freezed`)
- Project rules and docs (agent instruction files, architecture and performance docs). **Project rules win** over the bundled rules.
- Performance tracker, if one exists

### 2. Resolve the target

- No argument: the last commit (`HEAD~1..HEAD`).
- A single SHA `X`: `X~1..X`.
- A range or `base..head`: use it as given.
- If the base branch is unclear, ask the user.

### 3. Collect the changed files

Skip generated code:

```bash
git --no-pager diff --name-only <range> -- '*.dart' ':!*.g.dart' ':!*.freezed.dart' ':!*.mocks.dart' ':!*.gr.dart' ':!*.config.dart'
```

Also skip files the project marks as generated.

### 4. Read the diff and its context

Read the diff for each file, plus the current code around the changes. You can't judge rebuild scope or disposal from diff lines alone.

### 5. Scan for leads

Use [detection_patterns.md](./references/detection_patterns.md), and only the rows that apply to the detected state-management library. A match is a lead, not a finding.

### 6. Verify each lead

Read the code and drop false positives. Each pattern row has a "How to confirm" column. Common exceptions:
- **Rebuild guard:** a missing one is fine if the consumer really needs every emission.
- **`GlobalKey`:** typed form/scaffold/navigator keys are fine.
- **`MediaQuery.of`:** fine when the whole `MediaQueryData` is needed.
- **`StatefulWidget`:** fine when it caches data, owns controllers, or uses lifecycle hooks or mixins.
- **Lists:** a `Column` with `.map()` is fine for small, fixed-size lists.
- **Project rules:** a pattern the project's rules explicitly allow is not a finding.

### 7. Keep to scope

Report only problems in the changed lines or caused by them. List older problems in a separate "pre-existing" section. If the project has a tracker, note when a finding is already listed there.

### 8. Report

Use this format, sorted by severity and then by file:

| # | Severity | File:line | Issue | Rule | Suggested fix |
|---|---|---|---|---|---|
| 1 | 🔴 High | [orders_view.dart](path#L42) | `BlocBuilder` rebuilds on every emission | R1 | `buildWhen: (p, c) => p.orders != c.orders` |

Severity:
- 🔴 **High:** jank or rebuilds on a frequently used screen, memory leaks, work on every frame or keystroke.
- 🟡 **Medium:** extra rebuilds or work on normal screens.
- 🟢 **Low:** `const`, style issues, dead code.

Then list pre-existing problems and anything you weren't sure about. If you found nothing, say so and list what you checked.

### 9. Ask before acting

Use the ask-user tool if available; otherwise give a numbered list. Options:
- Fix all findings
- Fix selected findings (by number)
- Add the findings to the project's tracker only (if it has one)
- Do nothing

Don't edit, stage or commit anything until the user answers.

### 10. Apply approved fixes

Follow the `flutter-performance-implementation` skill for the selected items: implement, analyze, test, and update the tracker. Never build or run the app unless the user asks.
