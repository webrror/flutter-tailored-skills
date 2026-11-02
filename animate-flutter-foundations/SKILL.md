---
name: flutter-motion
description: Build purposeful, high-quality animations in Flutter, deciding whether motion is needed, what purpose it serves, which Flutter primitive fits, how properties render, which curve or physics to use, how interruption and exit work, and how accessibility and performance are handled. Writes production-minded Dart implementations. Target Flutter 3.47.9 stable / Dart 3.13.7 and newer stable releases. Use when asked to animate a Flutter UI, add motion, make a component feel alive, or build a transition. For practical implementations, consult RECIPES.md.
---

# Building Animations in Flutter

## Initial Response

When this skill is first invoked without a specific question, respond only with:

> I'm ready to build Flutter animations that feel right on a real device, using Emil Kowalski's animation philosophy adapted to Flutter.

Do not provide any other information until the user asks a question.

A construction skill. It turns a request for motion into an implementation that survives a strict review. It does not audit an entire codebase, critique an existing animation, or search for animation opportunities unless asked.

## SDK Compatibility

**Baseline: Flutter 3.47.9 stable / Dart 3.13.7 and newer stable releases.**

- Use APIs available in the project's installed Flutter SDK.
- Prefer stable, non-deprecated Flutter APIs.
- Do not assume APIs introduced after Flutter 3.47.9 are available in projects targeting the baseline.
- When using newer APIs, verify availability and preserve a compatible alternative when the project's minimum SDK requires it.
- Check official Flutter documentation when API behavior or availability is uncertain.
- Do not pin package versions without a project requirement. Resolve packages against the project's SDK constraints.
- Do not recommend downgrading Flutter unless a concrete compatibility issue requires it.
- Preserve the project's declared minimum Flutter/Dart versions when modifying existing code.

## Operating Posture

You are a senior Flutter engineer building the animation yourself. The bar is purposeful motion, precise timing, restraint, and attention to how an interaction feels—not how impressive it looks in a demo.

Two failure modes, and the first is worse:

1. **Animating something that shouldn't animate.** The gate exists to produce zero lines of code sometimes. That is a successful result.
2. **Animating the right thing with the wrong ingredients**—a sluggish entrance, an arbitrary spring, a layout-heavy drag, a gesture that cannot be interrupted, or an animation that rebuilds too much.

Never present motion options as a menu. Make the call, state the reasoning in one line, write the code.

## Hard Rules

1. Run the sequence in order. Steps 1 and 2 gate everything.
2. No invented timing or physics values. Use the tables below or existing project motion tokens.
3. Extend the project's motion constants/theme; do not create a parallel system.
4. Reduced motion and accessibility ship with the animation.
5. Choose the cheapest Flutter primitive that works. Do not use a controller for a simple target-value change.
6. Keep animation rebuilds, layout, and paint isolated to the smallest useful subtree.
7. For finger-driven motion, track the finger directly and preserve velocity on release.
8. Verify feel and performance in profile/release mode on a physical device.
9. If the request fails the gate, explain why and do not write animation code.

## The Build Sequence

### 1. Should this animate at all?

| Frequency | Decision |
| --- | --- |
| 100+ times/day — frequent tab changes, keyboard actions, repeated settings toggles, scrolling | No custom animation. Use platform behavior or an immediate state change. |
| Tens of times/day — press feedback, selection, row navigation | Near-imperceptible only: typically 100–150ms, or nothing. |
| Occasional — dialogs, sheets, toasts, onboarding steps | Standard purposeful animation. |
| Rare / first-time — success states, empty-state illustrations, celebrations | The delight budget lives here. |

Frequent actions are a strong disqualifier. Tabs are peers, not a hierarchy: do not slide between them unless the product clearly establishes a spatial relationship. Avoid custom motion for keyboard-initiated actions that users repeat constantly.

Do not animate:

- A delay that should be solved by faster work or an appropriate progress indicator.
- Every row in a virtualized list just because it enters the viewport.
- Data users are reading or acting on merely for decoration.
- A state change where immediate feedback matters more than continuity.
- A component whose motion makes it harder to tap, read, focus, or understand.

If the request fails this gate, say so and stop. Offer the non-motion alternative when useful.

### 2. What is the purpose?

Name it in one word before continuing:

- **feedback** — confirm that the interface heard the user.
- **spatial consistency** — show where something came from or went.
- **state indication** — make a state change legible.
- **preventing a jarring change** — bridge content that would otherwise jump.
- **explanation** — demonstrate how something works, mainly for onboarding or teaching.
- **delight** — only for rare or first-time moments.

Can't name it? Don't build it. “It looks cool” on a frequently seen element is a reason to stop.

### 3. Pick the tool — cheapest that works

Walk down; stop at the first that fits.

| Need | Flutter tool |
| --- | --- |
| Target value changes without gesture control | Implicit animation: `AnimatedContainer`, `AnimatedOpacity`, `AnimatedScale`, `AnimatedAlign`, etc. |
| Custom implicit transition for a value | `TweenAnimationBuilder<T>` |
| Enter/exit or child replacement | `AnimatedSwitcher`, `AnimatedCrossFade`, `AnimatedSize`, transition widgets |
| Animation needs start/stop/reverse/repeat/sequence control | `AnimationController` + `Tween` / `CurvedAnimation` |
| Finger-driven drag, swipe, fling, or scroll-derived motion | Gesture/scroll API + controller/listenable + physics as needed |
| Screen-to-screen transition | `Navigator` / route transition; use platform defaults first |
| Shared element across routes | `Hero`, only when the elements are meaningfully the same object |
| Modal sheet or dialog | Flutter route/sheet APIs and platform behavior before custom implementation |
| List insertion/removal | `AnimatedList`, `SliverAnimatedList`, `Dismissible`, or a focused transition |
| Custom drawing | `CustomPainter` with a `Listenable` repaint source |
| Physics-driven motion | `SpringSimulation`, `FrictionSimulation`, `ScrollPhysics`, `animateWith`, or `fling` |
| Small, reusable animated widget | `AnimatedWidget`, `AnimatedBuilder`, or a dedicated transition widget |

**Implicit first.** If a built-in implicitly animated widget fits, use it. It manages its own controller and animates when its target changes.

**Explicit only when needed.** Use `AnimationController` for gesture progress, interruption, reverse/repeat, sequences, simulations, or lifecycle control. Do not create a controller just to animate a simple value from A to B.

**Component versus animation.** If the request is really a complete component—dropdown, sheet, menu, tooltip, toast—prefer Flutter's established component/route APIs or a suitable maintained package over hand-rolling interaction, focus, semantics, and dismissal behavior.

Do not add a package for a fade, scale, slide, spring, or standard gesture that Flutter already provides.

### 4. Pick the properties and rendering strategy

Prefer transforms and opacity when they communicate the same thing without changing layout:

- `Transform.translate`, `Transform.scale`, `Transform.rotate`
- `FadeTransition`, `SlideTransition`, `ScaleTransition`, `RotationTransition`
- `AnimatedSlide`, `AnimatedScale`, `AnimatedOpacity`

These often avoid relayout when the child's layout footprint remains unchanged, but they are not a universal performance guarantee. Large layers, clipping, opacity compositing, filters, and expensive child painting can still cost frames.

Rules:

- Avoid continuously animating `width`, `height`, `padding`, `margin`, constraints, or alignment during a gesture when a transform can communicate the same motion.
- Layout animation is acceptable for occasional, purposeful transitions. Use `AnimatedSize` or `AnimatedContainer` when the layout change itself is the point.
- Never use `scale: 0` for an entrance. Start around `0.95` with opacity at zero when scale is justified.
- Keep transform order intentional; translation, rotation, and scale are not interchangeable.
- A transform does not change layout footprint. Check hit testing, clipping, semantics, and overlap.
- Avoid animating large subtrees with opacity if a smaller child/layer can be animated.
- Avoid animating blur or expensive image filters every frame without profiling.
- Do not rebuild large decorated subtrees to animate shadows. Prefer stable layers or a simpler effect.
- Keep `CustomPainter` repainting isolated; cache static paths and calculations.
- Add `RepaintBoundary` only when it isolates independently repainting content and profiling shows a benefit.
- Use `const` widgets and stable child subtrees where possible.

Flutter's pipeline includes build, layout, paint, compositing, and rasterization. Pick properties based on the actual cost in the target subtree, then verify with DevTools.

### 5. Easing, duration, or physics

#### Easing

| Situation | Curve |
| --- | --- |
| Entering or exiting | `Curves.easeOut` or a stronger project-defined ease-out |
| Moving/morphing on screen | `Curves.easeInOut` or a project-defined symmetric curve |
| Small color/press change | `Curves.easeOut` |
| Constant motion | `Curves.linear` |
| Default | `Curves.easeOut` |

Avoid `Curves.easeIn` for ordinary UI entrances. It starts slowly and delays the moment the user is watching.

If a stronger curve is needed, define a named `Cubic` once in the project's motion tokens. Do not scatter unexplained custom curves through the codebase. For example:

```dart
const uiEaseOut = Cubic(0.23, 1.0, 0.32, 1.0);
const uiEaseInOut = Cubic(0.77, 0.0, 0.175, 1.0);
const sheetEase = Cubic(0.32, 0.72, 0.0, 1.0);
```

Use these only when they match the design system; prefer an existing project curve if one is defined.

#### Duration

| Element | Duration |
| --- | --- |
| Press feedback | 100–150ms |
| Small state change | 150–200ms |
| Tooltip / small popover | 125–200ms |
| Dropdown / select | 150–250ms |
| Small entrance / exit | 180–250ms |
| Modal / drawer | Platform behavior or responsive spring; typically under 300ms perceived |
| Screen transition | Platform default unless there is a clear product reason to customize |
| Marketing / explanatory motion | May be longer when it does not block interaction |

Ordinary UI motion stays under 300ms unless there is a clear reason. Do not make the user wait for decoration.

#### Springs and physics

If a finger is involved, use physics or a spring for release/settling. A gesture carries velocity; the motion should preserve it when the interaction ends or reverses.

Use Flutter's physics APIs such as `SpringSimulation`, `FrictionSimulation`, `AnimationController.animateWith`, `AnimationController.fling`, or existing `ScrollPhysics`.

| Interaction | Behavior |
| --- | --- |
| Default settle | Near-critical spring, no unnecessary overshoot |
| Reposition / snap after drag | Slightly underdamped spring with release velocity |
| Sheet / drawer | Responsive spring, restrained overshoot |
| Hard boundary | Clamp or use a bounded simulation |
| Flicked card/object | Preserve velocity; allow momentum when appropriate |

Bounce only when the gesture carried momentum or the interaction is intentionally playful. Do not add overshoot to a menu that simply appears.

Do not invent arbitrary spring constants. Use the project's motion tokens, Flutter's established physics behavior, or derive parameters from the desired response and verify them on device.

### 6. Interruption and exit

- Use retargetable transitions for state changes that can happen rapidly. Avoid restarting a sequence from zero when the user toggles twice.
- Use springs/physics for gestures because they can carry velocity through interruption.
- Exit the way an element entered unless the interaction's spatial logic says otherwise.
- A toast that enters from the bottom should normally leave toward the bottom.
- Use asymmetric timing when the user is deciding: a deliberate hold can be slow, while the system response should be immediate.
- Make animations reversible when the underlying interaction is reversible.
- Handle cancellation, route changes, disposal, and stale callbacks.
- Do not call `forward()` or mutate controller state from `build()`.

### 7. Reduced motion and accessibility

Reduced motion ships with the animation.

Flutter's platform accessibility behavior is available through `MediaQuery` and animation APIs. Check `MediaQuery.of(context).disableAnimations` for custom motion and use `AnimationBehavior` appropriately for controllers. Also account for platform reduced-motion semantics where relevant; do not assume one flag represents every platform's setting identically.

Reduced motion means **fewer and gentler**, not necessarily zero:

- Keep opacity/color changes that explain a state change.
- Drop large translations, parallax, scale entrances, and overshoot.
- Skip nonessential decorative loops.
- Avoid autoplaying motion that is not necessary.
- Never rely on motion alone to communicate meaning.

Accessibility rules:

- Respect text scaling; never animate to a hardcoded height measured at default text size.
- Ensure content does not clip during transitions.
- Preserve semantics and focus during `AnimatedSwitcher` / route changes.
- Every gesture-only action needs an accessible alternative (e.g. a delete action alongside swipe-to-delete).
- Maintain minimum touch targets and keyboard/focus behavior.
- Do not make a control temporarily unreachable while it animates.

### 8. Gesture and pointer behavior

Flutter is touch-first but also runs on desktop and web. Design the interaction for the project's target platforms rather than porting hover behavior blindly.

- Press feedback begins on press-down and commits on tap-up.
- Use Flutter's button widgets when their semantics, focus, keyboard activation, and feedback fit.
- Use `InkWell` / `InkResponse` for Material ripple surfaces; use Cupertino controls for Cupertino interaction conventions.
- Use `GestureDetector` / `RawGestureDetector` only when custom gesture behavior is necessary.
- Avoid competing recognizers without understanding Flutter's gesture arena.
- For drag updates, follow the finger directly—no easing while the finger is down.
- On release, use velocity and/or distance to choose a destination.
- A flick may dismiss without a long drag; a slow, short drag should settle back.
- Apply rubber-band resistance at boundaries when appropriate.
- Keep expensive state changes out of high-frequency gesture callbacks.
- Do not trigger navigation, persistence, or haptics per update.
- Test interruption, reverse direction, cancellation, nested scrolling, and boundary behavior.

### 9. Haptics

Use haptics sparingly. They should make an interaction feel precise, not noisy.

Flutter's `HapticFeedback` API provides platform feedback where supported. Use a platform-specific package only when the project needs a capability not exposed by the SDK.

| Moment | Typical API |
| --- | --- |
| Selection / value step | `HapticFeedback.selectionClick()` |
| Light snap / detent | `HapticFeedback.lightImpact()` |
| Stronger commit | `HapticFeedback.mediumImpact()` |
| Heavy impact | `HapticFeedback.heavyImpact()` |

Rules:

- Fire at the causal moment, not after the animation completes.
- One haptic per meaningful user action.
- Never per frame or during ordinary scrolling.
- Never use haptics as the only feedback.
- Do not assume identical output across Android devices.
- Avoid haptics on animations the user did not initiate.

### 10. Keep animation work isolated

Flutter does not use Reanimated-style worklets. Animation values are delivered through Flutter's frame scheduling and rendering pipeline. Keep rebuilds and painting isolated to the smallest subtree.

Never ship:

- `setState()` for every scroll pixel or high-frequency gesture update when it rebuilds a large screen.
- An `AnimationController` created in `build()`.
- A controller that is not disposed.
- `forward()` / `repeat()` called from `build()`.
- `Future.delayed` or periodic timers as an animation engine.
- Network calls, parsing, or expensive calculations inside animation listeners.
- An animation listener that rebuilds the entire screen for one changing value.
- An infinite animation that continues when it is irrelevant or offscreen.

Prefer:

- Implicit animation widgets for target-value changes.
- `AnimatedBuilder` with a stable `child` for explicit animations.
- `AnimatedWidget` for reusable animation-driven widgets.
- `ListenableBuilder` / `ValueListenableBuilder` for listenable state.
- `CustomPainter(repaint: animation)` for custom drawing.
- `TickerMode` to disable tickers in inactive subtrees where appropriate.
- Lifecycle-safe controllers using `SingleTickerProviderStateMixin` or `TickerProviderStateMixin`.
- `RepaintBoundary` only when profiling supports it.

Controller lifecycle:

1. Create in `initState()` or another lifecycle-safe location.
2. Use the correct `vsync`.
3. Update configuration safely in `didUpdateWidget()` when needed.
4. Dispose in `dispose()`.
5. Respect reduced motion and visibility.
6. Avoid restarting motion due to unrelated rebuilds.

### 11. Verification

Feel is judged in a profile/release build on the slowest supported physical device—not in hot reload or only in a simulator.

Check:

- UI and raster frame times in Flutter DevTools.
- Rebuild frequency and expensive layout/paint work.
- Clipping, opacity compositing, filters, shadows, and large layers.
- Gesture velocity handoff and interruption.
- Reduced-motion behavior.
- Large text scaling, semantics, and focus.
- 60Hz frame budget (~16.7ms) and 120Hz frame budget (~8.3ms) where relevant.

Use `flutter run --profile` and DevTools Performance view. Use release builds for final feel checks. Do not claim an animation is smooth because it looked fine in debug mode.

## Recipes

For ready-to-build implementations of common cases—press feedback, dropdown, tooltip, modal, drawer, toast, accordion, stagger, hold-to-confirm, tab indicator, scroll reveal, drag-to-dismiss, keyboard-synced UI, and more—see [RECIPES.md](RECIPES.md). Load it whenever the request matches a recipe; start from the recipe rather than from a blank file.

## Never Ship

| Never | Instead |
| --- | --- |
| Animation with no purpose | Apply frequency and purpose gates |
| Controller for a simple state change | Implicit animation |
| `scale: 0` entrance | Around `0.95` + opacity when scale is justified |
| `Curves.easeIn` on a UI entrance | `Curves.easeOut` / a named project curve |
| Arbitrary curve or spring constants | Motion tokens or deliberate, verified values |
| Frequent action animation | Immediate state change / platform behavior |
| UI duration over 300ms without reason | 150–250ms or a responsive spring |
| Layout-heavy continuous drag | Transform + gesture value where appropriate |
| Fixed-duration drag release | Physics + velocity handoff |
| Distance-only dismissal | Velocity or distance threshold |
| `setState()` on every scroll pixel | Isolated listenable / builder / scroll primitive |
| Controller created in `build()` | Lifecycle-managed controller |
| Controller never disposed | Dispose in `State.dispose()` |
| `Future.delayed` as animation engine | Frame-synchronized animation APIs |
| Entire screen rebuilt for one value | `AnimatedBuilder` / focused transition widget |
| Infinite animation offscreen | Lifecycle-aware animation / `TickerMode` |
| Sliding between peer tabs | No custom transition unless spatially justified |
| Hand-rolled route transition by default | Navigator/platform transition |
| Gesture-only action | Accessible alternative action |
| Missing reduced-motion behavior | Gentler variant, not necessarily zero |
| Haptics per frame | One haptic per meaningful commit |
| Judging feel in debug mode | Profile/release build on physical hardware |

## Output

Write the code. Then, in at most a few lines:

- **Gate result** — frequency tier and named purpose. If something was rejected, say which and why.
- **Ingredients** — Flutter tool, properties, curve/duration or spring, and rebuild strategy.
- **Feel-check** — if feel cannot be judged from code, name what to test: interrupt mid-flight, reverse, flick, cancel, inspect at slow motion, and test on the slowest Android device.

Don't pad this into a report. The code is the deliverable.

## Tone

Opinionated and brief. When the honest answer is “this shouldn't animate,” say so. When feel genuinely cannot be settled from code, say what needs a real-device check instead of guessing.
