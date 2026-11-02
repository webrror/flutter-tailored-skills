---
name: animate-flutter
description: Build purposeful, high-quality Flutter animations that feel right on real devices. Decide whether motion is needed, choose the right Flutter animation primitive, handle gestures and velocity, optimize rendering, respect accessibility, and verify performance. Uses Flutter's implicit and explicit animation APIs, AnimationController, physics simulations, GestureDetector, Scrollable, Navigator, haptics, and custom painting. Compatible with Flutter 3.47.9 stable and newer releases. Use when animating anything in a Flutter app, adding gestures, sheets, screen transitions, press feedback, scroll effects, custom motion, or fixing animations that stutter on device.
---

# Building Animations in Flutter

A construction skill for Flutter. Turn a motion request into an implementation that survives review on a real device—not just hot reload, not just the simulator, and not just a flagship phone.

The goal is not to make every interaction move. The goal is to make the interface feel immediate, understandable, and physically coherent.

This skill follows the animation philosophy of Emil Kowalski: purposeful motion, precise timing, restraint, and attention to how an interaction feels rather than how impressive it looks.

## SDK Compatibility

**Baseline: Flutter 3.47.9 stable / Dart 3.13.7 and newer stable releases.**

- Target Flutter 3.47.9 or any newer stable Flutter release.
- Use APIs available in the project's installed Flutter SDK.
- Prefer stable, non-deprecated Flutter APIs.
- Do not assume APIs introduced after Flutter 3.47.9 are available.
- When using newer APIs, verify the project's Flutter version and provide a compatible alternative when necessary.
- Check official Flutter documentation when API availability or behavior is uncertain.
- Avoid pinning package versions. Resolve compatible versions using the project's SDK constraints.
- Do not recommend downgrading Flutter unless a specific compatibility issue requires it.
- Preserve compatibility with the project's minimum supported Flutter version when modifying existing code.
- When a newer SDK provides a better native animation primitive, use it only when the project's minimum SDK supports it.

## Operating Posture

You are a senior Flutter engineer building the animation yourself.

Make the call, state the reasoning in one line, write the code.

Never present motion options as a menu. Choose the cheapest implementation that produces the intended feel.

Two failure modes:

1. Animating something that should not animate.
2. Animating the right thing in a way that rebuilds too much, fights the gesture, or drops frames on a real device.

The first is worse.

## Hard Rules

1. Run the build sequence in order. Steps 1 and 2 gate everything.
2. Prefer Flutter's built-in animation widgets when they solve the problem.
3. Use explicit controllers only when the animation needs direct control.
4. Never approximate timing or spring parameters without a reason.
5. Keep animation work isolated from unrelated widget rebuilds.
6. Reduced motion and accessibility ship with the implementation.
7. Never use animation to hide slow application logic.
8. Judge motion in a profile or release build on the slowest supported device.
9. Do not introduce a dependency for an animation Flutter already handles.
10. If the animation fails the gate, write zero animation code.

---

# The Build Sequence

## 1. Should this animate at all?

Frequency determines how much motion the user can tolerate.

| Frequency | Decision |
|---|---|
| 100+ times/day — tab switches, keyboard changes, scrolling, frequent toggles | No custom animation. Use platform behavior or a direct state change. |
| Tens of times/day — press feedback, selection, list navigation | Near-imperceptible: typically 100–150ms, or nothing. |
| Occasional — sheets, dialogs, toasts, onboarding steps | Standard purposeful animation. |
| Rare / first-time — success states, empty-state illustrations, celebrations | The delight budget lives here. |

**Tab switches are peers, not a hierarchy.** Do not slide between tabs unless the design explicitly establishes a spatial relationship. Prefer the app's existing navigation system and platform conventions.

Do not animate:

- A value that updates constantly but has no meaningful visual transition.
- A loading delay that should be solved by faster data or an appropriate progress indicator.
- Every item in a large scrolling list just because it entered the viewport.
- A layout change that makes content harder to read or interact with.
- Motion that delays access to the next action.
- A state change where immediate feedback is more important than visual continuity.

If the request fails this gate, say so and don't write the animation.

## 2. What is the purpose?

Name it in one word before continuing:

- **feedback** — acknowledge a press, selection, or action.
- **spatial consistency** — show where something came from or where it went.
- **state indication** — make a state change understandable.
- **preventing a jarring change** — preserve continuity when layout or content changes.
- **explanation** — teach a relationship or interaction.
- **delight** — rare, intentional moments of personality.

If the purpose cannot be named, do not build it.

Every animation must have a clear start state, end state, and reason for existing.

## 3. Pick the Flutter tool — cheapest that works

Walk down the table and stop at the first solution that fits.

| Need | Flutter tool |
|---|---|
| Simple state-driven change — color, size, alignment, opacity | Implicit animation widgets |
| Custom implicit transition across multiple properties | `TweenAnimationBuilder<T>` |
| Animation that must start, stop, reverse, repeat, or sequence | `AnimationController` |
| Drag, swipe, fling, or finger-controlled motion | `GestureDetector` / `RawGestureDetector` + controller or physics |
| Scroll-derived visual effect | `ScrollController`, `ScrollNotification`, `SliverAppBar`, or a scroll-aware builder |
| Widget mounting or unmounting | `AnimatedSwitcher`, `AnimatedCrossFade`, `AnimatedSize` |
| Items changing position in a list | `AnimatedList`, `SliverAnimatedList`, `AnimatedPositioned`, or purpose-built layout |
| Screen-to-screen transition | `Navigator` / `PageRoute` transition |
| Shared element between screens | `Hero` |
| Modal bottom sheet | `showModalBottomSheet` with appropriate route animation |
| Native-feeling sheet or dialog | Platform widgets and route behavior |
| Custom path, chart, or complex visual | `CustomPainter` + `AnimationController` |
| Large animated scene or particle system | `CustomPainter`; specialized rendering only when justified |
| Physics-driven simulation | `AnimationController.animateWith`, `SpringSimulation`, `FrictionSimulation`, or `ScrollPhysics` |
| Complex sequence | `AnimationController` + `Interval` / `TweenSequence` |
| Simple press state | `AnimatedScale` or `AnimatedContainer` |
| Haptics | Flutter's `HapticFeedback` API or a justified platform package |

### Implicit animation decision

Use implicit animations first.

Examples:

- `AnimatedContainer`
- `AnimatedOpacity`
- `AnimatedScale`
- `AnimatedAlign`
- `AnimatedPadding`
- `AnimatedPositioned`
- `AnimatedDefaultTextStyle`
- `AnimatedPhysicalModel`
- `AnimatedRotation`
- `AnimatedSlide`
- `AnimatedSize`
- `AnimatedSwitcher`
- `AnimatedCrossFade`
- `TweenAnimationBuilder<T>`

They are appropriate when a target value changes and Flutter should handle the transition.

Do not introduce an `AnimationController` just to animate a value from A to B once.

### Explicit animation decision

Use an explicit controller when the implementation needs:

- Gesture-controlled progress.
- Interruptible motion with velocity.
- Reverse, repeat, or pause behavior.
- Multiple animations coordinated on one timeline.
- A simulation or fling.
- Lifecycle-aware animation.
- Progress controlled by scroll or another continuous input.

Use `SingleTickerProviderStateMixin` for one controller and `TickerProviderStateMixin` for multiple controllers.

Do not create controllers in `build()`.

### Dependencies

Prefer Flutter SDK APIs.

Add packages only when a real capability gap exists.

| Need | Package / API |
|---|---|
| Standard motion | Flutter SDK |
| Gesture recognition | `GestureDetector`, `Scrollable`, `RawGestureDetector` |
| Haptics | `HapticFeedback` |
| Complex vector illustration | `lottie` when authored animation assets are needed |
| Custom graphics / very large scenes | `CustomPainter`; consider specialized rendering only when justified |
| Advanced physics / specialized motion | Evaluate package compatibility and maintenance before adding one |

Do not install a package for a fade, scale, slide, spring, or standard gesture.

---

## 4. Pick the properties

### Prefer transform and opacity

The cheapest visual changes are generally:

- `Transform.translate`
- `Transform.scale`
- `Transform.rotate`
- `Opacity`
- `FadeTransition`
- `SlideTransition`
- `ScaleTransition`
- `RotationTransition`

These can often avoid layout work when the child's layout remains unchanged.

But a transform does not change the layout footprint. Make sure hit testing, clipping, and semantics still match the visual result.

### Layout animation costs

Changes to these properties can cause layout work:

- `width`
- `height`
- `padding`
- `margin`
- `constraints`
- `alignment`
- `flex`
- `position`

They are not forbidden. They are simply more expensive when updated every frame.

Use `AnimatedSize` or `AnimatedContainer` for occasional layout transitions. Avoid continuously animating large layouts during a gesture when a transform can communicate the same motion.

### Rules

- Never use `Opacity` to hide a widget that can simply be removed from the tree, unless a fade is intended.
- Avoid animating large, complex subtrees through opacity when a smaller layer can be animated.
- Do not use `Transform.scale(0)` for entrances. Start near `0.95` with opacity at zero.
- Keep transform order intentional. Translation, rotation, and scale are not interchangeable.
- Avoid rebuilding a large subtree for every animation tick.
- Use `ClipRect` / `ClipRRect` only when clipping is necessary; clipping can add rendering cost.
- Do not animate blur sigma or expensive image filters every frame without profiling.
- Do not animate shadows by rebuilding large decorated widgets every frame. Prefer a stable shadow layer or opacity transition where possible.
- Keep `CustomPainter` repainting isolated with a `Listenable` when appropriate.
- Use `RepaintBoundary` when it isolates an independently repainting subtree and profiling shows it helps. Do not add it everywhere.

### Flutter's rendering pipeline

Flutter animation values update on frame ticks. Depending on the implementation, the widget tree may rebuild, layout may run, and paint may occur.

A `Transform` is not a universal performance guarantee. Large composited layers, clipping, opacity, raster effects, and expensive child painting can still be costly.

Profile the actual subtree.

---

## 5. Timing or physics?

### If a finger is involved, use physics

A gesture carries velocity. The animation should preserve that velocity when the interaction ends or changes direction.

Use:

- `SpringSimulation`
- `ScrollSpringSimulation`
- `FrictionSimulation`
- `AnimationController.animateWith`
- `AnimationController.fling`
- `ScrollPhysics` for scrollable behavior

Do not fake a fling with a fixed-duration `CurvedAnimation`.

### Spring behavior

Choose the spring based on the physical intent.

| Interaction | Behavior |
|---|---|
| Default settle, no overshoot | Critically damped or near-critical spring |
| Reposition / snap after drag | Slightly underdamped spring with velocity handoff |
| Sheet / drawer | Responsive spring with restrained overshoot |
| Must not cross a hard edge | Clamp or use a bounded simulation |
| Flicked card / object | Preserve gesture velocity; allow momentum when appropriate |

Use Flutter's spring simulation APIs with explicit parameters. Do not invent arbitrary constants until the motion feels plausible.

A spring should settle quickly without wobbling unnecessarily.

**Bounce only when the gesture carried momentum.** A menu fading in should not bounce. A card flicked by the user may.

### Easing

For motion without a finger:

| Situation | Curve |
|---|---|
| Entering or exiting | `Curves.easeOut` |
| Moving / morphing on screen | `Curves.easeInOut` |
| Constant motion | `Curves.linear` |
| Default UI transition | `Curves.easeOut` |

Avoid `Curves.easeIn` for ordinary UI entrances. It delays the visual response at the moment the user is watching.

Use named curves consistently. If a custom curve is needed, define it once and explain why it is better than a standard curve.

### Duration

| Element | Duration |
|---|---|
| Press feedback | 100–150ms |
| Small state change | 150–200ms |
| Small entrance / exit | 180–250ms |
| Toast / lightweight overlay | 180–250ms |
| Modal / sheet | Platform behavior or a responsive spring |
| Screen transition | Platform default unless the design requires otherwise |

Keep ordinary UI motion under 300ms.

Do not make users wait for a decorative animation before they can continue.

---

## 6. Keep rebuilds and painting under control

Flutter does not have a separate Reanimated-style UI runtime. Its animation system is driven by frame scheduling and the rendering pipeline.

The craft is isolating animation work so that only the necessary subtree updates.

### Never ship

- `setState()` from a high-frequency gesture callback when it rebuilds a large screen.
- Calling `setState()` for every scroll pixel.
- Creating an `AnimationController` inside `build()`.
- Recreating tweens, controllers, or expensive objects unnecessarily during builds.
- Calling `controller.forward()` from `build()`.
- Using `Future.delayed` as an animation engine.
- Driving animations with periodic timers when frame-synchronized animation is needed.
- Animating the entire page when only one child needs to move.
- Performing network calls, parsing, or expensive calculations inside animation listeners.
- Calling `setState()` from an animation listener when an `AnimatedBuilder` or transition widget would isolate the rebuild.
- Running an infinite animation when the widget is offscreen or no longer meaningful.

### Prefer

- `AnimatedBuilder` with a stable `child` for static subtrees.
- `ListenableBuilder` for non-animation listenables.
- `AnimatedWidget` for reusable animation-driven widgets.
- `ValueListenableBuilder` for value-driven UI.
- `CustomPainter(repaint: animation)` for custom drawing.
- `TickerMode` to disable tickers in inactive subtrees.
- `RepaintBoundary` only when it isolates useful repaint work.
- `const` widgets wherever possible.
- Lifecycle-aware controllers that are disposed correctly.

### Controller lifecycle

Every `AnimationController` must:

1. Be created in `initState()` or an equivalent lifecycle-safe location.
2. Use the correct `vsync`.
3. Be updated or replaced safely in `didUpdateWidget()` when configuration changes.
4. Be disposed in `dispose()`.
5. Respect `TickerMode` and reduced-motion preferences.
6. Avoid continuing work when the widget is no longer visible or relevant.

Use `AnimationController.unbounded` only when the animation genuinely needs an unbounded value.

### Gesture callbacks

For high-frequency gesture updates:

- Update only the value needed for motion.
- Keep expensive state changes out of `onPanUpdate`.
- Do not trigger navigation, database writes, or haptics per update.
- Commit the actual action in `onPanEnd`, `onTap`, or the relevant completion callback.
- Use `details.velocity` when handing motion into a simulation.
- Handle cancellation and interruption deliberately.

Flutter's gesture arena determines which recognizer wins. Do not layer competing gesture recognizers blindly. Consider scroll behavior, nested gestures, accessibility actions, and platform conventions.

---

## 7. Press, not hover

Mobile affordances must work without hover.

For press feedback:

- Animate on press-down, not only after the tap completes.
- Use a subtle scale around `0.97` when appropriate.
- Return to `1.0` on release or cancellation.
- Keep feedback around 100–150ms.
- Make the press interruptible.
- Use the same visual language across platforms unless platform-specific behavior is intentional.

Use `InkWell` / `InkResponse` for Material surfaces where ripple is part of the design.

Use `GestureDetector` or `Listener` when custom interaction behavior is required.

Use `CupertinoButton` or other Cupertino widgets when matching iOS interaction behavior.

Do not combine a custom scale animation with a ripple unless the combination has been deliberately designed.

### Touch targets

- Minimum touch target: 48dp on Android.
- Aim for approximately 44pt on iOS.
- Use `MaterialTapTargetSize`, padding, or a larger hit region instead of enlarging the visual element.
- Ensure transformed widgets remain easy to hit.
- Preserve semantics and focus behavior.

Press feedback should acknowledge the touch without making the control feel soft, delayed, or unstable.

---

## 8. Gestures and velocity handoff

The user's finger is the primary input. Motion must remain interruptible.

### Drag interactions

- Track the finger directly.
- Do not apply easing while the finger is moving.
- Use the gesture's actual delta and velocity.
- Apply rubber-band resistance at boundaries where appropriate.
- Snap to a destination on release.
- Allow a flick to complete a dismissal even when the distance is short.
- Do not require both a large distance and high velocity.
- Provide a clear cancellation path.

### Dismissal thresholds

Use either distance or velocity:

- A deliberate drag beyond a threshold can dismiss.
- A fast flick can dismiss without a long drag.
- A slow, short drag should return to the original position.

Do not use distance alone.

### Scroll-linked motion

Prefer existing Flutter scroll primitives when possible:

- `SliverAppBar`
- `FlexibleSpaceBar`
- `ScrollController`
- `ScrollNotification`
- `AnimatedBuilder`
- `CustomScrollView`

Do not use `setState()` for every scroll update if a listenable or builder can isolate the effect.

Do not create custom parallax or collapsing behavior unless the design needs it.

### Interruptibility

Test:

- Dragging in the opposite direction mid-animation.
- Releasing at different velocities.
- Re-grabbing an element while it is settling.
- Cancelling a gesture.
- Reaching a boundary.
- Competing with a scroll gesture.

The element should never feel like it is moving independently of the user's finger.

---

## 9. Haptics

Use haptics sparingly. They should make an interaction feel precise, not noisy.

Flutter's `HapticFeedback` API provides platform feedback where supported.

| Moment | Call |
|---|---|
| Selection / value step | `HapticFeedback.selectionClick()` |
| Light snap / detent | `HapticFeedback.lightImpact()` |
| Strong commit / heavier action | `HapticFeedback.mediumImpact()` |
| Strong impact | `HapticFeedback.heavyImpact()` |
| Success / warning / error | `HapticFeedback` or a justified platform-specific implementation |

### Rules

- Fire haptics at the causal moment, not after the animation completes.
- Use one haptic per meaningful user action.
- Never trigger haptics per frame or during ordinary scrolling.
- Never use haptics as the only feedback.
- Respect platform behavior and user expectations.
- Do not assume identical haptic output across Android devices.
- Avoid haptics on animations the user did not initiate.

If haptics are required from a custom platform integration, isolate the platform call behind a small service.

---

## 10. Reduced motion and accessibility

Reduced motion is part of the animation implementation, not a later enhancement.

Use Flutter's accessibility settings:

```dart
final reduceMotion =
    MediaQuery.of(context).disableAnimations;
```

Where appropriate, also consider `MediaQuery.of(context).accessibleNavigation`.

### Reduced-motion behavior

When reduced motion is enabled:

- Keep state changes understandable.
- Prefer immediate transitions or short opacity changes.
- Remove large translations, parallax, scale entrances, and overshoot.
- Avoid autoplaying decorative motion.
- Do not rely on motion alone to communicate meaning.
- Preserve focus, semantics, and navigation behavior.

Do not blindly disable all animation. A short fade or color transition can still communicate state.

### Text and layout

- Respect text scaling.
- Never animate to a hardcoded height based on default text size.
- Use `AnimatedSize` when content height genuinely changes.
- Test with large accessibility text.
- Ensure content does not clip during transitions.
- Preserve semantics during `AnimatedSwitcher` transitions.
- Avoid making a control temporarily inaccessible while it animates.

### Accessibility actions

Every gesture-only interaction must have an accessible alternative.

For example:

- A swipe-to-delete row also needs a delete action.
- A drag-to-dismiss sheet needs a close button or equivalent action.
- A custom slider needs appropriate semantics and keyboard behavior where applicable.
- A drag handle should not be the only way to move or resize an element.

---

# Flutter-specific animation patterns

## Implicit state change

Use this for a small visual state change with no gesture-controlled progress.

```dart
AnimatedContainer(
  duration: const Duration(milliseconds: 180),
  curve: Curves.easeOut,
  width: isSelected ? 48 : 40,
  height: isSelected ? 48 : 40,
  decoration: BoxDecoration(
    color: isSelected ? selectedColor : unselectedColor,
    borderRadius: BorderRadius.circular(12),
  ),
  child: Icon(
    isSelected ? Icons.check : Icons.circle_outlined,
  ),
)
```

Purpose: state indication.

Do not use a controller for this.

## Press feedback

Use a state-driven transition rather than a frame-by-frame controller.

```dart
class PressScale extends StatefulWidget {
  const PressScale({
    super.key,
    required this.child,
    required this.onPressed,
  });

  final Widget child;
  final VoidCallback onPressed;

  @override
  State<PressScale> createState() => _PressScaleState();
}

class _PressScaleState extends State<PressScale> {
  bool _pressed = false;

  void _setPressed(bool value) {
    if (_pressed == value) return;
    setState(() => _pressed = value);
  }

  @override
  Widget build(BuildContext context) {
    final reduceMotion =
        MediaQuery.of(context).disableAnimations;

    return GestureDetector(
      behavior: HitTestBehavior.opaque,
      onTapDown: (_) => _setPressed(true),
      onTapUp: (_) => _setPressed(false),
      onTapCancel: () => _setPressed(false),
      onTap: widget.onPressed,
      child: AnimatedScale(
        scale: _pressed && !reduceMotion ? 0.97 : 1,
        duration: const Duration(milliseconds: 120),
        curve: Curves.easeOut,
        child: widget.child,
      ),
    );
  }
}
```

For production controls, prefer Flutter's existing button widgets when their interaction behavior fits. Custom press wrappers must preserve semantics, focus, keyboard activation, and minimum touch targets.

## Explicit controller

Use this when the animation needs direct lifecycle control.

```dart
class Entrance extends StatefulWidget {
  const Entrance({
    super.key,
    required this.child,
  });

  final Widget child;

  @override
  State<Entrance> createState() => _EntranceState();
}

class _EntranceState extends State<Entrance>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _opacity;
  late final Animation<Offset> _offset;

  @override
  void initState() {
    super.initState();

    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 220),
    );

    _opacity = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeOut,
    );

    _offset = Tween<Offset>(
      begin: const Offset(0, 0.04),
      end: Offset.zero,
    ).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.easeOut,
      ),
    );
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();

    if (MediaQuery.of(context).disableAnimations) {
      _controller.value = 1;
    } else if (!_controller.isAnimating &&
        !_controller.isCompleted) {
      _controller.forward();
    }
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return FadeTransition(
      opacity: _opacity,
      child: SlideTransition(
        position: _offset,
        child: widget.child,
      ),
    );
  }
}
```

Purpose: preventing a jarring change.

Keep the animated child stable. Do not rebuild the entire screen for each tick.

## Gesture-driven motion

For a drag:

1. Use a gesture recognizer.
2. Track the finger with a controller or animation value.
3. Do not apply easing during direct manipulation.
4. On release, use gesture velocity to choose the destination or simulation.
5. Keep updates local to the animated subtree.

Use `AnimationController.animateWith()` and a physics simulation when the release should preserve momentum.

Do not replace gesture physics with an arbitrary timer.

## Screen transitions

Use Flutter's navigation system.

- Prefer the platform's default route transition.
- Use `PageRouteBuilder` only when a custom transition is required.
- Keep transitions consistent across the app.
- Respect reduced motion.
- Avoid sliding between peer tabs.
- Use `Hero` for a genuine shared-element relationship, not as decoration.

## Sheets and dialogs

Prefer:

- `showModalBottomSheet`
- `showDialog`
- `DraggableScrollableSheet`
- Platform-native sheet behavior

Do not hand-roll a sheet if Flutter's route or sheet APIs already provide the correct interaction.

When custom motion is justified:

- Use a spring.
- Preserve drag velocity.
- Support interruption.
- Handle drag cancellation.
- Keep the dismiss threshold deliberate.
- Respect safe areas and keyboard insets.
- Provide a close action independent of dragging.

## List entrances

Do not animate every row in a virtualized list.

Use list entrance animation only when it provides meaningful hierarchy or explanation.

- Keep the number of animated items small.
- Avoid staggered animation for frequently refreshed lists.
- Do not rely on `GlobalKey` for every row.
- Avoid expensive clipping and opacity on large subtrees.
- Ensure recycled list items do not replay unexpectedly.
- Prefer animating the container or a small child rather than the whole row.

## Custom painting

Use `CustomPainter` when the widget tree is not the right abstraction.

- Keep the painter deterministic.
- Use `repaint` with a `Listenable` when appropriate.
- Avoid allocating objects on every paint.
- Cache static paths and expensive calculations.
- Use `shouldRepaint` correctly.
- Separate static and animated layers.
- Profile before moving to a shader or specialized renderer.

---

# Setup and performance checks

Before debugging an animation that stutters or fails to run:

- Confirm the project is using the intended Flutter SDK.
- Check `flutter doctor -v`.
- Check controller lifecycle and `vsync`.
- Confirm the widget is mounted and not disposed.
- Check `TickerMode`.
- Check for `TickerCanceled` and lifecycle errors.
- Confirm the animation isn't being restarted on every build.
- Inspect rebuilds using Flutter DevTools.
- Inspect raster and UI frame times.
- Check clipping, opacity, shadows, and image filters.
- Test on a physical device in profile or release mode.
- Test the slowest supported device.

### Flutter version notes

- Target Flutter 3.47.9 stable or newer stable releases.
- Use Dart APIs supported by the project's installed SDK.
- Do not assume a package version from an older Flutter project is compatible.
- Re-check platform behavior after SDK upgrades.
- Verify newer APIs against official documentation before using them.
- Preserve compatibility with the project's minimum supported SDK.
- Do not use deprecated APIs when a stable replacement is available.

### Performance verification

Use:

```bash
flutter run --profile
```

And:

```bash
flutter build apk --release
```

Use Flutter DevTools Performance view to inspect:

- UI thread frame time.
- Raster thread frame time.
- Janky frames.
- Rebuild frequency.
- Shader compilation or rasterization issues.
- Excessive layout and paint work.

A smooth animation should stay within the frame budget:

- 60Hz: approximately 16.7ms per frame.
- 120Hz: approximately 8.3ms per frame.

Do not claim an animation is smooth because it looked fine in hot reload.

---

# Never ship

| Never | Instead |
|---|---|
| Animation with no purpose | Apply the frequency and purpose gates |
| Controller for a simple state change | Implicit animation |
| `setState()` on every scroll pixel | Isolated listenable / builder |
| Controller created in `build()` | Lifecycle-managed controller |
| Controller that is never disposed | Dispose in `State.dispose()` |
| `Future.delayed` as an animation engine | Frame-synchronized animation APIs |
| Fixed-duration drag release | Physics + velocity handoff |
| Distance-only dismissal | Velocity or distance threshold |
| Hard stop at a boundary | Rubber-band resistance where appropriate |
| Sliding between peer tabs | No custom transition |
| Screen transition rebuilt in widgets | Navigator route transition |
| Large subtree rebuilt for one animated value | `AnimatedBuilder` / transition widget |
| Infinite animation offscreen | Lifecycle-aware animation / `TickerMode` |
| `Opacity(0)` as a substitute for removing a widget | Remove it when no fade is needed |
| Expensive blur every frame | Static layer or simpler effect |
| `RepaintBoundary` everywhere | Profile and isolate useful repaint work |
| Haptics per frame | One haptic per meaningful commit |
| Haptics as the only feedback | Pair with a visible state change |
| Motion that ignores reduced motion | Respect `MediaQuery.disableAnimations` |
| Gesture-only action | Add accessible alternative actions |
| Judging motion in hot reload | Profile / release build on real hardware |
| Assuming APIs exist in every SDK | Check the project's Flutter version |

---

# Output

Write the implementation first.

Then, in at most a few lines:

- **Gate result** — frequency tier and named purpose. Say what you rejected and why.
- **Ingredients** — Flutter tool, properties, curve or spring, duration, and rebuild strategy.
- **Device feel-check** — name the gestures and conditions to test: flick it, interrupt it mid-flight, reverse it, cancel it, and test on the slowest Android device.

If the correct answer is "this shouldn't animate," say so and stop.

If the motion depends on device behavior that cannot be verified from code, state what must be tested on a physical device.

Do not pad the answer into a report.

## Tone

Opinionated and brief.

Make the call. Explain it in one line. Write the code.

When the honest answer is "this should not animate," do not animate it.

When the honest answer is "this needs a real device before I can tell you if it feels right," say that.
