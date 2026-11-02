# Flutter UI Cheatsheet

A Flutter adaptation of [Emil Kowalski's UI Cheatsheet](https://github.com/emilkowalski/skills/blob/main/ui-cheatsheet.md).

| UI problem | Flutter solution |
|---|---|
| Old timestamps read “340 days ago” | Use relative time for recent events, then switch to an absolute date after about a week. Use `DateFormat` from `intl` for locale-aware formatting; choose the cutoff based on product context. |
| Numbers jitter when they change | Use tabular numerals where the font supports them: `TextStyle(fontFeatures: const [FontFeature.tabularFigures()])` (`dart:ui`). Consider a fixed/minimum width for rapidly changing counters. |
| Large headings look visually loose | Tune `TextStyle(letterSpacing: ...)` relative to the font size (for example, a subtle negative value for large display text). Check the actual typeface and platform rendering; do not apply tracking globally. |
| Body text feels too wide or awkward | Constrain reading width with `ConstrainedBox(maxWidth: ...)` or `Align`/`Center`. A rough target is around 60–70 characters per line; Flutter has no CSS `ch` unit, so validate with real text and responsive layouts. |
| Button feels unresponsive | Provide immediate pressed feedback using `InkWell`/`InkResponse` and, when appropriate, a subtle `AnimatedScale` (e.g. `0.97` while pressed). Keep hit targets stable and preserve semantics. |
| Avatar becomes stretched or oval | Give it a square constraint (`SizedBox.square` or `AspectRatio(1)`) and clip with `ClipOval`; use `BoxFit.cover`. Prevent flex layouts from shrinking it unexpectedly with `Flexible`/`Expanded` applied to adjacent content instead. |
| Gradient fade has a visible edge | Add intermediate `Color` stops to shape the fade, and ensure the final stop reaches the intended background color. Use `LinearGradient`/`RadialGradient`; preview on the actual background and in light/dark themes. |
| Transition feels visually harsh | Prefer a small, purposeful combination of opacity and translation/scale. Avoid adding blur by default; if blur is truly needed, use `ImageFiltered` or `BackdropFilter` carefully and profile it because it can be expensive. |
| Dropdown feels sluggish | Use a responsive ease-out curve such as `Curves.easeOutCubic` and keep ordinary menu transitions short (often roughly 150–250ms). Avoid `Curves.easeIn` for entrances. Respect platform conventions and reduced-motion preferences. |
| Popover grows from the wrong spot | Set the transform/scale alignment toward the trigger (`Transform.alignment`, `ScaleTransition.alignment`, or `AnimatedScale.alignment`). Position the surface relative to its anchor; centered dialogs can animate from center. |
| Count reads “1 members” | Use plural-aware localization (`Intl.plural` / ARB plural resources) instead of string concatenation. Include zero, one, and other forms as required by the locale. |
| Text or controls overflow on small screens | Design from constraints: `LayoutBuilder`, `Wrap`, `Flexible`, `Expanded`, scrollable layouts, and `TextOverflow` only where truncation is appropriate. Test narrow widths and large text scaling. |
| UI ignores the system's reduced-motion preference | Check `MediaQuery.disableAnimationsOf(context)` for custom nonessential motion and provide a reduced/no-motion alternative while preserving state feedback. Explicit controllers should also use appropriate `AnimationBehavior`. |
| UI looks different with large accessibility text | Support text scaling; avoid fixed-height text containers and hardcoded line assumptions. Test with large text, bold text, and localization. |
| A card looks flat or outlined too harshly | Prefer hierarchy through surface color, subtle elevation/shadow, and spacing before adding strong borders. Use `Theme`/`ColorScheme` tokens so it works in light and dark themes. |
| List scrolling stutters | Use lazy lists (`ListView.builder`, slivers), avoid expensive work in `build`, keep animated rebuild scopes small, and profile in profile mode with DevTools before adding optimizations. |
| UI looks good only with short sample content | Test long names, large counts, missing values, long localized labels, emoji, and right-to-left text. Use realistic worst-case data before finalizing. |

## Quick Flutter snippets

### Tabular numerals

```dart
import 'dart:ui' show FontFeature;

Text(
  count.toString(),
  style: const TextStyle(
    fontFeatures: [FontFeature.tabularFigures()],
  ),
)
```

### Press feedback

```dart
class PressScale extends StatefulWidget {
  const PressScale({
    required this.child,
    required this.onPressed,
    super.key,
  });

  final Widget child;
  final VoidCallback onPressed;

  @override
  State<PressScale> createState() => _PressScaleState();
}

class _PressScaleState extends State<PressScale> {
  bool _pressed = false;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: widget.onPressed,
      onTapDown: (_) => setState(() => _pressed = true),
      onTapUp: (_) => setState(() => _pressed = false),
      onTapCancel: () => setState(() => _pressed = false),
      child: AnimatedScale(
        scale: _pressed ? 0.97 : 1,
        duration: const Duration(milliseconds: 120),
        curve: Curves.easeOutCubic,
        child: widget.child,
      ),
    );
  }
}
```

For production controls, prefer `InkWell`/`InkResponse` or a button widget when Material feedback, focus, keyboard behavior, and semantics are needed. Avoid replacing those behaviors with a bare `GestureDetector`.

### Reduced motion

```dart
final reduceMotion = MediaQuery.disableAnimationsOf(context);

AnimatedScale(
  scale: selected ? 1.04 : 1,
  duration: reduceMotion
      ? Duration.zero
      : const Duration(milliseconds: 180),
  curve: Curves.easeOutCubic,
  child: child,
)
```

### Anchored scale

```dart
ScaleTransition(
  alignment: Alignment.topRight, // choose the actual trigger-facing edge
  scale: animation,
  child: menu,
)
```

## Flutter references

- [Layout constraints](https://docs.flutter.dev/ui/layout/constraints)
- [Typography](https://docs.flutter.dev/ui/design/text/typography)
- [`AnimatedScale`](https://api.flutter.dev/flutter/widgets/AnimatedScale-class.html)
- [`MediaQuery.disableAnimationsOf`](https://api.flutter.dev/flutter/widgets/MediaQuery/disableAnimationsOf.html)
- [Animation curves](https://api.flutter.dev/flutter/animation/Curves-class.html)
- [Original UI Cheatsheet](https://github.com/emilkowalski/skills/blob/main/ui-cheatsheet.md)
