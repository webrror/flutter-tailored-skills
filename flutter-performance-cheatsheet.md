# Flutter Performance Cheatsheet

A Flutter/Dart adaptation of [Emil Kowalski's Performance Cheatsheet](https://github.com/emilkowalski/skills/blob/main/performance-cheatsheet.md). Use this as a quick diagnostic map, not a substitute for profiling. Flutter's rendering pipeline differs from the browser; verify the actual bottleneck before applying a fix.

## Quick fixes

| Problem | Flutter solution |
|---|---|
| Animation stutters | Prefer animating `Transform`/translation/scale and `Opacity` over layout-affecting properties. For size/position changes that must affect layout, use them intentionally and profile. |
| Long list scrolls slowly | Use lazy builders such as `ListView.builder`, `GridView.builder`, or slivers. Avoid eagerly building large collections. Provide stable keys when item identity/reordering requires them. |
| Blur causes performance issues | Avoid animating large or strong `ImageFilter.blur` / `BackdropFilter` effects. Reduce blur radius and affected area, simplify layers, and profile raster time on target devices. There is no universal safe pixel threshold in Flutter. |
| Motion drops frames | Keep animated rebuild scope small with `AnimatedBuilder`, `ListenableBuilder`, or selective state subscriptions. Prefer transforms/opacity when appropriate. Profile UI vs raster work in DevTools. |
| Random properties animate | Do not animate every property by default. Choose only the properties that communicate the transition; avoid broad implicit animation wrappers when only one value changes. |
| Widget rebuilds every animation frame | Use `AnimatedBuilder` with a `child` for static subtrees, or listen only to the relevant `Listenable`. Keep per-tick work minimal; do not move state into globals just to avoid rebuilds. |
| Element shifts at animation start | Check constraints, fractional translation, alignment, device-pixel ratio, and layout changes. Use `Transform` only when the visual movement should not affect layout. Confirm with slow-motion/frame inspection; Flutter has no direct equivalent to CSS `will-change`. |
| Expensive shadows or clipping | Reduce blur/elevation/clip area; avoid animating complex shadows or clipping unless necessary. Profile raster/compositing costs. Do not remove visual effects blindly if they are central to the design. |
| Too many repaints | Use DevTools repaint diagnostics to find the source. Consider `RepaintBoundary` around independently changing content only when it reduces repaint work; boundaries can also add layer/memory cost. |
| UI isolate is blocked | Move demonstrably CPU-heavy work (large parsing, transforms, computation) to an isolate, e.g. `Isolate.run` or `compute` where supported. Network I/O is already asynchronous and generally does not need an isolate. |
| Image-heavy screen janks | Request appropriately sized images, avoid decoding huge assets for small displays, use caching deliberately, and consider `cacheWidth`/`cacheHeight` for `Image` where suitable. Profile memory and decode costs. |
| Too many items update at once | Reduce the number of animated/rebuilt children, use lazy lists, batch updates, and avoid triggering full-list transitions for routine data refreshes. |
| Layout thrashes during animation | Avoid repeatedly changing layout constraints across large subtrees. Prefer a visual transform when layout need not change; use `AnimatedSize`/position animation only when layout movement is required. |
| Scroll feels inconsistent | Avoid nested unbounded scrollables and expensive work in item builders. Use slivers for complex scrolling screens and verify physics/constraints match the intended platform behavior. |
| App stutters only in debug | Measure in profile mode on a representative device. Debug instrumentation and assertions distort performance. |
| App stutters on lower-end devices | Profile on representative hardware and refresh rates. Identify UI-thread vs raster-thread bottlenecks before simplifying effects or architecture. |
| Repeated timers/listeners waste work | Ensure timers, stream subscriptions, animation controllers, and listeners are owned and disposed correctly. Pause work when offscreen/backgrounded when appropriate. |
| Reduced-motion mode still runs elaborate motion | Respect `MediaQuery.disableAnimationsOf(context)` for nonessential animation; preserve state feedback with a reduced or immediate alternative. |
| A performance fix adds complexity | Prefer the simplest solution that meets measured frame-time goals. Remove speculative caches, custom render objects, and blanket boundaries unless profiling justifies them. |

## Fast profiling checklist

1. Reproduce on a real or representative device in **profile mode**.
2. Use Flutter DevTools Performance view to inspect frame timing and determine whether the **UI** or **raster** work is the bottleneck.
3. Use rebuild/repaint diagnostics to identify the subtree doing excess work.
4. Inspect image decode, shader compilation, clipping, opacity layers, shadows, and blur when raster time is high.
5. Inspect synchronous computation, build/layout work, and broad state updates when UI time is high.
6. Change one thing at a time and re-measure.
7. Verify the visual result, interaction behavior, memory use, and accessibility after optimization.

## Useful commands

```bash
flutter run --profile
flutter analyze
flutter test
```

For targeted performance investigations, use DevTools rather than relying only on average FPS or debug-mode observations.

## References

- [Flutter performance best practices](https://docs.flutter.dev/perf/best-practices)
- [Flutter DevTools Performance view](https://docs.flutter.dev/tools/devtools/performance)
- [Rendering performance](https://docs.flutter.dev/perf/rendering-performance)
- [Isolates and concurrency](https://docs.flutter.dev/perf/isolates)
- [Image class](https://api.flutter.dev/flutter/widgets/Image-class.html)
- [RepaintBoundary](https://api.flutter.dev/flutter/widgets/RepaintBoundary-class.html)
- [Original Performance Cheatsheet](https://github.com/emilkowalski/skills/blob/main/performance-cheatsheet.md)
