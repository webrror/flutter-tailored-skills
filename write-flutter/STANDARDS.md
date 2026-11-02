# Dart & Flutter Craft Standards

Use this reference for detailed decisions when applying `write-flutter`. Confirm current SDK/package support before relying on version-sensitive APIs.

## Language feature selection

| Feature | Use when | Avoid when |
|---|---|---|
| `final` / `const` | Values should not be reassigned / compile-time constants | It makes code awkward without benefit |
| Records | Lightweight grouped values with clear shape | Values need behavior, invariants, or stable public identity |
| Sealed classes | Closed state/result hierarchies and exhaustive switches | Open-ended extension is expected |
| Patterns | Destructuring or branching becomes clearer | Pattern syntax obscures simple logic |
| Extensions | A coherent operation belongs naturally on an existing type | Adds surprising behavior or collides with APIs |
| Extension types | Stronger static distinction with low runtime overhead, when supported | Adds complexity or compatibility concerns |
| Generics | Reusable behavior preserves useful type information | Only one concrete use exists and abstraction obscures intent |
| Code generation | Repetitive, error-prone models or APIs benefit materially | A small hand-written model is simpler |

Use features only if the project's Dart SDK supports them. Prefer stable language features for production code.

## Async and concurrency

- Network and file I/O are asynchronous; do not move them to isolates by default.
- CPU-heavy parsing, image processing, or computation may need an isolate after measurement.
- Avoid unawaited work without an explicit reason and error strategy.
- Prevent stale responses from overwriting newer state.
- Model loading, success, empty, and failure states explicitly where useful.
- Use streams for ongoing event sequences and manage subscription cancellation.
- Prefer structured ownership and cancellation patterns over detached background work.

## Flutter state ownership

- Local animation, focus, selection, and transient form state usually belong near the widget.
- Shared application state belongs in the project's chosen state-management layer.
- Derived state should be computed from source state unless profiling justifies caching.
- Keep side effects at event, controller, or service boundaries—not inside `build`.
- Do not introduce a state management package solely for a small isolated feature.

## Widget and layout decisions

- Use constraints, flex, wrapping, and slivers rather than fixed screen-size assumptions.
- Use `ListView.builder`/slivers for large or unbounded collections.
- Extract widgets when it improves naming, ownership, testability, or rebuild isolation.
- Avoid extracting trivial one-line widgets that make navigation harder.
- Use keys to preserve or distinguish identity when stateful children move or are replaced.
- Ensure responsive layouts account for keyboard insets, safe areas, text scaling, and orientation.

## Lifecycle checklist

- Controller/node/subscription created? Identify owner.
- Owner disposed? Release resource.
- Configuration changed? Update/rebind as needed.
- Async callback after dispose? Guard or cancel.
- Listener added? Remove it.
- Route/app backgrounded? Pause or stop work if appropriate.
- Context retained across await? Verify it remains valid before use.

## Performance evidence

- Profile in profile mode on representative devices.
- Inspect UI and raster frame times separately.
- Use rebuild/repaint diagnostics to locate actual work.
- Keep animated rebuild scopes narrow.
- Avoid blanket `RepaintBoundary` use.
- Prefer simple standard widgets over custom render objects unless requirements demand them.
- Treat performance claims as hypotheses until measured.

## Accessibility and interaction

- Use semantics for custom controls and painted UI.
- Ensure keyboard/focus operation where relevant.
- Respect reduced motion.
- Support text scaling and localization.
- Avoid hover-only affordances.
- Keep touch targets usable and feedback immediate.
- Ensure transitions do not leave invisible interactive elements active.

## Dependency evaluation

Before adding a package, check:
- Does it solve a real need better than SDK APIs?
- Is it compatible with the project's SDK and target platforms?
- Is it maintained and documented?
- What transitive dependencies and binary size does it add?
- Does it require code generation, native setup, or special lifecycle handling?
- Is its API stable enough for the project's release cadence?

## Verification baseline

Run the project's expected commands when possible:

```bash
dart format .
flutter analyze
flutter test
```

Prefer targeted paths for large repositories, then expand as appropriate. Do not report commands as run unless they were actually executed. Platform-specific behavior requires device/emulator verification when feasible.
