# Flutter Performance Skills

Two reusable **agent skills** that teach AI coding assistants (Claude Code, GitHub Copilot, and others that support the skills format) to write and review Flutter code following proven performance rules.

They were born out of a full performance pass on a large production Flutter app. The background and all the rules are explained in the companion article: [Making a Large Flutter App Fast: The Changes That Actually Mattered](#) <!-- TODO: replace with the Medium link once published -->

## What's inside

```text
skills/
├── flutter-performance-implementation/   # use when writing or refactoring code
└── flutter-performance-review/           # use when reviewing committed code
```

Each skill is a folder with a `SKILL.md` file containing a name, a description with trigger words, and step-by-step instructions. Your assistant can load it by name or pick it up automatically when a request matches the description.

## Installation

Copy both folders into your repository's skills directory. For Claude Code:

```bash
mkdir -p .claude/skills
cp -r skills/flutter-performance-implementation .claude/skills/
cp -r skills/flutter-performance-review .claude/skills/
```

For other assistants, place the folders wherever that tool discovers skills or reusable instructions.

No configuration needed — the skills detect the project setup themselves.

## The implementation skill

Use it when you write new code or refactor existing code. It tells the assistant to:

1. Detect the project setup, and ask about anything unclear in the request.
2. Read the relevant rules, the project's own docs, and the files it will change.
3. Check that the problem still exists, since audit findings go stale.
4. Make a small, focused change that follows the project's conventions, and remove whatever it leaves unused.
5. Run `flutter analyze` and the related tests. It never builds or runs the app unless asked.
6. Update the project's performance tracker if one exists, and summarize what changed with the rule IDs.

**Example prompts:**

```text
/flutter-performance-implementation add buildWhen to all BlocBuilders in the documents feature
/flutter-performance-implementation refactor OrdersView — it regroups orders on every rebuild
```

## The review skill

Use it on code that's already committed. It defaults to the last commit, and also accepts a commit, a range, or a branch diff. It tells the assistant to:

1. Collect the changed Dart files, skipping generated code.
2. Scan them with a table of search patterns, one per rule, using only the rows for the project's state-management library.
3. Read the code around each match, and drop the legitimate cases.
4. Report a severity table: file and line, issue, rule ID, suggested fix.
5. **Ask before changing anything.** The options are fix all, fix selected, only log the findings, or do nothing.

**Example prompts:**

```text
/flutter-performance-review
/flutter-performance-review develop..HEAD
```

## How the skills adapt to your project

- **Bundled rules:** each skill carries its own copy of the performance rules, each with an ID (R1–R24), covering rebuild scope, value equality, `MediaQuery` subscriptions, lifecycle mistakes, isolates, lazy lists, paint cost, and memory leaks.
- **Project detection:** they detect how the project is set up instead of assuming — whether it uses FVM, which state-management library it uses (Bloc, Provider, Riverpod…), whether it has `equatable` or `freezed`, and whether it has code generation.
- **Project rules take priority:** if the project has its own instruction files, architecture docs, or performance guide, the skills read them and follow them where they conflict with the bundled rules.

## Why skills instead of a style guide

- The rules are applied automatically while the assistant writes code, not only when someone remembers to check.
- Reviews look at the diff, not the whole codebase, so findings stay relevant.
- Searches only find candidates — the skill reads the surrounding code before reporting, which filters out most false positives.
- Nothing changes without approval: the review only reports until a person decides what to fix.

## License

Free to use and adapt in any project.
