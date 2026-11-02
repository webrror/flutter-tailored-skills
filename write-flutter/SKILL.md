---
name: write-flutter
description: Write idiomatic, modern Dart and Flutter code with strong API design, lifecycle safety, maintainability, accessibility, and performance. Use for Flutter/Dart implementation tasks and code generation.
---

# Write Flutter

## Purpose

Write high-quality, idiomatic Dart and Flutter code. Prefer the simplest design that clearly expresses intent, follows current language/framework conventions, and remains easy to maintain.

This skill adapts the language-craft intent of `write-swift` to Dart and Flutter. It is not a syntax translation. Apply Dart language guidance to Dart code and Flutter framework guidance to widgets, rendering, navigation, state, and platform integration.

## Version and source-of-truth policy

- First inspect `pubspec.yaml`, `pubspec.lock`, `analysis_options.yaml`, existing code patterns, and the project's Flutter/Dart SDK constraints.
- The project's declared SDK constraints and installed toolchain take precedence over assumptions about the latest release.
- For version-sensitive APIs, verify current official Dart/Flutter documentation and release notes. Prefer stable APIs unless the project explicitly targets prerelease features.
- Do not hardcode a patch version into generated project code or skill logic. Re-check the latest stable versions when starting a new project or when the user asks.
- Use package APIs only after confirming the package is present and its installed version supports the API.
- Do not introduce a dependency for a small, straightforward task without a clear benefit.

## Core principles

1. **Clarity over cleverness.** Prefer readable, direct code over dense expressions, clever type tricks, or unnecessary abstractions.
2. **Use the language.** Lean on null safety, inference, named parameters, enums, records, patterns, sealed classes, extensions, and generics when they improve clarity and are supported by the SDK.
3. **Use Flutter declaratively.** Describe UI from state; avoid imperative widget-tree manipulation or unnecessary manual synchronization.
4. **Keep state ownership clear.** Each piece of state should have an obvious owner and lifecycle. Avoid duplicated sources of truth.
5. **Make invalid states hard to represent.** Use types and constructors to encode invariants where this simplifies the design.
6. **Optimize only with evidence.** First choose a sound design; profile before introducing caches, isolates, custom render objects, or complexity.
7. **Respect platform conventions.** Use Material, Cupertino, accessibility, navigation, and input behavior appropriately for the target.
8. **Follow the surrounding project.** Match established naming, formatting, architecture, state management, and testing patterns unless they are clearly harmful.
9. **Prefer small, composable units.** Extract widgets/functions when they clarify responsibility or reduce rebuild scope—not merely to make files shorter.
10. **Avoid speculative generality.** Do not build frameworks, plugin abstractions, or configuration systems for one use case.

## Dart language craft

### Types and API design

- Prefer sound null safety. Avoid `!` unless the invariant is established and documented or obvious locally.
- Use `final` by default for local variables and fields that do not need reassignment; use `const` when compile-time constants are possible and useful.
- Prefer named parameters for APIs with multiple arguments or arguments whose meaning is not obvious.
- Use positional parameters for small, conventional, tightly related values.
- Choose types that communicate intent. Avoid `dynamic` and `Object?` when a meaningful type is available.
- Use `sealed` classes for closed hierarchies when exhaustive handling is useful and the SDK supports them.
- Use enums for finite, simple states; use sealed types when states carry different data or behavior.
- Use records for lightweight grouped values when named classes would add unnecessary ceremony.
- Use extension methods/types only when they improve discoverability or model a coherent operation. Avoid extensions that obscure ownership or create surprising behavior.
- Keep constructors and factories predictable. Validate invariants at clear boundaries.
- Avoid public APIs that expose implementation details or mutable collections unnecessarily.
- Prefer immutable data models when practical; do not add immutability machinery without a reason.

### Async, streams, and concurrency

- Use `async`/`await` for readable asynchronous control flow; return a `Future` directly when no transformation or error handling is needed.
- Use `Stream` for multiple events over time, not as a substitute for a one-shot result.
- Handle errors at the boundary where the application can respond meaningfully. Do not silently swallow exceptions.
- Avoid unawaited futures unless intentional; document or explicitly handle fire-and-forget work.
- Guard against stale async results after a widget/state owner is disposed or the request is superseded.
- Use isolates (`Isolate`, `compute` where supported) for CPU-bound work that demonstrably blocks the UI isolate—not ordinary network I/O.
- Avoid sharing mutable state across async boundaries when a simple immutable message/result is sufficient.
- Consider cancellation, timeouts, retries, and duplicate requests where the user experience or resource usage requires them.
- Keep async methods small enough that loading, success, and failure states remain easy to understand.

### Collections, iteration, and errors

- Prefer expressive collection literals, spreads, collection-if/for, and `where`/`map` when they remain readable.
- Avoid long chained transformations when a named intermediate makes the logic clearer.
- Preserve ordering and duplicate semantics intentionally; do not replace a list with a set unless behavior permits it.
- Use exceptions for exceptional failures, not routine branching.
- Use typed result/state models when expected failure is part of normal application flow.
- Avoid catch-all catches that erase diagnostic information. Log or propagate with context as appropriate.

### Formatting and naming

- Follow `dart format` and the project's lints.
- Use `UpperCamelCase` for types, `lowerCamelCase` for members and variables, and `lowercase_with_underscores` for library/file names.
- Name booleans as predicates (`isLoading`, `hasError`, `canSubmit`) and callbacks as actions (`onPressed`, `onChanged`).
- Avoid abbreviations unless they are standard in the domain.
- Keep comments focused on why, invariants, or non-obvious constraints—not narrating obvious code.
- Use documentation comments for public APIs when the project expects them.

## Flutter framework craft

### Widget composition

- Prefer small, composable widgets with clear inputs over large build methods with intertwined conditions.
- Keep `build` methods declarative and side-effect free.
- Do not perform network calls, mutate state, start timers, or subscribe to streams from `build`.
- Use `const` constructors where possible, but do not contort code just to add `const`.
- Use `Builder` or a dedicated widget only when a new context or lifecycle boundary is actually needed.
- Use keys when identity matters (reordering, preserving state, differentiating siblings); do not add keys indiscriminately.
- Prefer layout widgets and constraints over hardcoded screen dimensions. Handle safe areas, text scaling, localization, and orientation.
- Avoid deeply nested widget expressions when extracting a named widget improves comprehension.
- Use `LayoutBuilder`/`MediaQuery` when responsive behavior genuinely depends on constraints or environment.

### State and architecture

- Keep ephemeral UI state close to the widget that owns it.
- Lift state only when multiple parts of the UI need to coordinate or a longer-lived owner is required.
- Follow the project's state management choice (Provider, Riverpod, Bloc, etc.); do not migrate libraries as part of an unrelated task.
- Avoid duplicating derived state. Compute it from the source of truth unless caching is justified.
- Keep business logic out of presentation widgets when it becomes nontrivial; use focused models, services, controllers, or view models consistent with the codebase.
- Avoid introducing repository/use-case layers for trivial features. Architecture should scale with actual complexity.
- Make loading, empty, error, and success states explicit when relevant.
- Keep navigation decisions and side effects out of deeply nested UI callbacks when a coordinator/controller is already used.

### Lifecycle and resources

- Dispose controllers, focus nodes, animation controllers, subscriptions, and other owned resources.
- Cancel timers and subscriptions when no longer needed.
- Use `mounted`/`context.mounted` only where async work can outlive the relevant state/context; avoid redundant checks.
- Handle `didUpdateWidget` when a stateful widget's configuration changes in a way that affects owned resources or subscriptions.
- Avoid storing `BuildContext` beyond its valid lifetime.
- Ensure listeners are removed and callbacks cannot mutate disposed state.
- Respect app lifecycle and route visibility for ongoing work, animations, and media.
- Use `TickerProviderStateMixin` only when multiple tickers are needed; otherwise prefer `SingleTickerProviderStateMixin`.

### Rendering and performance

- Avoid unnecessary rebuilds, but do not prematurely optimize.
- Narrow rebuild scopes when a high-frequency state update affects a large subtree.
- Use `AnimatedBuilder`, `ListenableBuilder`, selectors, or state-management-specific granular subscriptions when appropriate.
- Avoid expensive synchronous work in `build` or on the UI isolate.
- Use lazy builders for large/long lists (`ListView.builder`, slivers).
- Use `RepaintBoundary` only when profiling or repaint behavior supports it.
- Avoid unnecessary opacity, clipping, shadows, blur, and large offscreen layers.
- Prefer standard widgets before custom render objects. Custom painting/rendering requires a clear need and careful semantics/ hit-testing.
- Profile in profile mode with DevTools before claiming a performance issue or adding complexity.
- Consider image sizing, caching, decoding, and disposal in image-heavy interfaces.

### Animation and interaction

- Prefer implicit animations for simple state-driven transitions; use explicit controllers for sequencing, gestures, or fine control.
- Make animations interruptible and lifecycle-safe.
- Respect reduced-motion preferences where relevant and preserve state communication.
- Use gestures that work with touch, mouse, keyboard, and accessibility semantics as appropriate.
- Do not make hover the only feedback path.
- Keep motion short and purposeful; consult the project's animation standards/skill when one is available.
- Avoid custom gesture recognizers unless built-in widgets cannot meet the interaction requirements.

### Accessibility and localization

- Use semantic labels and correct control roles.
- Ensure controls have adequate hit targets and keyboard/focus behavior.
- Do not convey state by color or motion alone.
- Support text scaling and avoid fixed-height text containers that clip.
- Use localization resources for user-facing strings; avoid hardcoded text in reusable components when the project is localized.
- Respect right-to-left layouts and locale-sensitive formatting.
- Provide accessible alternatives for custom-painted or complex interactive content.

## Packages and platform integration

- Inspect installed dependencies before selecting a package.
- Prefer official Flutter/Dart packages or well-maintained packages with clear compatibility, but evaluate fit rather than popularity alone.
- Check maintenance status, supported platforms, SDK constraints, issue activity, license, and transitive dependency cost when adding a dependency.
- Keep platform-specific code behind a clear boundary; use existing plugins or platform channels consistently with the project.
- For FFI/platform channels, handle errors, lifecycle, threading, and data ownership explicitly.
- Do not introduce code generation unless it materially improves correctness or maintainability.
- Verify generated files are regenerated with the project's established commands and are not manually edited unless the project explicitly does so.

## Testing and verification

For nontrivial changes, choose tests at the appropriate level:

- Unit tests for pure logic, transformations, validation, and model behavior.
- Widget tests for rendering states, interaction, semantics, and layout behavior.
- Integration tests for important end-to-end flows and platform interactions.
- Golden tests only when visual stability and maintenance cost are justified.

Before finalizing, when feasible:

1. Run `dart format` on changed Dart files.
2. Run `flutter analyze` (or the project's equivalent).
3. Run focused tests, then broader tests when appropriate.
4. Verify relevant flows on target platforms/devices for platform-specific or visual behavior.
5. Report commands actually run and their outcomes. Never claim tests passed if they were not run.

## Implementation workflow

1. Clarify the requested behavior and identify constraints.
2. Inspect relevant files, SDK constraints, dependencies, and local conventions.
3. Choose the smallest design that meets the requirements.
4. Implement with idiomatic Dart and declarative Flutter.
5. Handle loading/error/empty states, lifecycle, accessibility, and edge cases proportionate to the task.
6. Add or update focused tests for meaningful behavior.
7. Format, analyze, and test when tools are available.
8. Summarize changed files, key decisions, and verification status.

## Avoid

- Overengineering or introducing abstractions before they earn their cost.
- Migrating state management, architecture, or dependencies without request.
- Using `dynamic`, `!`, broad catches, or global mutable state to bypass type design.
- Side effects in `build`.
- Unowned controllers, listeners, timers, or subscriptions.
- Premature micro-optimization or unsupported performance claims.
- Hardcoded device dimensions or inaccessible custom controls.
- Rewriting unrelated code just to impose personal style.
- Treating the latest SDK as automatically available in every project.
- Generating code that depends on unverified package APIs.

## Output expectations

When implementing code:
- Provide the code or make the requested edits directly.
- Explain important decisions briefly.
- State assumptions only when they affect behavior.
- Report verification accurately, including anything not run.
- Prefer complete, usable implementation over disconnected snippets when the task calls for a feature.
