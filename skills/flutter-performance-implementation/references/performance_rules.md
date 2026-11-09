# Flutter Performance Rules

Bundled rule set. Each rule has a stable ID (`R1`…), which the review skill uses in its findings. If the project has its own performance guide, follow it where it conflicts with these rules.

Apply a state-management rule only if the project uses that library. Detect this from `pubspec.yaml`.

---

## A. Rebuild scope (highest impact)

**R1. Guard state-management rebuilds.** A consumer should rebuild only when the fields it renders change.

| Library | Use |
|---|---|
| flutter_bloc | `buildWhen` on `BlocBuilder`/`BlocConsumer`, `listenWhen` on `BlocListener`, or `context.select` |
| provider | `context.select<T, R>(...)` or `Selector<T, R>` instead of `context.watch<T>()` / `Consumer<T>` |
| riverpod | `ref.watch(provider.select((s) => s.field))` instead of `ref.watch(provider)` |
| GetX / MobX / signals | Keep `Obx`/`Observer`/`Watch` around the smallest widget that reads the value |

```dart
BlocBuilder<SettingsCubit, SettingsState>(
  buildWhen: (p, c) => p.isDark != c.isDark,
  builder: (_, s) => ThemeToggle(isDark: s.isDark),
)
```

**R2. Value equality on state and entities.** Classes used in state, in rebuild guards or in `listEquals` need `==`/`hashCode`. Use `Equatable`, `freezed`, records, or hand-written equality, and include **all** fields. Without it, R1 guards compare by reference and always see a change. Add a `const` constructor.

**R3. Narrow `MediaQuery` access.** Use `MediaQuery.sizeOf`, `paddingOf`, `viewInsetsOf`, `viewPaddingOf`, `orientationOf`, `devicePixelRatioOf` and `textScalerOf` instead of `MediaQuery.of(context).x`. Keep `MediaQuery.of` only when you need the whole `MediaQueryData`.

**R4. Keep `setState` small.** If only a leaf changes, use a `ValueNotifier` + `ValueListenableBuilder`, or move the state into a smaller widget.

**R5. `const` everywhere possible.** Enable the `prefer_const_constructors` and `prefer_const_literals_to_create_immutables` lints.

**R6. Pass data down.** If the parent already has the data, pass it to children as parameters instead of having each child read the same store. Remove consumers whose state is never used.

## B. Widget type

**R7. Choose the widget type by what it needs, not by assumed cost.**
- Use `StatelessWidget` by default.
- Use `StatefulWidget` when the widget caches derived data, owns controllers or notifiers, needs lifecycle hooks (`initState`/`didUpdateWidget`/`dispose`), or needs mixins (tickers, keep-alive).
- Convert a `StatefulWidget` to stateless when its `State` is never really used, for example a measure-then-`setState` whose result nothing reads.
- App/business state goes in the state-management layer. UI-only state (controllers, toggles, cached view data) goes in `State`.

**R8. Widget classes instead of `_buildX()` helpers.** A helper method always rebuilds with its parent. An extracted widget can be `const` and can have its own rebuild guard.

## C. Cost of `build()`

**R9. Keep `build()` cheap.** Don't allocate objects, do O(n) mapping, run I/O, or create futures inside it. Never write `FutureBuilder(future: load())` in `build`; create the future in `initState` and store it.

**R10. Cache derived data.** Compute it in `initState` and recompute in `didUpdateWidget` only when the input changes. Check `identical()` first, then fall back to a deep compare.

```dart
@override
void didUpdateWidget(OrdersView old) {
  super.didUpdateWidget(old);
  if (!identical(old.orders, widget.orders) && !listEquals(old.orders, widget.orders)) {
    _byDay = groupByDay(widget.orders);
  }
}
```

**R11. Move repeated work out of loops.** Look up `Theme.of`, styles and other values once, before the loop.

**R12. Choose the right lifecycle hook.**
- Fetch data in `initState` (or `addPostFrameCallback` if you need `context`), never in `didChangeDependencies` without a one-time guard.
- Skip reloading data that's already loaded when the user returns to the screen.

## D. Work and paint

**R13. Lazy lists.** For lists that can grow, use `ListView.builder`/`.separated` or slivers, not `Column(children: list.map(...))`.

**R14. Keep heavy work off the UI isolate.**
- For medium result sets, map in batches and `await Future.delayed(Duration.zero)` between them.
- For CPU-heavy work, use `Isolate.run`/`compute` on plain data. Database live objects (Realm, ObjectBox…) can't cross isolates.

**R15. Pause hidden pages.** Pages inside `TabBarView`/`PageView`/`IndexedStack` that run animations, maps or tickers should take an `isActive` flag and show a static version when hidden.

**R16. `RepaintBoundary` where it helps.** Use it on subtrees that update independently (animated rows, timelines, clocks). Confirm with Repaint Rainbow; don't add it everywhere.

**R17. No static `Opacity` on large subtrees.** Use `color.withValues(alpha:)`, `ColorFiltered`, or `FadeTransition` for animated fades.

**R18. No `GlobalKey` for layout measurement.** Use `LayoutBuilder` or `context.findRenderObject()`. Never create keys inside `build`. Typed `GlobalKey<FormState|ScaffoldState|NavigatorState>` (and form-builder state keys) are fine.

**R19. Binary search** for lookups in sorted data (scroll offsets, dates, ranges).

**R20. Fast startup.**
- Run independent init with `Future.wait`.
- Move non-critical init (remote config, analytics, loggers) after the first frame.
- Create services lazily on first use where the DI/service-locator approach allows it.

## E. Memory

**R21. Dispose what you create.**
- `dispose()`: every `TextEditingController`, `ScrollController`, `FocusNode`, `AnimationController`, `ValueNotifier`/`ChangeNotifier` the widget creates.
- `cancel()`: every `StreamSubscription` and `Timer`.
- `close()`: every Bloc/Cubit or store created by hand (not through its provider).

**R22. Dispose order.** Dispose a media or platform controller **before** closing the object that holds it.

## F. Hygiene

**R23. Remove dead code.** Delete widgets/classes with no references, unused imports, and unused packages, along with their platform setup. `dart_dependency_validator` or a similar tool can find unused packages.

**R24. Remove redundant wrappers.** Collapse multi-provider wrappers that contain a single provider, and remove builders whose state isn't used.
