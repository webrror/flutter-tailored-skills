# Flutter Animation Audit Playbook

This is the rule catalog for `improve-flutter-animations`. Use it to evaluate existing Flutter motion and to choose exact target values for plans. It adapts the source skill's principles to Flutter's animation, gesture, and rendering model.

**Do not copy CSS/browser implementation details into Flutter plans.** Use the Flutter-native equivalents below. When a value is a starting recommendation rather than a universal rule, label it as a proposed target and include a feel-check.

## 1. Purpose & frequency

Every animation must answer: **why does this animate?**

Valid purposes:
- Feedback.
- State indication.
- Spatial continuity.
- Preventing a jarring change.
- Explanation.
- Rare delight.

Frequency guide:

| Frequency | Decision |
|---|---|
| 100+ times/day: keyboard actions, repeated tab switching, common commands | Usually no animation; instant state change |
| Tens of times/day: selection, row navigation, press feedback | Subtle and brief, or none |
| Occasional: dialogs, sheets, menus, toasts | Standard purposeful transition |
| Rare / first-time: onboarding, success, celebration | Limited opportunity for delight |

Hunt for:
- Keyboard-triggered actions with unnecessary transitions.
- Repeated navigation and tab changes that add latency.
- Decorative motion on frequently encountered list rows.
- Controls whose feedback is slower than the action.
- Repeated animation that does not explain a state change.

The strongest fix is often to remove the animation.

## 2. Curves & duration

### Curve intent

| Situation | Flutter target |
|---|---|
| Entering / appearing | Ease-out, e.g. `Curves.easeOutCubic` |
| Exiting / dismissing | Direct, responsive exit; choose curve based on direction and platform convention |
| Moving / morphing on screen | Ease-in-out, e.g. `Curves.easeInOutCubic` |
| Constant motion | `Curves.linear` |
| Gesture-driven movement | Track gesture directly; settle with a spring or velocity-aware simulation |
| Platform route transition | Prefer platform default unless the design system deliberately overrides it |

Reference cubic curves:

```dart
const easeOutUi = Cubic(0.23, 1.0, 0.32, 1.0);
const easeInOutUi = Cubic(0.77, 0.0, 0.175, 1.0);
const easeSheet = Cubic(0.32, 0.72, 0.0, 1.0);
```

Use existing project tokens if present. These are reference targets, not a reason to create duplicate tokens.

**`Curves.easeIn` on a routine UI entrance is a finding** when it creates a slow start at the moment the user expects a response. It is not automatically wrong for every exit or deliberate sequence.

### Duration budgets

| Element | Typical target |
|---|---:|
| Press feedback | 100–160 ms |
| Tooltip / compact popover | 125–200 ms |
| Dropdown / select | 150–250 ms |
| Small state transition | 150–200 ms |
| Dialog / sheet / drawer | 200–350 ms perceived; platform defaults may differ |
| Marketing / explanatory / celebratory motion | May be longer when justified |

Routine UI motion should usually remain under 300 ms. Long durations are findings only when they harm responsiveness or conflict with the component's role.

Hunt for:
- Slow-start curves on entrances.
- Arbitrary custom curves repeated across the app.
- Routine transitions longer than their purpose warrants.
- A delay plus animation on every tooltip in a toolbar.
- A transition duration that changes unpredictably between related components.

## 3. Physicality & spatial origin

Flutter has no CSS `transform-origin` property. Use `Transform.alignment`, `FractionalOffset`, `Alignment`, or the appropriate route/overlay geometry.

### Scale and opacity

Avoid `scale: 0` as a default entrance. Use a restrained initial scale around **0.90–0.97** with opacity only when scaling adds meaning. A fade alone may be better.

For press feedback, a subtle scale around **0.97** is a reasonable starting point; keep it between approximately **0.95–0.98** and validate the actual control.

### Origin

- Trigger-anchored menus/popovers should visually emerge from their trigger or placement edge.
- Centered dialogs should generally remain centered; center is not a defect for a centered modal.
- Bottom sheets should move from the bottom edge and follow platform expectations.
- If an overlay flips near a screen edge, its transform alignment should follow the final placement.
- Prefer native menu/dialog/sheet APIs if they already provide correct geometry and motion.

### Flutter-specific caveats

- `Transform` changes painting, not layout footprint.
- `AnimatedPositioned` and `AnimatedAlign` can cause layout work; this may be correct if siblings need to move.
- `AnimatedScale` and `ScaleTransition` are not automatically cheap if the subtree is expensive to paint.
- Avoid arbitrary scale effects that reduce legibility or clip shadows.

## 4. Interruptibility & state changes

Dynamic UI must respond to new input instead of replaying stale motion.

Prefer:
- Implicit animations for simple state-driven property changes that can retarget.
- `AnimationController` when explicit control, sequencing, or interruption is needed.
- `AnimatedSwitcher` for genuine content replacement, with stable keys and intentional transitions.
- Gesture-linked values and simulations for direct manipulation.
- Existing project spring/physics abstractions when they are established.

Hunt for:
- `forward()` restarted from zero on every rapid tap.
- Repeated mount animations caused by unstable keys.
- Keyed child replacement that causes unnecessary transitions.
- Toasts, toggles, or expand/collapse states that jump or restart awkwardly.
- Gesture animations that ignore release velocity.
- Dismissal based only on distance when a deliberate flick should count.
- Hard boundary stops where a small resistance would feel more natural.
- Controllers not disposed, tickers not muted offscreen, or repeating animations that continue when irrelevant.

### Gesture guidance

Flutter drag velocity is expressed in logical pixels per second. Evaluate:
- Direction.
- Distance relative to the component extent.
- Velocity in the intended direction.
- Gesture cancellation.
- Scroll/gesture arena ownership.

Do not import web velocity thresholds directly. Set the actual threshold from the component's scale and test slow drags, quick flicks, reverse direction, and cancellation.

### Spring targets

Flutter's `SpringSimulation` uses physical parameters (`mass`, `stiffness`, `damping`) rather than the exact high-level spring syntax used by some web libraries. Do not pretend those configurations are interchangeable.

For plans:
- Reuse the project's existing spring helper if available.
- Otherwise choose a physically justified `SpringDescription` and include a device feel-check.
- Avoid visible bounce on routine UI; reserve it for momentum-bearing gestures or intentional playfulness.

## 5. Performance

Flutter animation performance is about build, layout, paint, compositing, rasterization, and platform work—not CSS compositor rules.

### Hunt for

- Broad `setState` or listeners rebuilding large subtrees every frame.
- `AnimatedBuilder` that rebuilds static children unnecessarily.
- Expensive blur (`ImageFiltered`, `BackdropFilter`) animated over large regions.
- Animated clipping (`ClipPath`, complex `CustomClipper`) over large or repeated content.
- Large opacity layers or nested opacity effects.
- Heavy shadows, shader/effect work, or expensive decorations in animated lists.
- Layout animation where paint-only movement would suffice, or paint-only movement where layout must actually change.
- Repeating controllers that remain active offscreen.
- Animation controllers/tickers not disposed correctly.
- Animation on many virtualized list rows at once.
- Unnecessary `RepaintBoundary` additions or missing boundaries confirmed by profiling.
- Unbounded animation work or unnecessary timers.

### Flutter performance rules

- Scope frame updates with `AnimatedBuilder`, `ListenableBuilder`, or the project's established pattern.
- Pass static subtrees through `AnimatedBuilder.child`.
- Keep animated subtrees small.
- Prefer transforms/opacity when they produce the same visual behavior without layout; do not treat them as universally free.
- Use `RepaintBoundary` only when profiling supports it.
- Do not animate layout properties merely because an implicit widget makes it easy; but do not reject layout animation when reflow is the intended behavior.
- Profile with Flutter DevTools in profile/release mode on representative devices.
- Distinguish UI-thread frame cost from raster-thread cost.
- Do not claim a performance fix is verified from source inspection alone.

## 6. Accessibility

### Reduced motion

Flutter's system accessibility preference is surfaced through `MediaQuery.disableAnimations` (or the project's equivalent policy).

Reduced motion means fewer and gentler animations, not necessarily zero:
- Preserve opacity/color feedback that communicates state.
- Reduce or remove translation, scale, parallax, overshoot, and decorative loops.
- Make essential state changes understandable without movement.
- Avoid making information available only through animation.
- Verify route/sheet behavior under the platform preference.

Do not use CSS `prefers-reduced-motion` in Flutter plans.

### Input and semantics

- Touch interactions must not depend on hover.
- Desktop hover can be appropriate for pointer devices, but must not be the only feedback.
- Keyboard focus and activation should remain functional.
- Custom animated controls need meaningful semantics and labels.
- Ensure animation does not hide focus, clip semantics, or reduce target size.
- Test large text scaling and localized labels in animated layouts.

## 7. Cohesion & motion tokens

Motion should match product personality.

- A productivity or data-heavy app should generally feel crisp and quiet.
- Playful products may use more expressive movement, but remain predictable.
- Platform-native flows should respect Material/Cupertino conventions unless intentionally customized.
- Related components should share duration and curve tokens.
- Do not create a parallel motion system if the project already has one.
- Near-identical hard-coded curves/durations are a consolidation finding when they create drift.
- Inconsistent route, dialog, sheet, menu, and toast motion is a finding when the mismatch is visible.
- Staggering can help explain a curated sequence, but must never block interaction.
- Avoid decorative motion that fights the product's information hierarchy.

## 8. Missed opportunities

This is additive. Report only a handful, grounded in observed UX seams.

Potential opportunities:
- Content swaps that visibly teleport and could use a short `AnimatedSwitcher`.
- A panel that appears from a trigger without spatial continuity.
- A rare success state that would benefit from restrained feedback.
- An expand/collapse that changes layout abruptly and could use `AnimatedSize`.
- A list insertion/removal that could use `AnimatedList` or a project-standard list transition.
- A state change where a brief opacity/color transition would clarify what changed.

Do not recommend motion for:
- Every tab switch.
- Every list row.
- Every value update.
- Keyboard-driven actions that should be immediate.
- Decorative motion without a user-facing purpose.

Report at most 2–4 missed opportunities, and only when evidence supports them.

## Severity guide

- **HIGH:** responsiveness, comprehension, or interaction reliability is materially harmed.
- **MEDIUM:** motion is noticeably wrong, inaccessible, non-interruptible, or spatially incoherent.
- **LOW:** cohesion, token, or subtle polish issue.

Prioritize by impact, frequency, effort, and confidence—not by how visually interesting the fix is.
