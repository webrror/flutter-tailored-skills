# Flutter Animation Recipes

Ready-to-adapt implementations for common Flutter interactions. Start from the recipe, then adapt it to the product's design, accessibility needs, and minimum supported Flutter SDK.

**Compatibility:** Flutter 3.47.9 stable / Dart 3.13.7 and newer stable releases. Prefer SDK APIs and verify any newer API against the project's installed SDK. These recipes use Flutter SDK APIs unless explicitly noted.

The principles follow the companion `SKILL.md`: animate only with a purpose, choose the cheapest suitable primitive, preserve gesture continuity, respect reduced motion, and verify on a physical device in profile/release mode.

---

## Shared conventions

- Prefer implicit animation widgets for a state change without gesture-controlled progress.
- Use `AnimationController` only when you need explicit lifecycle, sequencing, interruption, or physics.
- Use Flutter's native widgets for navigation, sheets, scroll, and dismissal before hand-rolling them.
- Keep frame-driven work local. Prefer `AnimatedBuilder` with a stable `child`, transition widgets, or `CustomPainter(repaint: ...)` over rebuilding a screen from an animation listener.
- Respect `MediaQuery.disableAnimations` for custom motion. Platform widgets may already adapt their behavior.
- Dispose every controller. Create controllers in `initState`, never in `build`.
- Use `const` where possible and profile before adding `RepaintBoundary`.

Common imports:

```dart
import 'dart:math' as math;
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
```

A helper for reduced motion:

```dart
bool reduceMotionOf(BuildContext context) =>
    MediaQuery.of(context).disableAnimations;
```

---

## Press feedback

**Gate:** frequent interaction; feedback only. Keep it subtle (about 120ms, scale 0.97). Prefer the platform button if it already provides the desired response.

This wrapper demonstrates custom scale feedback while preserving a semantic button and a minimum target. For production, prefer `FilledButton`, `IconButton`, `InkWell`, or `CupertinoButton` when their built-in behavior fits.

```dart
class PressScale extends StatefulWidget {
  const PressScale({
    super.key,
    required this.onPressed,
    required this.child,
    this.enabled = true,
  });

  final VoidCallback onPressed;
  final Widget child;
  final bool enabled;

  @override
  State<PressScale> createState() => _PressScaleState();
}

class _PressScaleState extends State<PressScale> {
  bool _pressed = false;

  void _setPressed(bool value) {
    if (_pressed != value) setState(() => _pressed = value);
  }

  @override
  Widget build(BuildContext context) {
    final reduceMotion = reduceMotionOf(context);

    return Semantics(
      button: true,
      enabled: widget.enabled,
      child: GestureDetector(
        behavior: HitTestBehavior.opaque,
        onTapDown: widget.enabled ? (_) => _setPressed(true) : null,
        onTapUp: widget.enabled ? (_) => _setPressed(false) : null,
        onTapCancel: () => _setPressed(false),
        onTap: widget.enabled ? widget.onPressed : null,
        child: AnimatedScale(
          scale: _pressed && !reduceMotion ? 0.97 : 1,
          duration: const Duration(milliseconds: 120),
          curve: Curves.easeOut,
          child: ConstrainedBox(
            constraints: const BoxConstraints(minHeight: 48, minWidth: 48),
            child: widget.child,
          ),
        ),
      ),
    );
  }
}
```

Notes:
- `setState` happens only on press-in/out, not every frame; `AnimatedScale` owns the transition.
- If this is nested inside another gesture surface, verify the gesture arena and semantics.
- Keep the visual child's intended size; use padding/hit regions for touch targets rather than enlarging artwork.

---

## Bottom sheet you can drag to dismiss

**First choice:** use `showModalBottomSheet` and `DraggableScrollableSheet` when their built-in behavior fits. Do not build a custom sheet just to reproduce standard platform behavior.

For a scrollable sheet with draggable extents:

```dart
Future<void> showDraggableContentSheet(BuildContext context) {
  return showModalBottomSheet<void>(
    context: context,
    isScrollControlled: true,
    useSafeArea: true,
    showDragHandle: true,
    builder: (context) {
      return DraggableScrollableSheet(
        expand: false,
        initialChildSize: 0.65,
        minChildSize: 0.35,
        maxChildSize: 0.92,
        builder: (context, scrollController) {
          return ListView(
            controller: scrollController,
            children: const [
              ListTile(title: Text('Sheet content')),
              // Put the actual content here.
            ],
          );
        },
      );
    },
  );
}
```

If the sheet must be a custom in-screen surface, the essential interaction is:

1. Capture the current offset when the drag starts, including when interrupting an animation.
2. Follow the finger directly during the drag; do not apply an easing curve to finger movement.
3. On release, dismiss when either the distance threshold or velocity threshold is crossed.
4. Otherwise settle back using a spring and pass the release velocity into the simulation.
5. Provide a visible close action and respect reduced motion.

A compact physics pattern for a custom `State` with a `AnimationController` and a `double _offset` is:

```dart
// Controller is created in initState with vsync: this and disposed in dispose.
// _offset is the current logical drag offset; render it with Transform.translate.

void settleBack(double velocityPxPerSecond) {
  if (reduceMotionOf(context)) {
    setState(() => _offset = 0);
    return;
  }

  final simulation = SpringSimulation(
    const SpringDescription(mass: 1, stiffness: 420, damping: 38),
    _offset,
    0,
    velocityPxPerSecond,
  );
  _controller
    ..stop()
    ..reset();
  _controller.animateWith(simulation);
}
```

For a complete custom sheet, make the controller's animation value the source of truth for the rendered offset (rather than calling `setState` every tick), and ensure the simulation's bounds and dismissal route are handled correctly. Do not copy this partial physics fragment as a standalone widget.

Feel-check: grab it mid-settle, drag slowly, flick it a short distance, reverse direction, and test whether the scrollable content still scrolls naturally.

---

## Swipe to delete a row

**First choice:** Flutter's `Dismissible` handles ordinary swipe-to-dismiss, including gesture recognition and dismissal animation. Use it instead of custom gesture physics unless the interaction is materially different.

```dart
Dismissible(
  key: ValueKey(item.id),
  direction: DismissDirection.endToStart,
  background: Container(
    alignment: Alignment.centerRight,
    padding: const EdgeInsets.symmetric(horizontal: 20),
    color: Theme.of(context).colorScheme.error,
    child: Icon(
      Icons.delete_outline,
      color: Theme.of(context).colorScheme.onError,
    ),
  ),
  onDismissed: (_) => onDelete(item),
  child: ListTile(title: Text(item.title)),
)
```

The list owns the gap closing: remove the item from the backing collection in `onDismissed` and let the list rebuild. Use stable keys. Do not also animate the row's height manually unless the result has been carefully coordinated.

For swipe-to-reveal actions rather than commit-to-delete, use a maintained Flutter package only if the project needs that pattern and the package is compatible with its SDK; otherwise implement a focused gesture surface with explicit horizontal-axis intent and careful scroll coordination.

Feel-check: vertical scrolling must not be stolen by the horizontal dismiss gesture. Test slow drags, fast flicks, cancellation, and accessibility alternatives for the delete action.

---

## Collapsing header on scroll

**First choice:** use `SliverAppBar` / `FlexibleSpaceBar`. Flutter already coordinates the header with the scroll position.

```dart
CustomScrollView(
  slivers: [
    SliverAppBar(
      pinned: true,
      expandedHeight: 220,
      flexibleSpace: FlexibleSpaceBar(
        title: const Text('Page title'),
        background: Image.asset(
          'assets/header.jpg',
          fit: BoxFit.cover,
        ),
      ),
    ),
    SliverList(
      delegate: SliverChildBuilderDelegate(
        (context, index) => ListTile(title: Text('Row $index')),
        childCount: 50,
      ),
    ),
  ],
)
```

For a custom scroll-linked effect, derive the visual from a `ScrollController` or `ScrollNotification` and clamp the interpolation. Avoid calling `setState` on the whole screen for every scroll pixel. Prefer transforms and opacity for decorative content; do not animate the height of a large header every frame unless profiling shows it is safe.

Feel-check: fling to the top and bottom, reverse direction, and verify title readability and scroll performance on a lower-end device.

---

## List entrances and insertions

**Gate:** only animate content the user is waiting for or a rare, meaningful change. Do not animate every recycled row whenever it scrolls into view.

For actual insert/remove operations, use `AnimatedList` or `SliverAnimatedList`:

```dart
class AnimatedItems extends StatefulWidget {
  const AnimatedItems({super.key});

  @override
  State<AnimatedItems> createState() => _AnimatedItemsState();
}

class _AnimatedItemsState extends State<AnimatedItems> {
  final _items = <String>['One', 'Two', 'Three'];
  final _listKey = GlobalKey<AnimatedListState>();

  void _insert() {
    const index = 0;
    _items.insert(index, 'New item');
    _listKey.currentState?.insertItem(
      index,
      duration: const Duration(milliseconds: 220),
    );
  }

  void _remove(int index) {
    final removed = _items.removeAt(index);
    _listKey.currentState?.removeItem(
      index,
      (context, animation) => SizeTransition(
        sizeFactor: animation,
        child: FadeTransition(
          opacity: animation,
          child: ListTile(title: Text(removed)),
        ),
      ),
      duration: const Duration(milliseconds: 180),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        ElevatedButton(onPressed: _insert, child: const Text('Insert')),
        Expanded(
          child: AnimatedList(
            key: _listKey,
            initialItemCount: _items.length,
            itemBuilder: (context, index, animation) {
              return FadeTransition(
                opacity: animation,
                child: SizeTransition(
                  sizeFactor: animation,
                  child: ListTile(
                    title: Text(_items[index]),
                    onTap: () => _remove(index),
                  ),
                ),
              );
            },
          ),
        ),
      ],
    );
  }
}
```

For a one-time small group of newly revealed elements, a short stagger can be appropriate. Keep delays around 30–60ms and the total sequence short. For long or virtualized lists, do not use entrance animation as a scroll effect; animate the container once or animate only genuine insertions.

---

## Keyboard-synced UI

Flutter already exposes keyboard insets through `MediaQuery.viewInsets`. Prefer normal scaffold resizing and layout that follows the actual inset updates. Do not guess the keyboard's duration with a separate timer; it can drift from the platform keyboard curve.

```dart
Scaffold(
  resizeToAvoidBottomInset: true,
  body: SafeArea(
    child: Column(
      children: [
        const Expanded(child: FormContent()),
        Padding(
          padding: EdgeInsets.only(
            bottom: MediaQuery.viewInsetsOf(context).bottom,
          ),
          child: const ComposerBar(),
        ),
      ],
    ),
  ),
)
```

Use this pattern only when the layout needs explicit inset handling. In many screens, `Scaffold` resizing plus a scrollable form is sufficient. Avoid adding `AnimatedPadding` on top of a system-driven inset unless the visual has been checked for double animation/lag.

Feel-check: open and close the keyboard repeatedly, switch input fields, test landscape, and verify the bottom action remains reachable without a visible lag.

---

## Tab / segmented indicator

**First choice:** use `TabBar` and its built-in indicator. It already animates the indicator and respects tab layout.

```dart
DefaultTabController(
  length: 3,
  child: Column(
    children: const [
      TabBar(
        tabs: [
          Tab(text: 'Today'),
          Tab(text: 'Week'),
          Tab(text: 'Month'),
        ],
      ),
      Expanded(
        child: TabBarView(
          children: [
            Center(child: Text('Today content')),
            Center(child: Text('Week content')),
            Center(child: Text('Month content')),
          ],
        ),
      ),
    ],
  ),
)
```

For a custom segmented control, measure each segment once and animate an absolutely positioned indicator using `AnimatedPositioned` / `AnimatedContainer`. Do not rebuild the entire control every frame. Avoid sliding entire peer tabs unless the interaction explicitly represents a spatial sequence.

Feel-check: change selections quickly in succession and verify the indicator retargets smoothly without lagging behind the selected state.

---

## Screen transitions

Use Flutter's route system. Do not rebuild a full-screen transition with a controller inside the page.

Prefer platform route behavior:

```dart
Navigator.of(context).push(
  MaterialPageRoute<void>(
    builder: (_) => const DetailsPage(),
  ),
);
```

For iOS-style navigation, use `CupertinoPageRoute`. Use `PageRouteBuilder` only when the design truly requires a custom transition, and keep the transition consistent across the app.

A restrained custom fade/slide route example:

```dart
class FadeSlideRoute<T> extends PageRouteBuilder<T> {
  FadeSlideRoute({required Widget page})
      : super(
          pageBuilder: (context, animation, secondaryAnimation) => page,
          transitionDuration: const Duration(milliseconds: 220),
          reverseTransitionDuration: const Duration(milliseconds: 180),
          transitionsBuilder: (context, animation, secondaryAnimation, child) {
            if (MediaQuery.of(context).disableAnimations) {
              return FadeTransition(opacity: animation, child: child);
            }
            final curved = CurvedAnimation(
              parent: animation,
              curve: Curves.easeOut,
              reverseCurve: Curves.easeInOut,
            );
            return FadeTransition(
              opacity: curved,
              child: SlideTransition(
                position: Tween<Offset>(
                  begin: const Offset(0.03, 0),
                  end: Offset.zero,
                ).animate(curved),
                child: child,
              ),
            );
          },
        );
}
```

Use `Hero` only when a shared element genuinely connects the two routes. Preserve the platform back gesture and route semantics. Do not custom-slide between peer tabs.

---

## Toast / transient feedback

**First choice:** use `ScaffoldMessenger` and `SnackBar` for standard app feedback. It handles queueing, accessibility, and scaffold placement.

```dart
ScaffoldMessenger.of(context)
  ..hideCurrentSnackBar()
  ..showSnackBar(
    const SnackBar(
      content: Text('Saved'),
      duration: Duration(seconds: 2),
      behavior: SnackBarBehavior.floating,
    ),
  );
```

Use a custom `OverlayEntry` only for a product-specific toast that cannot be expressed with `SnackBar`. For custom motion:

- Enter and exit from the same direction.
- Keep each transition quick (roughly 180–250ms).
- Make exit slightly faster than entry.
- Respect safe areas and text scaling.
- Avoid blocking taps unless the toast is intentionally modal.
- Ensure the message is exposed to accessibility services.

Do not build a custom toast system just to change a color or corner radius.

---

## Threshold haptics

Fire haptics when the user crosses a meaningful threshold, not every frame and not when the animation eventually finishes.

```dart
class ThresholdDrag extends StatefulWidget {
  const ThresholdDrag({super.key});

  @override
  State<ThresholdDrag> createState() => _ThresholdDragState();
}

class _ThresholdDragState extends State<ThresholdDrag> {
  double _distance = 0;
  bool _armed = false;
  static const _threshold = 96.0;

  void _update(double next) {
    final nowArmed = next >= _threshold;
    if (nowArmed != _armed) {
      _armed = nowArmed;
      HapticFeedback.selectionClick();
    }
    setState(() => _distance = next);
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onVerticalDragUpdate: (details) {
        _update(math.max(0, _distance + details.delta.dy));
      },
      onVerticalDragEnd: (details) {
        // Commit or reset here; do not trigger haptics per update.
        setState(() {
          _distance = 0;
          _armed = false;
        });
      },
      child: Transform.translate(
        offset: Offset(0, _distance),
        child: const Card(
          child: SizedBox(
            height: 100,
            child: Center(child: Text('Drag down')),
          ),
        ),
      ),
    );
  }
}
```

This is a threshold pattern, not a complete production drag component. In a real implementation, isolate continuous drag state from unrelated widgets, handle cancellation, use velocity for settle/dismiss decisions, and provide an accessible action that does not require dragging.

Haptics may be silent or vary by platform. Always pair them with visible feedback.

---

## Pull to refresh

Use `RefreshIndicator` for standard pull-to-refresh. It already handles the gesture threshold, indicator, and scroll integration.

```dart
RefreshIndicator(
  onRefresh: reload,
  child: ListView.builder(
    physics: const AlwaysScrollableScrollPhysics(),
    itemCount: items.length,
    itemBuilder: (context, index) => ListTile(
      title: Text(items[index].title),
    ),
  ),
)
```

Only hand-roll pull-to-refresh when the interaction is a signature part of the product. If custom, define an arming threshold, trigger haptics once when crossing it, preserve scroll physics, support cancellation, and avoid polling from build.

---

## Repeating / progress animation

For a progress value that changes because of app state, use `TweenAnimationBuilder` or an implicit widget. For a genuinely continuous progress indicator, use Flutter's built-in progress widgets where possible.

```dart
TweenAnimationBuilder<double>(
  tween: Tween<double>(begin: 0, end: progress),
  duration: const Duration(milliseconds: 180),
  curve: Curves.easeOut,
  builder: (context, value, child) {
    return LinearProgressIndicator(value: value);
  },
)
```

Do not animate progress if it needs to communicate exact live values or if the transition would misrepresent the underlying operation. Avoid infinite decorative loops in frequently visited screens.

---

## Device verification checklist

Run in profile or release mode on a physical device, including the slowest supported Android device.

- Interrupt the animation mid-flight and reverse it.
- Flick, drag slowly, cancel, and re-grab during settling.
- Verify velocity handoff and dismissal thresholds.
- Scroll while gestures are nested; confirm the correct recognizer wins.
- Test reduced motion and large text.
- Test keyboard open/close and safe areas.
- Check UI and raster frame times in Flutter DevTools.
- Watch for unnecessary rebuilds, layout passes, clipping, opacity layers, and expensive paint effects.
- Confirm haptics happen once at the causal moment and are not required for understanding.
- Check the behavior after the app is backgrounded and resumed.

A recipe is a starting point, not proof of performance. The final feel must be judged on-device.
