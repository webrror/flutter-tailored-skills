---
name: animate-flutter-foundations
description: Apply the foundations of purposeful animation in Flutter. Decide whether motion is needed, identify its purpose, choose the simplest Flutter animation primitive, select properties and timing, handle interruption and exit, and ship accessible, performant motion. Use when building or modifying Flutter animations, transitions, gestures, sheets, dialogs, feedback, or motion-driven UI. Includes a companion RECIPES.md with Flutter SDK, Motor, and Flutter Animate implementations. Compatible with Flutter 3.47.9 stable / Dart 3.13.7 and newer stable releases.
---

# Animate Flutter Foundations

A construction skill for Flutter. It turns a request for motion into an implementation that feels intentional on a real device—not merely attractive in a simulator.

The guiding philosophy is restraint, clarity, responsiveness, and physical coherence. The goal is not to animate everything; it is to make the interface feel immediate and understandable.

## Initial response

When this skill is invoked without a specific animation request, respond only with:

> I'm ready to build purposeful Flutter animations using the foundations of motion design.

Do not provide other information until the user asks a question.

## Scope

This is a **foundations and construction** skill. It builds or modifies a specific animation.

It does not:
- Audit an entire app for animation opportunities.
- Perform a broad motion-design critique unless requested.
- Replace Flutter's native component behavior with custom animation without a reason.
- Add motion to satisfy a vague request such as “make it fancy” without identifying a purpose.

## SDK compatibility

**Baseline: Flutter 3.47.9 stable / Dart 3.13.7 and newer stable releases.**

- Use APIs available in the project's installed Flutter SDK.
- Prefer stable, non-deprecated Flutter APIs.
- Do not assume APIs introduced after Flutter 3.47.9 are available.
- When using newer APIs, verify the project's SDK version and provide a compatible alternative if needed.
- Preserve compatibility with the project's declared minimum Flutter version.
- Resolve package versions through the project's SDK constraints; do not pin arbitrary versions.
- Check official Flutter documentation when API availability or behavior is uncertain.
- Do not recommend downgrading Flutter unless a specific compatibility issue requires it.

## Operating posture

Act as a senior Flutter engineer building the animation.

Make the call, explain the reason in one line, and write the code. Do not present motion options as a menu.

Two failure modes:
1. Animating something that should not animate.
2. Animating the right thing with the wrong primitive, timing, rendering strategy, or gesture handoff.

The first is worse. A correct outcome may be zero lines of animation code.

## Hard rules

1. Run the decision sequence in order. Steps 1 and 2 gate everything.
2. Prefer Flutter's built-in animation widgets and platform behavior.
3. Use explicit controllers only when direct control is needed.
4. Choose properties and rendering strategy before tuning curves.
5. Motion must remain interruptible when the user can touch or drag it.
6. Reduced motion and accessibility ship with the implementation.
7. Never use animation to conceal slow logic or delayed data.
8. Isolate rebuilds, layout, and paint work to the smallest useful subtree.
9. Do not add a package for a transition Flutter already provides.
10. Verify feel in profile/release mode on a physical device.

# Build sequence

## 1. Should this animate at all?

Use frequency and consequence to decide.

| Frequency / context | Decision |
|---|---|
| 100+ times/day: tab switching, keyboard changes, frequent toggles, scrolling | No custom animation. Use platform behavior or immediate state change. |
| Tens of times/day: press feedback, row selection, list navigation | Near-imperceptible, typically 100–150ms, or nothing. |
| Occasional: sheets, dialogs, toasts, onboarding steps | Standard purposeful motion. |
| Rare / first-time: success states, empty-state illustration, celebration | Delight may be appropriate, but must not block the next action. |

Rules:
- Tabs are peers, not a hierarchy. Do not add sliding tab transitions unless the design explicitly communicates spatial movement.
- Do not animate keyboard-initiated or accessibility-triggered actions just for decoration.
- Do not animate every item in a frequently used list.
- Do not add motion to a state change that is clearer immediately.
- Do not animate loading time that should be fixed or represented with a progress indicator.
- Do not make the user wait for decorative motion to finish.

If it fails this gate, say so and stop. Offer the direct state change or native behavior instead.

## 2. What is the purpose?

Name one purpose before implementation:

- **Feedback** — acknowledge a press, selection, or action.
- **Spatial consistency** — show where an element came from or went.
- **State indication** — make a state change legible.
- **Preventing a jarring change** — preserve continuity through a layout/content change.
- **Explanation** — demonstrate a relationship or interaction, usually onboarding/marketing.
- **Delight** — a rare, intentional moment of personality.

If the purpose cannot be named, do not build it. “It looks cool” is not sufficient for a frequently used control.

## 3. Choose the cheapest Flutter tool that works

Walk down the list and stop at the first suitable primitive.

| Need | Flutter tool |
|---|---|
| State-driven change without gesture control | Implicit animation widgets |
| Custom tween from a changing value | `TweenAnimationBuilder<T>` |
| Start/stop/reverse/repeat/sequence | `AnimationController` |
| Animated value mapped to a widget property | `Tween`, `Animation`, `CurvedAnimation` + transition widget |
| Drag, swipe, fling, finger-controlled motion | `GestureDetector` + controller/physics |
| Scroll-driven behavior | `SliverAppBar`, `ScrollController`, `ScrollNotification`, builder/listenable |
| Enter/exit or content replacement | `AnimatedSwitcher`, `AnimatedCrossFade`, `AnimatedSize` |
| List insertion/removal | `AnimatedList`, `SliverAnimatedList`, or `Dismissible` |
| Screen transition | `Navigator` / route transition |
| Shared element between routes | `Hero` |
| Modal/sheet | `showDialog`, `showModalBottomSheet`, `DraggableScrollableSheet`, platform-native presentation |
| Custom vector/chart motion | `CustomPainter` + `Listenable`/controller |
| Physics and momentum | `SpringSimulation`, `FrictionSimulation`, `AnimationController.animateWith` |
| Press state | Existing button widget or `AnimatedScale` |
| Haptic feedback | Flutter `HapticFeedback` API |

### Implicit vs explicit

Prefer implicit animation for simple target-state changes:
- `AnimatedContainer`
- `AnimatedOpacity`
- `AnimatedScale`
- `AnimatedAlign`
- `AnimatedPadding`
- `AnimatedPositioned`
- `AnimatedDefaultTextStyle`
- `AnimatedRotation`
- `AnimatedSlide`
- `AnimatedSize`
- `AnimatedSwitcher`
- `AnimatedCrossFade`
- `TweenAnimationBuilder<T>`

Use explicit animation when you need:
- Gesture-driven progress or velocity handoff.
- Repeat, reverse, pause, or sequencing.
- A simulation.
- Scroll-linked continuous values.
- Coordinated animations.
- Lifecycle-aware playback.

Use `SingleTickerProviderStateMixin` for one controller and `TickerProviderStateMixin` for multiple controllers. Never create an `AnimationController` in `build()`.

### Dependencies

Prefer Flutter SDK APIs. Add a package only for a real capability gap. The package-aware policy below covers `motor` and `flutter_animate`; neither is mandatory. Verify compatibility against the project's Flutter/Dart constraints and resolved package version before using package-specific APIs.


### Package-aware implementation policy

Use packages as specialized tools, not as the default for every animation. The built-in Flutter animation system remains the baseline. Select the package only after the purpose and interaction model are clear.

#### Package selection

| Need | Preferred tool | Why |
|---|---|---|
| Simple target-state change (color, opacity, scale, padding) | Flutter implicit animation (`Animated*`) | Lowest ceremony and clear state-driven behavior |
| Custom timeline, controller lifecycle, reverse/repeat, exact choreography | Flutter SDK animation APIs | Full control without adding a dependency |
| Physics-based springs, velocity-preserving target changes, multi-dimensional motion, motion sequences | `motor` | Unified spring/curve API and interactive motion primitives |
| Declarative widget effects, entrance chains, staggered effects, shimmer, shake, scroll adapters | `flutter_animate` | Concise effect composition and reusable effect presets |
| Native route, sheet, tab, menu, or platform behavior | Flutter/Router/platform primitive | Preserve platform conventions instead of rebuilding them |
| Complex custom rendering or a measured view-tree bottleneck | `CustomPainter` / `CustomRenderObject` or a rendering package | Use only when ordinary widgets are insufficient |

#### `motor`

Use Motor when the motion model itself matters:
- Spring-driven motion that should respond naturally to changing targets.
- Interactive drag/return/snap behavior where velocity continuity matters.
- Multi-dimensional values such as `Offset`, `Size`, or `Rect` with independent dimensions.
- Named or staged motion sequences.
- A project wants a consistent motion abstraction across spring and duration-based animations.

Prefer built-in Flutter APIs instead when:
- A simple `AnimatedContainer`, `AnimatedScale`, or `TweenAnimationBuilder` fully solves the change.
- A standard route, sheet, scroll, or platform transition already owns the behavior.
- The interaction does not need spring physics, interruption, or Motor's unified API.

Implementation rules:
- Confirm the project's resolved Motor version and SDK constraints before using API examples.
- Prefer documented presets (`CupertinoMotion` / `MaterialSpringMotion`) over invented spring constants.
- Match the preset family to the app's design language; do not mix motion systems arbitrarily.
- For direct manipulation, track the finger without easing, then hand off release velocity to the spring/simulation.
- Do not introduce Motor solely to animate a one-off opacity or color change.
- Treat package API names as version-sensitive; check the installed package docs/changelog when writing code.

#### `flutter_animate`

Use Flutter Animate for declarative, time-based effect composition:
- Simple entrance effects such as fade, slide, or scale.
- Short sequential or parallel effect chains.
- Staggered group/list entrances when they serve a purpose.
- Reusable effect presets.
- Shimmer or other visual effects that are appropriate for a widget-level effect.
- Scroll/value-driven effects through its documented adapters when needed.

Implementation rules:
- Prefer `.animate()` chains for local, readable effects; use `Animate(child:, effects:)` when declarative composition or reusable effect lists are clearer.
- Effects run in parallel by default. Use explicit delays or `then()` when sequencing is intended; do not assume chained effects automatically run one after another.
- Keep the chain short and understandable. Split complex choreography into named helpers or use a controller/sequence abstraction.
- Avoid animating every row in a large or frequently updated virtualized list. Consider a one-time container entrance or a bounded visible subset.
- Avoid unbounded/repeating decorative effects unless they communicate ongoing state and stop when no longer relevant.
- Check whether an effect introduces blur, shader, clipping, or compositing cost; profile expensive effects on target devices.
- Do not use effect callbacks for business-critical state transitions or timing-sensitive logic.
- Respect reduced motion: disable or replace nonessential movement, looping, and stagger; preserve meaningful state feedback.
- Do not assume package defaults match the app's motion tokens. Specify duration/curve when consistency matters.

#### Dependency and compatibility rules

- Install through the project's Flutter SDK workflow: `flutter pub add motor` or `flutter pub add flutter_animate`, only after the package is justified.
- Inspect `pubspec.yaml`, lockfile, and the resolved package version. Do not copy a version constraint blindly from a web page.
- Current package pages may describe APIs newer than the project's resolved version. Confirm constructor signatures and examples against the installed version.
- Keep package use local to the feature; do not create a global abstraction around a package unless the app genuinely needs shared motion tokens.
- If both packages are present, define a clear division: Motor for spring/interactive/sequence motion; Flutter Animate for declarative effects. Avoid implementing the same motion twice.
- Package selection does not bypass the motion gates, reduced-motion handling, lifecycle, accessibility, or device verification.

## 4. Choose properties and rendering strategy

### Prefer transform and opacity when they express the same motion

Common options:
- `Transform.translate`
- `Transform.scale`
- `Transform.rotate`
- `Opacity`
- `FadeTransition`
- `SlideTransition`
- `ScaleTransition`
- `RotationTransition`

Transforms generally avoid changing layout geometry, but they do not change the widget's layout footprint. Check hit testing, clipping, semantics, and overlap.

### Layout properties

Changes to `width`, `height`, padding, margin, constraints, alignment, and flex can cause layout work. They are not forbidden, but avoid continuously animating large layouts during direct manipulation if a transform can communicate the same motion.

Use `AnimatedSize` or `AnimatedContainer` for occasional layout transitions. For a collapse/expand, measure content or use a Flutter primitive rather than hardcoding a height based on default text size.

### Rendering rules

- Never use `scale: 0` for an entrance. Start around `0.95` with opacity at zero.
- Keep transform order intentional; translation, rotation, and scale compose differently.
- Do not rebuild an entire screen for one animated value.
- Avoid expensive blur/filter changes every frame.
- Avoid animating large shadows or repeatedly rebuilding decorated subtrees.
- Keep `CustomPainter` repainting isolated with `repaint: animation` where appropriate.
- Use `RepaintBoundary` only when it isolates useful repaint work and profiling supports it.
- Do not assume `Transform` guarantees zero raster cost.
- Preserve semantics and hit testing when widgets move or scale.

## 5. Choose timing, curve, or physics

### If a finger is involved, use physics

Gestures carry velocity. Preserve it when the gesture ends or changes direction.

Use Flutter's simulation APIs when appropriate:
- `SpringSimulation`
- `FrictionSimulation`
- `AnimationController.animateWith`
- `AnimationController.fling`
- Existing `ScrollPhysics`

Do not fake a fling with a fixed-duration curve.

### Easing

| Situation | Curve |
|---|---|
| Entering / exiting | `Curves.easeOut` |
| Moving / morphing on screen | `Curves.easeInOut` |
| Constant motion / progress | `Curves.linear` |
| Default UI transition | `Curves.easeOut` |

Avoid `Curves.easeIn` for ordinary UI entrances: it delays the response at the moment the user is watching.

Use named curves consistently. Define a custom `Curve` only when a clear motion requirement is not met by a standard curve.

### Duration

| Element | Typical duration |
|---|---|
| Press feedback | 100–150ms |
| Small state change | 150–200ms |
| Tooltip / small popover | 125–200ms |
| Dropdown / lightweight overlay | 150–250ms |
| Toast / modal | 180–250ms, platform behavior where appropriate |
| Screen transition | Platform default unless a deliberate custom transition is required |

Keep ordinary UI motion under 300ms unless there is a clear reason. Platform navigation and physics-driven motion may follow platform conventions.

### Springs

Choose spring behavior based on intent:
- Default settle: near-critical, no visible overshoot.
- Drag snap-back: preserve release velocity with restrained bounce.
- Sheet/drawer: responsive spring with controlled overshoot.
- Hard boundary: clamp or use a bounded simulation.
- Flicked object: preserve momentum when it helps communicate physicality.

Do not invent arbitrary spring constants. Use a documented Flutter simulation configuration and tune on device. Bounce only when the gesture carried momentum or the design intentionally calls for playfulness.

## 6. Keep work isolated and lifecycle-safe

Flutter does not use a separate Reanimated UI runtime. Animation ticks are integrated with Flutter's frame pipeline. The goal is to isolate updates and avoid unnecessary build, layout, and paint work.

Never:
- Call `setState()` for every scroll pixel if a listenable/builder can isolate updates.
- Create controllers in `build()`.
- Start/restart an animation from `build()`.
- Perform network calls, parsing, or expensive calculations in animation listeners.
- Rebuild a large page for one animated child.
- Use periodic timers as a substitute for frame-synchronized animation.
- Run an infinite animation when offscreen or no longer meaningful.

Prefer:
- `AnimatedBuilder` with a stable `child`.
- `AnimatedWidget`, transition widgets, or `ListenableBuilder`.
- `ValueListenableBuilder` for value-driven UI.
- `CustomPainter(repaint: animation)` for custom drawing.
- `TickerMode` for inactive subtrees.
- `const` widgets and stable child subtrees.

Controller lifecycle:
1. Create in `initState()` or another lifecycle-safe location.
2. Use the correct `vsync`.
3. Update configuration safely in `didUpdateWidget()` when needed.
4. Dispose in `dispose()`.
5. Respect `TickerMode` and reduced motion.
6. Avoid work when the widget is no longer visible or relevant.

## 7. Handle interaction and interruption

### Press feedback
- Show feedback on press-down, not only after tap completion.
- Use a subtle scale around `0.97` when appropriate.
- Return to rest on release or cancellation.
- Keep it quick and interruptible.
- Prefer platform buttons when they already provide the desired behavior.

### Dragging
- Track the finger directly; do not ease while the finger is moving.
- Use gesture deltas and release velocity.
- Apply rubber-band resistance near boundaries when appropriate.
- Snap to a destination on release.
- A fast flick may dismiss without a long drag.
- A slow, short drag should return to rest.
- Handle cancellation, multi-touch, and competing scroll gestures.
- Provide a non-gesture alternative for important actions.

Flutter's gesture arena decides which recognizer wins. Do not stack competing recognizers without considering nested scrolling and platform behavior.

## 8. Haptics

Use haptics sparingly and pair them with visual feedback.

| Moment | Flutter API |
|---|---|
| Selection / step | `HapticFeedback.selectionClick()` |
| Light snap / detent | `HapticFeedback.lightImpact()` |
| Stronger commit | `HapticFeedback.mediumImpact()` |
| Heavy impact | `HapticFeedback.heavyImpact()` |

Rules:
- Fire at the causal moment, not after the animation.
- One haptic per meaningful action.
- Never per frame or ordinary scroll.
- Never as the only feedback.
- Do not assume identical output on every Android device.
- Respect platform conventions.

## 9. Reduced motion and accessibility

Reduced motion is part of the implementation.

Use Flutter accessibility settings:

```dart
final reduceMotion = MediaQuery.of(context).disableAnimations;
```

When reduced motion is enabled:
- Prefer immediate changes or short opacity/color transitions.
- Remove large translations, parallax, scale entrances, and overshoot.
- Avoid autoplaying decorative motion.
- Preserve focus, semantics, and navigation behavior.
- Do not rely on motion alone to communicate meaning.

Also consider `MediaQuery.of(context).accessibleNavigation`.

Accessibility requirements:
- Respect text scaling; do not animate to hardcoded heights based on default text.
- Avoid clipping content during transitions.
- Preserve semantics through `AnimatedSwitcher`.
- Provide accessible alternatives to swipe/drag-only actions.
- Keep controls reachable and hit targets adequate.
- Ensure focus and keyboard activation work for custom controls.

## 10. Verify on device

Use profile or release mode, not hot reload as a performance verdict.

Check:
- UI and raster frame times.
- Rebuild frequency.
- Layout/paint work.
- Clipping, opacity, filters, shadows, and large layers.
- Controller lifecycle and ticker warnings.
- Slowest supported physical device.
- 60Hz (~16.7ms) and high-refresh budgets (~8.3ms at 120Hz).

Useful commands:

```bash
flutter run --profile
flutter build apk --release
```

Use Flutter DevTools Performance view to inspect frame timing and jank. Do not claim smoothness based only on code inspection.

## Recipes

For ready-to-build implementations, see [RECIPES.md](RECIPES.md). Load it whenever the request matches a recipe; adapt the implementation to the project's architecture and SDK rather than starting from scratch.

## Never ship

| Never | Instead |
|---|---|
| Motion with no purpose | Apply the frequency and purpose gates |
| Controller for a simple target-state change | Implicit animation |
| `setState()` on every scroll tick | Isolated listenable / builder |
| Controller created in `build()` | Lifecycle-managed controller |
| Controller never disposed | Dispose in `State.dispose()` |
| Fixed-duration drag release | Physics + velocity handoff |
| Distance-only dismissal | Distance or velocity threshold |
| Hard stop at a boundary | Rubber-band resistance where appropriate |
| Custom transition between peer tabs | Native/no transition |
| Large subtree rebuilt for one value | `AnimatedBuilder` / transition widget |
| Infinite offscreen animation | Lifecycle-aware animation / `TickerMode` |
| Expensive filter every frame | Static layer or simpler effect |
| Haptics per frame | One haptic per commit |
| Haptics as only feedback | Pair with visible feedback |
| Motion ignoring reduced motion | Respect `disableAnimations` |
| Gesture-only critical action | Accessible alternative |
| Judging feel in hot reload | Profile/release build on physical device |
| Assuming APIs exist in every SDK | Check project constraints |

## Output

Write the code first. Then, in a few lines:
- **Gate result** — frequency tier and named purpose; state what you rejected.
- **Ingredients** — Flutter primitive, properties, curve/spring, duration, and rebuild strategy.
- **Device feel-check** — what to flick, interrupt, reverse, cancel, and test on the slowest supported Android/iOS device.

If the honest answer is “this should not animate,” say so and stop.

## Tone

Opinionated and brief. Make the call, explain it in one line, write the code. If the feel depends on real-device behavior, say what must be verified.
