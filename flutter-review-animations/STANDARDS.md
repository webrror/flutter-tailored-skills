# Flutter Animation Review Standards

Reference for precise review values and rationale. These are starting points, not universal laws; platform conventions, interaction size, user control, and product character can justify exceptions.

## Should it animate? (frequency guide)

| Frequency | Decision |
| --- | --- |
| Keyboard shortcuts / command palette / 100+ times per day | No animation, unless essential feedback is immediate and nearly imperceptible |
| Tens of times per day | Remove or drastically reduce |
| Occasional (dialogs, drawers, toasts) | Standard transition |
| Rare / first-time (onboarding, success celebration) | Restrained delight can be appropriate |

Valid purposes: spatial continuity, state indication, feedback, explanation, preventing a jarring change. “Looks cool” alone is not enough for frequently used UI.

## Easing in Flutter

- Enter/exit and system response: `Curves.easeOut`, `Curves.easeOutCubic`, or a verified custom curve.
- On-screen movement/morphing: `Curves.easeInOut` or a curve suited to the path.
- Hover/color feedback: a short, subtle curve; ensure touch has equivalent feedback.
- Constant motion: linear progression where mathematically appropriate.
- Default for ordinary UI: responsive ease-out.

Avoid `Curves.easeIn` for ordinary UI entrances/responses because it starts slowly. A deliberate exit or physical simulation may be an exception.

Custom cubic curves can translate from CSS cubic-bezier values with `Cubic(x1, y1, x2, y2)`, but review the visual result rather than assuming a direct perceptual match:
- Strong ease-out reference: `Cubic(0.23, 1.0, 0.32, 1.0)`
- Strong ease-in-out reference: `Cubic(0.77, 0.0, 0.175, 1.0)`
- Drawer-like reference: `Cubic(0.32, 0.72, 0.0, 1.0)`

## Duration guide

| Element | Typical duration |
| --- | --- |
| Button press feedback | 100–160ms |
| Tooltip / small popover | 125–200ms |
| Dropdown / select | 150–250ms |
| Dialog / drawer | 200–300ms for most UI; longer only when justified |
| Explanatory / onboarding motion | May be longer if it remains skippable and non-blocking |

Rule of thumb: ordinary UI transitions should generally stay under 300ms. Faster is not always better; choose the shortest duration that communicates the state clearly. Platform-native transitions may be appropriate.

## Physicality and origin

- Avoid scale from zero for ordinary UI. If scale is useful, begin around 0.9–0.97 with opacity, or use a directional slide.
- Trigger-anchored menus/popovers should originate near their anchor/edge. Flutter's `Transform.scale` alignment and `Alignment` should reflect the actual geometry.
- Centered dialogs may scale/fade from center.
- Press feedback is subtle: scale around 0.97–0.99, typically 100–160ms; avoid distorting text or causing layout changes.
- Do not animate a full-screen surface from an arbitrary origin without a spatial explanation.

## Springs and physics

Use spring/physics simulations for drag, fling, momentum, and interruptible gesture-driven motion—not as decoration everywhere.

Flutter options include `SpringSimulation`, `ScrollSpringSimulation`, `AnimationController.animateWith`, `animateTo`, and package-specific spring APIs. Verify parameters against the installed Flutter/package version. Keep overshoot restrained in ordinary UI; stronger bounce is reserved for playful or direct-manipulation interactions.

For controller-driven motion, preserve current value/velocity when reversing or retargeting where possible. Avoid blindly calling `forward(from: 0)` on rapid repeated input.

## Interruptibility and state

- Implicit animations such as `AnimatedContainer` or `AnimatedOpacity` retarget when their target values change; use them when state-driven interpolation is sufficient.
- Explicit controllers are appropriate for sequencing, gestures, custom timelines, and fine control. Ensure rapid changes do not visibly restart from stale initial values.
- Handle `AnimationStatus`, cancellation, reverse, and disposal correctly.
- Avoid callbacks that fire multiple times, after a widget is disposed, or after a transition has been superseded.
- Keep semantics and hit testing consistent while elements enter/exit. Consider `IgnorePointer`, `ExcludeSemantics`, and state-specific accessibility behavior where relevant.

## Rendering and performance

- Prefer animating `Transform` and `Opacity` when they achieve the intended effect without layout changes.
- `AnimatedPositioned`, `AnimatedSize`, and other layout-affecting animations may be correct, but can trigger layout each frame. Use intentionally and profile when complex.
- Blur, shadows, clipping, opacity layers, shader effects, and large overlapping surfaces can increase raster/compositing cost. Do not call them automatically bad; assess complexity and profile.
- Keep animated rebuild scopes small with `AnimatedBuilder`, `ListenableBuilder`, or a suitable package widget. Avoid rebuilding large unrelated subtrees every tick.
- Do not claim a widget is “GPU-only” without evidence. Flutter's pipeline and device behavior differ from CSS/browser assumptions.
- Profile on representative physical devices in profile mode. Use Flutter DevTools Performance view, frame chart, raster stats, and repaint indicators as appropriate.
- Target smooth frame delivery for the device refresh rate (e.g. 16.7ms at 60Hz, 8.3ms at 120Hz); identify whether UI or raster work is the bottleneck before prescribing a fix.

## Gestures and drag

- A flick can be more meaningful than distance alone; inspect velocity and user expectation when deciding dismissal thresholds.
- Drag should feel continuous and reversible. Avoid jumps when gesture ownership changes.
- Apply resistance near natural boundaries rather than arbitrary hard stops when the design calls for overdrag.
- Handle pointer cancellation, gesture arena behavior, nested scrollables, and multi-touch where relevant.
- Snap/settle behavior should be predictable and interruptible.

## Stagger and sequences

- A restrained stagger of roughly 30–80ms can help a group read as a sequence.
- Stagger is decorative, not a gate: keep interaction available and avoid long cascades.
- Coordinate opacity, position, scale, and clipping so the group reads as one designed motion.
- Avoid animating every item in a large list on every rebuild.

## Accessibility and reduced motion

- Check `MediaQuery.disableAnimations` (and the app's accessibility/platform conventions) for nonessential movement.
- Reduced motion should usually remove or reduce spatial movement, large scaling, parallax, and decorative sequences while retaining essential state communication.
- Keep an alternate cue: opacity/color/state change, immediate layout, semantics, or clear selection indication.
- Do not rely on hover-only animation or visual motion as the sole indication of state.
- Ensure focus, semantics, and interaction remain correct during transitions.

## Debugging and evidence

- Temporarily increase duration 2–5× to inspect path, origin, easing, and synchronization; restore production timing afterward.
- Inspect frame-by-frame when coordinated properties drift or ghost.
- Test gesture-driven motion on real devices, including low/mid-range hardware where appropriate.
- Use profile mode for performance claims; debug mode timing is not representative.
- If the reviewer cannot establish feel from code alone, state that limitation and request a runtime recording or recommend device verification.

## Cohesion

Motion should match the component's personality and the product. Playful surfaces may use restrained bounce; professional tools should remain crisp. Consistency matters, but identical timing everywhere is not cohesion—motion should fit the action and spatial context.
