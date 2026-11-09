# Detection Patterns

Use these to find leads in the changed files. They are ripgrep regexes; use the grep tool or `rg`. Every match must be confirmed by reading the code. Skip rows for state-management libraries the project doesn't use.

## Rebuild scope

| Rule | Pattern | How to confirm |
|---|---|---|
| R1 (bloc) | `BlocBuilder<\|BlocConsumer<\|BlocListener<\|BlocSelector<` | No `buildWhen:`/`listenWhen:` in the same constructor call, and the state has several fields or emits often |
| R1 (provider) | `context\.watch<\|Consumer<\|Provider\.of<` | Only part of the model is used → `context.select` / `Selector` |
| R1 (riverpod) | `ref\.watch\(\w+\)` | Only part of the state is used → `ref.watch(p.select(...))` |
| R1 (GetX/MobX) | `Obx\(\|Observer\(` | Wraps a large subtree but only a leaf reads the observable |
| R2 | `^class \w+` in entity/model/state files | Used in state, rebuild guards or `listEquals`, but has no `==` (no `Equatable`/`freezed`/record); or `props` misses fields |
| R3 | `MediaQuery\.of\(context\)` | Only one property is used → `sizeOf`/`paddingOf`/`viewInsetsOf`/… |
| R4 | `setState\(` | The `State` builds a large tree but the changed field affects only a leaf |
| R5 | (run `<flutter> analyze`) | `prefer_const_*` lint warnings in the changed files |
| R6, R24 | `builder: \((context\|_), _\)` · `context\.read<\w+>\(\)\.state` inside a builder | The consumer ignores its own state, or a child re-reads data the parent already has |
| R24 | `MultiBlocProvider\(\|MultiProvider\(` | Only one provider inside |

## Widget type

| Rule | Pattern | How to confirm |
|---|---|---|
| R7 | `extends StatefulWidget` | The `State` has no fields that are read, no controllers, no lifecycle use or mixins → stateless |
| R7, R10 | `extends StatelessWidget` + `.map(`/`.where(`/`.toList()`/`groupBy` in `build` | O(n) work on a frequently rebuilt widget → stateful with a cache |
| R8 | `Widget _build\w+\(` | Builds a large or independently changing subtree → extract a widget class |

## Cost of `build()`

| Rule | Pattern | How to confirm |
|---|---|---|
| R9 | `FutureBuilder\(\s*future:\s*\w+[\.\w]*\(` | The future is created inline in `build` |
| R9 | `\.map\(.*\)\.toList\(\)` · `context\.read<\w+>\(\)\.\w+\(` · `ref\.read\(` inside `build` | Computation or side effect on every build |
| R11 | `Theme\.of\(context\)\|MediaQuery\.\w+Of\(context\)` inside `for` / `.map(` | Could be done once, before the loop |
| R12 | `didChangeDependencies` | Fetches data or does one-time init without a guard |

## Work and paint

| Rule | Pattern | How to confirm |
|---|---|---|
| R13 | `children:\s*\[?\s*\.\.\.?\w+\.map\(` · `children:\s*\w+\.map\(` | The list can grow → `ListView.builder` / slivers |
| R14 | `jsonDecode\(\|\.map\(.*fromJson` on large payloads · DB `.all<` / `.query` + `.map(` | Heavy synchronous work on the UI isolate |
| R15 | `TabBarView\|PageView\|IndexedStack` | Child pages run animations, maps or tickers without an `isActive` guard |
| R16 | `itemBuilder:` in animated, timeline or image-heavy lists | Rows update independently and there's no `RepaintBoundary` |
| R17 | `\bOpacity\(` | Static value (not animated) wrapping a large subtree |
| R18 | `GlobalKey\(\)` · `GlobalKey<` inside `build` | Untyped key used for measuring, or a key created on every build |
| R19 | `for \(.*\) \{[^}]*if \(.*>=.*&&.*<` on sorted data | Linear scan where binary search would work |
| R20 | consecutive `await \w+\.(init\|initialize\|setup)\w*\(` in `main.dart` | Independent calls → `Future.wait`; non-critical ones → after the first frame |

## Memory

| Rule | Pattern | How to confirm |
|---|---|---|
| R21 | `TextEditingController\(\|ScrollController\(\|FocusNode\(\|AnimationController\(\|PageController\(\|TabController\(\|ValueNotifier<\|ChangeNotifier\(` | Created by the widget but no `dispose()` in `dispose()` |
| R21 | `StreamSubscription\|\.listen\(\|Timer(\.periodic)?\(` | No `cancel()` in `dispose()` |
| R21 | `= \w+(Cubit\|Bloc\|Store\|Notifier)\(` outside providers | Never `close()`d / `dispose()`d |
| R22 | `close\(\)` or `dispose()` of an owner before the dispose of a controller it holds | Wrong order |

## Hygiene

| Rule | Pattern | How to confirm |
|---|---|---|
| R23 | New or changed classes/widgets | Zero references (use a find-references/usages tool); imports or packages no longer used |
