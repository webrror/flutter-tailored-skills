# Flutter Animation Recipes

Ready-to-build Flutter implementations for common motion patterns. Includes SDK-native patterns plus package-aware recipes for Motor and Flutter Animate. Start from the recipe, verify the resolved package API, and adapt to the project's architecture and installed SDK.

**Skill:** [SKILL.md](SKILL.md)  
**Compatibility:** Flutter 3.47.9 stable / Dart 3.13.7 and newer stable releases.

Use the motion gates in `SKILL.md` before applying any recipe. These are patterns, not permission to animate every instance.

---


## Package recipes: choose by motion model

These are package-specific patterns, not defaults. Apply the purpose/frequency gates in `SKILL.md` first. Confirm the project's resolved package version before copying code.

### Motor — spring-driven target change

**Purpose:** spatial consistency / state indication. Use when a target should settle with spring behavior or remain responsive to changing targets. Do not add Motor for a simple fade or color change.

```dart
import 'package:flutter/material.dart';
import 'package:motor/motor.dart';

class SpringPositionDemo extends StatelessWidget {
  const SpringPositionDemo({
    super.key,
    required this.offset,
    required this.child,
  });

  final Offset offset;
  final Widget child;

  @override
  Widget build(BuildContext context) {
    return MotionBuilder<Offset>(
      motion: CupertinoMotion.smooth(),
      value: offset,
      converter: OffsetMotionConverter(),
      builder: (context, animatedOffset, child) {
        return Transform.translate(
          offset: animatedOffset,
          child: child,
        );
      },
      child: child,
    );
  }
}
```

Use a documented `CupertinoMotion` or `MaterialSpringMotion` preset that matches the app. The example demonstrates target-driven spring motion; a true drag interaction must additionally track the pointer and preserve release velocity using the package's documented interactive/controller API for the resolved version.

**Check:** change the target repeatedly before settling. Motion should retarget smoothly rather than restart from a stale position. Confirm the converter and constructor signatures against the installed Motor version.

### Motor — multi-dimensional motion

**Purpose:** spatial consistency. Use when multiple dimensions should have independent spring behavior, such as a draggable object settling differently on X and Y.

```dart
MotionBuilder<Offset>(
  motion: MaterialSpringMotion.standardSpatialDefault(),
  value: targetOffset,
  converter: OffsetMotionConverter(),
  builder: (context, offset, child) {
    return Transform.translate(
      offset: offset,
      child: child,
    );
  },
  child: card,
)
```

Do not use this for ordinary layout transitions or for a single scalar that `AnimatedPositioned` / `AnimatedSlide` already handles.

### Flutter Animate — concise entrance

**Purpose:** preventing a jarring change. Use for occasional content appearance, not every rebuild or every row in a long list.

```dart
import 'package:flutter_animate/flutter_animate.dart';

Widget buildCard() {
  return Card(
    child: const Padding(
      padding: EdgeInsets.all(16),
      child: Text('New content'),
    ),
  )
      .animate()
      .fadeIn(
        duration: 180.ms,
        curve: Curves.easeOut,
      )
      .slideY(
        begin: 0.04,
        end: 0,
        duration: 180.ms,
        curve: Curves.easeOut,
      );
}
```

Effects are composed in parallel by default. Keep the chain short and specify timing when it must match project motion tokens.

### Flutter Animate — intentional sequence

**Purpose:** explanation / occasional delight. Use only when the order itself communicates something.

```dart
Widget buildIntro() {
  return const Text('Ready')
      .animate()
      .fadeIn(duration: 160.ms, curve: Curves.easeOut)
      .then(delay: 80.ms)
      .scale(
        begin: const Offset(0.97, 0.97),
        end: const Offset(1, 1),
        duration: 180.ms,
        curve: Curves.easeOut,
      );
}
```

Use `then()` to establish a sequential baseline. Without it, effect chains generally overlap in time. Avoid long choreographies that delay interaction.

### Flutter Animate — staggered group

**Purpose:** preventing a jarring change. Use for a small, finite group in an occasional screen entrance. Avoid on frequently revisited or large virtualized lists.

```dart
Column(
  children: items
      .map<Widget>((item) => ItemTile(item: item))
      .toList()
      .animate(interval: 45.ms)
      .fadeIn(duration: 160.ms, curve: Curves.easeOut)
      .slideY(
        begin: 0.035,
        end: 0,
        duration: 160.ms,
        curve: Curves.easeOut,
      ),
)
```

The list extension returns an `AnimateList`; apply `.animate(interval: ...)` to the widget list, then chain the effects. For dynamic or lazily built collections, verify how the package wraps children and avoid applying this to a large virtualized list. Keep stagger count and total entrance time bounded; preserve immediate access to content and respect reduced motion.

### Reduced motion with package effects

Package animations must obey the app's accessibility policy. For stateful widgets, gate the effect configuration using `MediaQuery.of(context).disableAnimations` and render the child directly or use a brief opacity-only change when motion is disabled.

```dart
Widget buildMotionAwareCard(BuildContext context, Widget child) {
  if (MediaQuery.of(context).disableAnimations) {
    return child;
  }

  return child
      .animate()
      .fadeIn(duration: 160.ms, curve: Curves.easeOut)
      .slideY(begin: 0.03, end: 0);
}
```

Do not rely on package defaults to infer the app's reduced-motion requirements. Also verify `accessibleNavigation` when motion is part of navigation or interaction.

### Package recipe review checklist

- [ ] Package is justified by the motion model, not merely convenience.
- [ ] Resolved version and API signatures are verified.
- [ ] Effect purpose and frequency pass the gates in `SKILL.md`.
- [ ] Curves/durations use project tokens or explicit approved values.
- [ ] Effects do not restart unexpectedly on rebuild.
- [ ] Dynamic/lazy lists are bounded and do not animate every recycled row.
- [ ] Repeating effects stop when offscreen or irrelevant.
- [ ] Reduced motion disables nonessential movement and loops.
- [ ] Gesture-driven Motor motion tracks the finger and hands off velocity.
- [ ] Shader/blur/clip effects are profiled on target hardware when costly.
- [ ] Package use does not duplicate existing Flutter or project motion primitives.

---

## Button press

**Purpose:** feedback. Use for occasional pressables where platform button feedback is insufficient.

Prefer an existing `InkWell`, `FilledButton`, `CupertinoButton`, or project button if it already provides the desired behavior. For custom press scale:

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
    final reduceMotion = MediaQuery.of(context).disableAnimations;

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

For production, ensure custom controls also support semantics, focus, keyboard activation, and a minimum touch target. Use `InkWell` when a Material ripple is desired; do not combine ripple and scale without intent.

---

## Dropdown, popover, menu, select

**Purpose:** spatial consistency. Prefer Flutter's menu/select primitives when they fit.

For a custom anchored overlay, animate from a small scale and opacity, with the transform origin visually near the trigger. Flutter's `ScaleTransition` alignment can approximate the anchor:

```dart
ScaleTransition(
  alignment: Alignment.topRight, // choose the actual trigger-facing corner
  scale: CurvedAnimation(
    parent: controller,
    curve: Curves.easeOut,
  ),
  child: FadeTransition(
    opacity: CurvedAnimation(
      parent: controller,
      curve: Curves.easeOut,
    ),
    child: menu,
  ),
)
```

Initialize the controller at a small nonzero scale (for example `0.95`) and opacity zero; animate to one. Choose alignment from the actual anchor position. Do not hardcode `topRight` if the menu can open from another edge.

For one-off state changes, use `AnimatedScale` and `AnimatedOpacity` rather than a controller. Handle overlay dismissal, focus, semantics, and keyboard navigation separately from motion.

---

## Tooltip

**Purpose:** explanation / feedback. Use native `Tooltip` unless a custom tooltip is required.

A custom tooltip should be subtle and quick:

```dart
FadeTransition(
  opacity: CurvedAnimation(
    parent: controller,
    curve: Curves.easeOut,
  ),
  child: ScaleTransition(
    scale: Tween<double>(begin: 0.97, end: 1).animate(
      CurvedAnimation(parent: controller, curve: Curves.easeOut),
    ),
    child: tooltip,
  ),
)
```

Use a short duration (roughly 125–180ms). Keep hover behavior platform-aware; tooltips must also work for long-press/focus where appropriate. Do not animate repeatedly when moving between adjacent controls if it makes the interface feel slower.

---

## Modal

**Purpose:** spatial consistency. Prefer `showDialog` and platform dialog behavior.

For a custom dialog route, use `Dialog`/`PageRouteBuilder` only when the design needs a custom transition. A restrained scale and fade works:

```dart
FadeTransition(
  opacity: animation,
  child: ScaleTransition(
    scale: Tween<double>(begin: 0.96, end: 1).animate(
      CurvedAnimation(parent: animation, curve: Curves.easeOut),
    ),
    child: dialog,
  ),
)
```

Animate the barrier/backdrop opacity with the route. Keep the dialog centered, support system back, focus traversal, safe areas, and reduced motion. Do not block dismissal until the animation completes.

---

## Drawer / sheet

**Purpose:** spatial consistency. Prefer `Scaffold.drawer`, `showModalBottomSheet`, or `DraggableScrollableSheet` when they fit.

For a custom sheet entrance, use a slide transition and fade, not animated height:

```dart
SlideTransition(
  position: Tween<Offset>(
    begin: const Offset(0, 1),
    end: Offset.zero,
  ).animate(
    CurvedAnimation(parent: controller, curve: Curves.easeOut),
  ),
  child: sheet,
)
```

Use route/platform behavior when available. If the sheet is draggable, the drag recipe below applies: track the finger directly and settle with physics. Respect safe areas, keyboard insets, and accessible close actions.

---

## Toast / snackbar

**Purpose:** feedback. Prefer `ScaffoldMessenger` and `SnackBar` unless a custom overlay is required.

For a custom toast, animate opacity and a small vertical offset:

```dart
FadeTransition(
  opacity: CurvedAnimation(parent: controller, curve: Curves.easeOut),
  child: SlideTransition(
    position: Tween<Offset>(
      begin: const Offset(0, 0.08),
      end: Offset.zero,
    ).animate(
      CurvedAnimation(parent: controller, curve: Curves.easeOut),
    ),
    child: toast,
  ),
)
```

Keep the entrance short and ensure stacked toasts do not obscure content or jump unpredictably. The timeout is a visibility policy, not an animation engine. Support reduced motion and dismiss actions.

---

## Accordion / collapse

**Purpose:** preventing a jarring change. Prefer `ExpansionTile`, `AnimatedSize`, or a dedicated expansion widget.

```dart
AnimatedSize(
  duration: MediaQuery.of(context).disableAnimations
      ? Duration.zero
      : const Duration(milliseconds: 200),
  curve: Curves.easeOut,
  alignment: Alignment.topCenter,
  child: isExpanded ? content : const SizedBox.shrink(),
)
```

Avoid animating a guessed fixed height. `AnimatedSize` follows actual content, including text scaling. Keep the duration short because this performs layout during the transition. If the child is expensive, profile it and isolate work where useful.

---

## Stagger a group entrance

**Purpose:** explanation / delight. Use only for occasional content the user sees as a group, not frequently visited or continuously scrolling lists.

A simple reusable pattern can stagger start times with `Future.delayed`, but do not use timers as the animation engine. The actual motion should remain frame-synchronized and lifecycle-safe. For a small fixed group, use `Interval` on one controller:

```dart
Animation<double> itemOpacity(
  AnimationController controller,
  int index,
  int itemCount,
) {
  final start = (index / itemCount).clamp(0.0, 0.8);
  final end = (start + 0.35).clamp(0.0, 1.0);

  return CurvedAnimation(
    parent: controller,
    curve: Interval(start, end, curve: Curves.easeOut),
  );
}
```

Use each item's animation in a `FadeTransition` and a small `SlideTransition`. Cap the group size and make items interactive immediately; the entrance must not block actions. Skip the stagger when reduced motion is enabled.

---

## Hold to confirm

**Purpose:** preventing an accidental destructive action. Prefer a clear confirmation dialog for high-consequence actions; use hold-to-confirm only when it genuinely improves the interaction.

Use a controller to fill a progress indicator while pressed. Start on press-down, reverse/reset on cancellation, and commit only after completion:

```dart
GestureDetector(
  onTapDown: (_) => controller.forward(),
  onTapUp: (_) {
    if (controller.isCompleted) {
      onConfirmed();
    } else {
      controller.reverse();
    }
  },
  onTapCancel: controller.reset,
  child: Stack(
    alignment: Alignment.center,
    children: [
      // Background/control surface.
      AnimatedBuilder(
        animation: controller,
        builder: (context, child) => Align(
          alignment: Alignment.centerLeft,
          child: FractionallySizedBox(
            widthFactor: controller.value,
            child: child,
          ),
        ),
        child: const ColoredBox(color: Colors.white24),
      ),
      label,
    ],
  ),
)
```

Use a linear curve for the fill because it represents progress. Ensure cancellation is reliable, the control has an accessible alternative, and destructive actions are not triggered by an accidental short press.

---

## Tab indicator

**Purpose:** state indication. Tabs are peers; animate the indicator, not the entire page.

For equal-width tabs:

```dart
AnimatedAlign(
  alignment: selectedIndex == 0
      ? Alignment.centerLeft
      : selectedIndex == 1
          ? Alignment.center
          : Alignment.centerRight,
  duration: const Duration(milliseconds: 180),
  curve: Curves.easeInOut,
  child: FractionallySizedBox(
    widthFactor: 1 / tabCount,
    child: indicator,
  ),
)
```

For variable-width tabs, use measured positions with `AnimatedPositioned` or a custom painter. Avoid hardcoded geometry when text scaling or localization changes widths. Keep the tab content switch immediate or use the platform's expected behavior; do not add a sliding page transition by default.

---

## Scroll reveal

**Purpose:** explanation / delight. Marketing or onboarding surfaces only; not routine functional UI.

Use `ScrollController` or `ScrollNotification` to detect visibility, then trigger a one-time fade/short slide. Avoid `setState()` on every scroll pixel; isolate the reveal state to the relevant child.

- Trigger once, not on every scroll-by.
- Do not delay interaction while the reveal plays.
- Skip or simplify under reduced motion.
- Avoid animating large lists of children.
- Prefer slivers and existing scroll behavior for functional content.

For standard collapsing headers, use `SliverAppBar` instead of a custom scroll reveal.

---

## Drag to dismiss

**Purpose:** spatial consistency / feedback. Use `Dismissible` for ordinary swipe-to-dismiss. Build custom physics only when the interaction needs specialized behavior.

For custom drag:
- Track pointer movement directly.
- Do not ease while the finger is down.
- Use `DragEndDetails.velocity` to determine whether a flick is sufficient.
- Dismiss on distance **or** velocity.
- Apply resistance beyond boundaries rather than a hard wall.
- Preserve release velocity when settling.
- Handle gesture cancellation and nested scrolling.
- Provide an accessible dismiss action.

A physics-based settle should use a `SpringSimulation` or an appropriate controller simulation. Do not use a fixed-duration animation after a drag if it discards the user's momentum.

---

## Masked / clipped reveal

**Purpose:** explanation or state indication. Flutter does not have CSS `clip-path`; use `ClipRect`, `Align(heightFactor/widthFactor)`, or a custom clipper only when the reveal itself is meaningful.

For a simple vertical reveal, an `Align` with animated `heightFactor` can work, but it performs layout. For large content, prefer a fade/slide or `AnimatedSize` if the layout change is the actual state transition.

```dart
AnimatedAlign(
  duration: const Duration(milliseconds: 200),
  curve: Curves.easeOut,
  alignment: Alignment.topCenter,
  heightFactor: isVisible ? 1 : 0,
  child: child,
)
```

Do not introduce a custom `CustomClipper` for a standard fade or slide. If using a custom clipper, implement `shouldReclip` correctly and profile.

---

## Keyboard-synced UI

**Purpose:** preventing a jarring change. Flutter already reports keyboard insets; follow those instead of guessing keyboard duration.

```dart
AnimatedPadding(
  duration: MediaQuery.of(context).disableAnimations
      ? Duration.zero
      : const Duration(milliseconds: 180),
  curve: Curves.easeOut,
  padding: EdgeInsets.only(
    bottom: MediaQuery.of(context).viewInsets.bottom,
  ),
  child: content,
)
```

Use `Scaffold`'s resize behavior when it already solves the layout. For precise keyboard-following motion, use platform-aware keyboard inset/metrics rather than an arbitrary timer. Test on iOS and Android, with hardware keyboards and large text.

---

## Screen transitions

**Purpose:** spatial consistency. Prefer platform-native route transitions.

Use `Navigator` and the app's route conventions. Use `PageRouteBuilder` only when a custom transition is required. Keep transitions consistent, respect reduced motion, and do not slide between peer tabs. Use `Hero` only for a genuine shared-element relationship.

A route transition must support system back, interruption, and lifecycle behavior. Avoid rebuilding the transition in the page body.

---

## Pull to refresh

**Purpose:** feedback. Prefer `RefreshIndicator` or the platform's native refresh control.

Do not hand-roll the gesture unless the app has a signature interaction that cannot be achieved with the standard widget. Keep the refresh indicator's motion tied to scroll physics and actual refresh state; do not fake completion with a timer.

---

## Progress animation

**Purpose:** state indication. Use `LinearProgressIndicator` or `CircularProgressIndicator` for real progress.

- Use determinate progress when the value is known.
- Use indeterminate progress when it is not.
- Do not animate fake progress to make a slow operation appear faster.
- Use `TweenAnimationBuilder<double>` for a meaningful change between known values.
- Use linear motion for continuous progress.
- Respect reduced motion without hiding that work is ongoing.

---

## Haptics at a detent

**Purpose:** feedback. Trigger one haptic when the interaction crosses or commits to a meaningful detent—not on every frame.

```dart
// Call from the gesture/controller logic when a detent is committed.
HapticFeedback.selectionClick();
```

Pair it with visible snapping/selection. Avoid firing repeatedly if a noisy gesture oscillates around the threshold; track the previous detent and emit only when the selected detent changes. Test on real devices because haptic strength varies by platform and hardware.

---

## Device verification checklist

Before calling a recipe finished:

- Test on a physical device in profile or release mode.
- Flick, interrupt mid-flight, reverse direction, and cancel the gesture.
- Test at minimum and maximum text scale.
- Test reduced motion / `disableAnimations`.
- Check touch targets, semantics, focus, and keyboard activation.
- Check nested scrolling and gesture-arena conflicts.
- Test on the slowest supported Android and at least one iOS device if applicable.
- Inspect UI/raster frame timing in Flutter DevTools.
