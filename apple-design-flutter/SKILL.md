---
name: apple-design-flutter
description: Build and review Apple-inspired interfaces in Flutter using Apple's Human Interface Guidelines and fluid-interface principles translated into Flutter's widget, gesture, animation, Cupertino, accessibility, and rendering systems. Use for iOS-first UI, gesture-driven sheets and drawers, drag/swipe interactions, spring motion, velocity handoff, interruptible transitions, translucent materials, typography, haptics, reduced motion, and Apple-style interaction design. Compatible with Flutter 3.47.9 stable / Dart 3.13.7 and newer stable releases.
---

# Apple Design for Flutter

## Initial Response

When this skill is first invoked without a specific question, respond only with:

> I'm ready to help build fluid, Apple-inspired interfaces in Flutter, grounded in Apple's Human Interface Guidelines and fluid-interface principles.

Do not provide any other information until the user asks a question.

## Purpose

Translate Apple's interface design principles—especially direct manipulation, spatial continuity, responsiveness, fluid motion, restraint, and platform familiarity—into practical Flutter decisions.

This skill is for **designing and implementing Apple-inspired interfaces in Flutter**, especially iOS-first experiences. It is not a mandate to make every Flutter app look like iOS. Choose the platform's conventions when they serve users; adapt thoughtfully when the product has a distinct design system.

The central idea:

**An interface feels fluid when it responds immediately, follows the user's input continuously, carries momentum naturally, respects boundaries, and can be redirected at any time.**

Flutter-specific implementation must preserve those qualities while respecting Flutter's gesture arena, widget lifecycle, animation controllers, rendering pipeline, platform integrations, and accessibility settings.

## Compatibility and implementation assumptions

**Target baseline: Flutter 3.47.9 stable / Dart 3.13.7 and newer stable releases.**

- Prefer stable Flutter SDK APIs available at the stated baseline.
- Do not use deprecated APIs when a current stable alternative exists.
- Check the project's actual Flutter/Dart version and `pubspec.yaml` before adding dependencies or assuming API availability.
- Prefer Flutter SDK APIs first: `AnimationController`, `Tween`, `CurvedAnimation`, `SpringSimulation`, `GestureDetector`, `Listener`, `ScrollPhysics`, `Hero`, `Overlay`, `CustomScrollView`, and Cupertino widgets where appropriate.
- Use packages only when they materially improve fidelity or reduce complexity. Do not install a package for a small interaction that Flutter already supports.
- Apple design principles are not identical to Cupertino widget styling. Use `Cupertino` components for familiar iOS controls, but custom Flutter widgets may be appropriate for a product-specific interface.
- Platform behavior varies by iOS version, device, accessibility setting, and Flutter implementation. Validate the actual target devices.
- Flutter's `Cupertino` library is available through the `cupertino_ui` package in current Flutter documentation; inspect the project's dependency setup before importing it. Do not assume every Cupertino API is part of `package:flutter/cupertino.dart` in every SDK configuration.
- On iOS, native system behavior may be preferable to a custom imitation. Prefer platform-native navigation, menus, sheets, text editing, and system affordances where Flutter exposes them.
- Do not claim an interaction is “native” merely because it uses a Cupertino widget. Validate gesture behavior, accessibility, keyboard/focus behavior, and platform conventions.

## Core idea

Apple-inspired interfaces should feel like an extension of the user's intent—not a sequence of canned animations.

A fluid interface:

- Responds at the moment input begins.
- Tracks touch or pointer movement continuously.
- Preserves the point where an element was grabbed.
- Carries release velocity into settling motion.
- Can be interrupted and redirected mid-animation.
- Uses resistance at boundaries instead of arbitrary hard stops.
- Maintains spatial relationships across transitions.
- Provides clear feedback without over-animating.
- Remains understandable, accessible, and performant.

Design serves human needs: **safety and predictability, understanding, achievement, and delight.** Motion should support those needs, not compete with them.

## 1. Response — eliminate perceived latency

The first response to input is the foundation of directness.

- Give visual feedback on pointer/touch down, not only after a tap completes.
- For a drag, slider, sheet, or interactive surface, update continuously while the gesture is active.
- Avoid unnecessary debounce, artificial delay, or waiting for an animation to finish before accepting input.
- Separate immediate visual response from the asynchronous action. A button can acknowledge the press immediately while a network operation continues.
- Keep feedback lightweight: a subtle highlight, opacity change, scale, or material response is often enough.
- Do not trigger expensive synchronous work in gesture callbacks.
- If an action can fail, provide a clear result state; immediate feedback must not falsely imply success.

Flutter guidance:

- Use `GestureDetector`, `InkWell`, `CupertinoButton`, or a custom `Listener`/gesture recognizer according to the required interaction.
- For simple pressed states, prefer widget state and a short implicit transition over a custom controller.
- For continuous movement, update a `ValueNotifier`, controller, or other local interaction state without rebuilding the entire screen.
- Keep network/database work out of frame callbacks.
- Use `HapticFeedback` only for meaningful events, and pair it with visible feedback.

## 2. Direct manipulation — keep content attached to the finger

> Touch and content should move together.

When the user drags an element, it should follow the pointer 1:1 and preserve the grab offset. Jumping the element's center to the finger breaks the physical illusion.

Rules:

- Record the pointer's starting position and the element's starting position.
- Preserve the offset from the exact point where the user grabbed the element.
- Track movement continuously during the gesture.
- Use local coordinates consistently; avoid mixing global and local coordinate spaces without conversion.
- Do not add easing while the finger is actively dragging. The element should follow the gesture directly.
- Keep the drag responsive even if the pointer moves outside the original hit region.
- Use Flutter's gesture system deliberately. `GestureDetector` participates in the gesture arena; recognizers can compete with scrolling, taps, and other gestures.
- For complex nested gestures, define clear gesture ownership and cancellation behavior rather than layering competing detectors blindly.
- Use `Listener` when raw pointer events are needed, but prefer gesture recognizers for semantic gestures.
- Ensure drag targets have adequate hit areas and do not rely on tiny visual handles.

Flutter implementation clues:

- `GestureDetector` with `onPanStart`, `onPanUpdate`, and `onPanEnd` for common drag interactions.
- `DragGestureRecognizer` or custom recognizers for more specialized gesture-arena behavior.
- `localPosition` / `globalPosition` and `RenderBox.globalToLocal` for coordinate conversion.
- `Transform.translate`, `Positioned`, or a suitable layout approach for visual movement.
- `Draggable` / `DragTarget` for standard drag-and-drop, when their behavior fits.
- `Listener` for pointer-level tracking when required.

## 3. Interruptibility — the most important motion principle

Every user-driven animation should be interruptible and redirectable whenever possible.

A user must be able to grab a moving element, reverse it, or change its destination without waiting for the previous animation to finish.

Rules:

- Never lock out input merely because a transition is running.
- When a gesture begins during an animation, continue from the element's current displayed position—not its old start or intended destination.
- Retarget animations from the current value.
- Preserve velocity where the interaction benefits from it.
- Avoid a sudden velocity discontinuity when changing direction.
- A closing sheet that is grabbed again should follow the finger immediately.
- Do not queue multiple obsolete animations that execute after the user has already changed intent.
- Keep logical state and visual presentation coordinated so an interruption does not cause a jump.

Flutter guidance:

- `AnimationController` can be stopped, retargeted, or driven from its current value.
- `animateTo` and `animateWith` can be used for controlled transitions and simulations.
- For spring settling, use `SpringSimulation` with current position and release velocity.
- When using a controller, derive the displayed value from its current animation state; do not reset it to a stale initial value on every gesture.
- For gesture-driven motion, maintain a clear distinction between **gesture-controlled position** and **settling animation position**, but transition between them without discontinuity.
- Prefer a spring or simulation for interactions that can be interrupted; fixed-duration tweens are better for non-interactive, predictable state changes.
- If a package is used, verify that it supports retargeting and velocity continuity rather than assuming it does.

## 4. Behavior over scripted animation — use springs where input matters

A fixed-duration animation prescribes a timeline. A spring responds to the current state and can naturally continue from new input.

Use springs for:

- Drag release and snap-back.
- Sheet or drawer settling.
- Repositioning floating elements.
- Gesture-driven rotation or scale.
- Interruptible state changes where physical continuity matters.

Use timing/curves for:

- Simple opacity changes.
- Decorative or non-interactive entrances.
- Small state changes that should not inherit gesture velocity.
- Platform-standard transitions that should remain consistent with system behavior.

Flutter's `SpringSimulation` uses physical parameters such as mass, stiffness, and damping. Do not copy Apple's designer-facing damping-ratio/response values directly into Flutter's physical spring parameters without translating them.

Conceptual mapping:

- **Damping ratio near 1:** settles without noticeable oscillation.
- **Damping below 1:** allows overshoot and bounce.
- **Higher stiffness:** responds more quickly.
- **Higher mass:** feels heavier and responds more slowly.
- **Initial velocity:** carries the gesture's speed into the settling phase.

Guidelines:

- Start with a restrained, non-oscillating settle for ordinary UI.
- Add bounce only when the gesture carries momentum or the object should feel playful/physical.
- Avoid bounce on routine menu appearances, simple state changes, or frequent controls.
- Tune by perceived behavior, not by copying arbitrary numbers.
- Remember that a spring's perceived settling time is not a fixed duration.
- Validate spring behavior on the slowest supported device and with reduced motion enabled.

Flutter clue:

- `SpringDescription(mass: ..., stiffness: ..., damping: ...)`
- `SpringSimulation(spring, start, end, velocity)`
- `AnimationController.animateWith(simulation)`

## 5. Velocity handoff — preserve the seam between drag and settle

When a gesture ends, the settling animation should continue in the same direction and at a compatible speed.

A visible seam occurs when the element stops abruptly and then starts a new animation.

Rules:

- Capture release velocity from the gesture.
- Pass velocity into the settling simulation when supported.
- Use the current displayed position as the simulation's starting point.
- Choose the destination using both position and velocity when the interaction is momentum-driven.
- Clamp velocity when necessary to avoid unsafe or absurd travel.
- Distinguish velocity units and coordinate systems. Flutter drag velocity is commonly expressed in logical pixels per second.
- Do not pass raw velocity into a simulation expecting normalized velocity or a different unit.

Flutter guidance:

- `DragEndDetails.velocity` provides gesture velocity.
- `Velocity.pixelsPerSecond` gives a vector in logical pixels per second.
- `SpringSimulation` accepts initial velocity in the simulation's coordinate units per second.
- For 2D motion, preserve X and Y components independently when their directions or targets differ.
- For scroll-like interactions, use Flutter's scroll physics/simulations rather than inventing a separate model unless the interaction is custom.

## 6. Momentum projection — land where the gesture is going

For flickable strips, cards, sheets, and custom pagers, snapping to the nearest point from the release position can feel unresponsive. Project the release momentum and select a destination near the projected endpoint.

Use this when:

- The user can flick an element or collection.
- The gesture has meaningful velocity.
- The destination should reflect intent, not just the final finger position.

Do not use momentum projection for every drag. A slow precision drag, a slider, or a constrained repositioning may need a direct nearest-target rule.

Flutter guidance:

- Prefer Flutter's existing scroll physics and snapping behavior for scrollables.
- For custom motion, estimate a projected endpoint from release velocity and a deceleration model, then choose the valid snap target.
- Keep the projection model consistent with the spring simulation that follows.
- Apply bounds and sensible velocity caps.
- Preserve release velocity in the spring so the element does not appear to hit an invisible wall.
- Test slow drags, fast flicks, reverse flicks, and release near each boundary.

## 7. Spatial consistency — maintain the user's mental map

Motion should explain where an element came from and where it went.

Rules:

- Enter and exit along compatible paths.
- If a panel enters from one edge, dismiss it in a way that preserves the same spatial relationship.
- Anchor menus and popovers to their triggering controls.
- Use shared-element transitions when the same object moves between contexts.
- Preserve hierarchy: related content should move together; unrelated content should not.
- Do not use arbitrary directional movement that implies a false relationship.
- Match the transition to the navigation model: forward/back, parent/child, peer-to-peer, or temporary overlay.
- Reversible transitions should feel like the same interaction played backward, not an unrelated animation.

Flutter guidance:

- Use `Hero` for shared-element transitions between routes when it fits.
- Use route transitions appropriate to the platform and navigation structure.
- Use `Overlay` and composited positioning for anchored popovers where needed.
- For custom transitions, preserve source/destination geometry and use a consistent alignment/origin.
- Prefer native route behavior for standard iOS navigation rather than recreating it unnecessarily.

## 8. Hint in the direction of the gesture

People predict an outcome from the trajectory of an object.

- Intermediate motion should communicate the likely destination.
- A sheet pulled upward should visually suggest expansion; pulled downward, dismissal.
- A card dragged toward a target should remain visually connected to that target.
- Use subtle scale, opacity, or depth cues only when they clarify the outcome.
- Avoid adding cues that conflict with the actual release decision.
- Ensure the interaction remains understandable when motion is reduced or disabled.

## 9. Rubber-banding — soften boundaries

At a boundary, resistance should increase progressively rather than stopping abruptly.

Use rubber-banding for:

- Custom draggable surfaces.
- Sheet overshoot.
- Repositionable cards.
- Custom carousels or interactive canvases.

Rules:

- Apply resistance only beyond the valid range.
- The farther the user pulls past the boundary, the less additional movement should be produced.
- Keep the response continuous—no sudden clamp or discontinuity.
- On release, settle to a valid position with a spring.
- Avoid rubber-banding where a hard boundary is essential for safety or precision.
- Do not replace standard platform scrolling physics with custom resistance unless the interaction requires it.

Flutter guidance:

- For standard scrolling, use platform-appropriate `ScrollPhysics`.
- For custom dragging, transform the overshoot with a resistance function.
- Use a spring to return to the nearest valid boundary.
- Test both directions and different container sizes.

## 10. Gesture design details

### Tap

- Show pressed feedback immediately.
- Commit the action on tap-up/recognized tap, not on pointer-down.
- Allow cancellation when the user drags away.
- Provide an adequate touch target (generally at least 44×44 logical points for iOS controls).
- Do not use a scale animation that makes the hit target unstable.
- Avoid delaying taps unless another gesture genuinely requires disambiguation.

### Drag and swipe

- Use a small, intentional movement threshold to distinguish taps from drags.
- Track movement 1:1 after the gesture is recognized.
- Preserve the grab offset.
- Hand off release velocity.
- Support cancellation and interruption.
- Avoid distance-only decisions when a clear flick should count; consider velocity and direction.
- Do not make users drag precisely to a tiny threshold.

### Nested gestures

- Consider scroll-vs-drag competition explicitly.
- Let vertical scrolling win when the intent is vertical scrolling; let horizontal gestures win only when their intent is clear.
- Use gesture-arena teams or custom recognizers only when necessary.
- Avoid multiple gesture detectors that each assume they own the pointer.
- Test diagonal gestures, short drags, cancellations, and accessibility alternatives.

### Sheets and drawers

- Use platform-native sheet behavior when it satisfies the interaction.
- For custom sheets, support grab, dismiss, snap points, and re-grab during motion.
- Preserve velocity at release.
- Make dismissal thresholds depend on intent (position and/or velocity), not distance alone.
- Ensure content remains scrollable and that sheet dragging does not steal scroll gestures unexpectedly.
- Keep drag handles and interactive regions large enough.
- Use a scrim when the sheet is modal; do not dim the background for a truly non-blocking panel.

## 11. Frame-level smoothness

Fluid motion is about consistent frame delivery and coherent movement—not merely a high FPS number.

Flutter-specific rules:

- Keep per-frame work small.
- Do not call `setState` for an entire screen on every pointer update if only a small element changes.
- Avoid expensive synchronous computation in animation listeners.
- Use `AnimatedBuilder`, `ListenableBuilder`, `ValueListenableBuilder`, or a narrowly scoped state object to limit rebuilds.
- Prefer transforms and opacity for visual movement when layout does not need to change.
- Use layout animation only when surrounding content must reflow.
- Avoid repeated intrinsic layout, large subtree rebuilds, expensive clipping, large blur regions, and unnecessary `saveLayer` operations during motion.
- Use `RepaintBoundary` only after profiling; it can reduce repaint work but adds layer/cache costs.
- Do not assume a transform is free. Rasterization, compositing, clipping, and GPU workload still matter.
- Profile in profile/release mode on physical devices, not only in debug mode or a simulator.
- Consider 60Hz, 90Hz, and 120Hz frame budgets where supported.
- Use Flutter DevTools to distinguish UI-thread and raster-thread bottlenecks.
- If a first-use effect stutters, investigate shader compilation and asset preparation separately from recurring frame cost.

## 12. Materials and depth — translucency conveys hierarchy

Apple uses translucent materials to create functional layers that preserve context while separating controls from content.

Flutter guidance:

- Use translucency when it communicates a floating layer, toolbar, sheet, or navigation surface—not as decoration everywhere.
- Content may scroll beneath floating chrome when that relationship is clear and readable.
- Use `BackdropFilter` with a translucent surface only when needed; blur can be expensive, especially over large areas.
- Keep text and controls legible over changing backgrounds. Use adequate contrast and, where necessary, a more opaque material.
- Avoid stacking multiple translucent surfaces if contrast and hierarchy collapse.
- Larger surfaces may need stronger separation than small chips, but shadows should remain subtle and consistent.
- Use scrims for modal focus; do not dim content behind a non-blocking panel unless the interaction calls for it.
- A glass-like surface should materialize through a coordinated change in opacity, scale, and position—not merely a generic fade.
- Do not animate blur radius continuously unless the effect is essential and performance has been measured.
- Respect platform appearance and accessibility settings; provide an opaque or reduced-transparency alternative where appropriate.

Flutter implementation clues:

- `BackdropFilter`, `ImageFilter.blur`, translucent `DecoratedBox`/`Container`
- `ClipRect` around the filtered region to constrain work
- `SafeArea` for system bars and cutouts
- `AnimatedOpacity` / `Transform` for material entrance and exit
- Theme-aware colors and contrast-aware foregrounds

## 13. Multimodal feedback — motion, sound, and haptics

Feedback should reinforce the same causal event.

Rules:

1. **Causality:** Trigger feedback when the action actually occurs—selection changes, a detent catches, a drag commits, an operation succeeds or fails.
2. **Harmony:** Visual, sound, and haptic feedback should feel synchronized. Do not wait until an animation finishes if the physical event occurs at the start.
3. **Utility:** Use haptics and sound sparingly. Reserve them for meaningful selection, success, error, commit, or snap moments.
4. **Accessibility:** Never make haptics or sound the only indication of state.
5. **Platform fit:** Use the platform's expected feedback style rather than forcing identical effects across iOS and Android.

Flutter guidance:

- `HapticFeedback.selectionClick()`, `HapticFeedback.lightImpact()`, `HapticFeedback.mediumImpact()`, and related APIs can provide platform feedback.
- Verify exact behavior on physical iOS and Android devices; support and intensity vary.
- Do not trigger haptics per pointer update, scroll frame, or animation tick.
- Trigger one haptic per meaningful user action.
- Provide visible state feedback even when haptics are disabled or unavailable.

## 14. Reduced motion, accessibility, and transparency

Reduced motion is not “remove all feedback.” It means provide a gentler equivalent while preserving meaning.

- Respect platform accessibility settings and Flutter's accessibility signals.
- Use `MediaQuery.disableAnimations` where applicable, and verify the signal against the project's target SDK and platform.
- Replace large translations, parallax, elastic overshoot, and repeated movement with opacity, color, or immediate state changes.
- Keep transitions short and calm when motion is reduced.
- Do not rely on animation alone to communicate hierarchy, success, failure, or selection.
- Avoid persistent looping movement that distracts from content.
- Ensure gestures have accessible alternatives where possible.
- Maintain minimum touch targets, semantics, focus order, keyboard support, and screen-reader labels.
- Respect text scaling. Never assume a measured fixed height remains valid at larger accessibility text sizes.
- Avoid clipping content when text scales.
- Provide a more opaque material when reduced transparency or contrast settings require it.
- Test VoiceOver, Dynamic Type/text scaling, bold text, increased contrast, reduced motion, and platform-specific navigation behavior.

Flutter guidance:

- `MediaQuery` accessibility properties and `Semantics`
- `SafeArea`, `Focus`, `FocusableActionDetector`, and keyboard shortcuts where appropriate
- `Cupertino` controls or adaptive widgets for familiar platform interaction
- `TextScaler`-aware layout rather than fixed text heights

## 15. Typography — optical sizing, tracking, and leading

Apple's typography is carefully tuned to size, hierarchy, and readability.

Flutter guidance:

- Prefer the platform's system font for iOS-first interfaces unless the product has a deliberate brand typeface.
- Use `CupertinoTheme`/`ThemeData` text styles consistently rather than styling every text widget independently.
- Build hierarchy from size, weight, line height, and color together—not font size alone.
- Large display text may need tighter tracking; body text should prioritize readability.
- Use line heights appropriate to the size and script.
- Respect text scaling and avoid fixed-height text containers.
- Avoid excessive all-caps, overly tight letter spacing, and low-contrast secondary text.
- Validate typography with longer strings, localization, Dynamic Type, and different scripts.
- Do not assume web CSS `rem`, `em`, or optical-sizing behavior maps directly to Flutter. Use Flutter's `TextStyle`, `TextScaler`, and layout constraints.

## 16. Design foundations

Use these principles to make decisions beyond motion.

1. **Purpose:** Build with intention. Every feature consumes time, attention, and trust. Remove elements that do not serve the user's goal.
2. **Agency:** Keep users in control. Offer clear choices, undo where practical, and confirmations only for genuinely destructive or irreversible actions.
3. **Responsibility:** Act in the user's interest. Ask for permissions when needed, explain why, collect only what is necessary, and make risks understandable.
4. **Familiarity:** Build on platform conventions. Similar-looking controls should behave consistently. Break a familiar pattern only when there is a clear benefit and evidence.
5. **Flexibility:** Adapt to device, context, input method, language, and ability. iPhone, iPad, Mac, and web may need different layouts and interaction models.
6. **Simplicity:** Remove unnecessary complexity, not useful context. Make the primary path obvious and advanced options discoverable.
7. **Craft:** Sweat the details—alignment, typography, contrast, responsive behavior, motion, and edge cases. Craft is validated through iteration.
8. **Delight:** Delight should emerge from the other principles being done well, not from decorative animation added afterward.

Tactical rules:

- Feedback should communicate status, completion, warning, or error.
- Every screen should answer: Where am I? What can I do? Where can I go? How do I leave?
- Place controls near the content they affect.
- Use specific labels rather than vague categories.
- Keep navigation predictable and avoid trapping users.
- Design empty, loading, error, and success states as part of the product—not afterthoughts.
- Prefer undo for recoverable mistakes; reserve confirmation dialogs for meaningful irreversible actions.
- Ask for permissions in context and explain their value.
- Avoid copying iOS visuals without copying the interaction logic that makes them familiar.

## 17. Flutter platform adaptation

Apple-inspired design does not mean forcing iOS patterns onto every platform.

### iOS

- Prefer familiar iOS navigation, gestures, safe-area behavior, system text editing, and Cupertino controls where appropriate.
- Respect back-swipe behavior and avoid gesture conflicts with system navigation.
- Use native-feeling sheets and context menus where supported.
- Validate on real iPhones and iPads, including Dynamic Island/notch and home-indicator areas.

### iPadOS

- Do not simply stretch an iPhone layout.
- Use appropriate multi-column layouts, popovers, sidebars, keyboard shortcuts, pointer support, and larger interaction surfaces.
- Support multitasking and changing window sizes.
- Avoid oversized empty space or controls that are too far apart.

### macOS

- Support pointer precision, hover, focus, keyboard navigation, window resizing, and standard menu behavior.
- Do not assume touch gestures are the primary input.
- Use desktop conventions for toolbars, menus, shortcuts, and window controls.

### Android and other platforms

- Keep the product's visual identity, but respect platform navigation, system back behavior, touch feedback, and accessibility conventions.
- Use adaptive patterns when the same control should behave differently across platforms.
- Avoid presenting an iOS-only affordance as universal if it conflicts with platform expectations.

### Web

- Consider pointer, keyboard, hover, focus, browser navigation, and responsive layouts.
- Do not assume iOS haptics or native sheet behavior exists on web.
- Use Flutter's web rendering behavior and accessibility support; validate in target browsers.

## 18. Process

- **Prototype interactively.** A working prototype reveals gesture and motion problems that static mockups cannot.
- **Design interaction and visuals together.** Motion is part of the interface, not a layer added after the pixels.
- **Make one intentional decision at a time.** Identify the user need, choose the interaction model, then tune visuals and motion.
- **Review interruption paths.** Test starting, stopping, reversing, and retargeting every gesture-driven animation.
- **Test real people in context.** Observe whether the interface communicates state and intent without explanation.
- **Review on physical devices.** Simulator feel is not sufficient for haptics, gesture conflicts, frame pacing, or platform fidelity.
- **Use accessibility settings during review.** Reduced motion, larger text, VoiceOver, contrast, and focus can change the right design.
- **Prefer evidence over taste alone.** If an interaction feels wrong, inspect latency, gesture ownership, velocity continuity, spatial consistency, and frame performance.

## Quick reference

| Need | Flutter approach |
|---|---|
| Immediate press response | Press state / `InkWell` / `CupertinoButton` |
| Finger-tracked movement | Gesture recognizer + local position |
| Preserve grab offset | Record pointer-to-element offset |
| Interruptible settle | Controller retargeting / spring simulation |
| Velocity handoff | `DragEndDetails.velocity` → simulation initial velocity |
| Flick to a snap point | Project release momentum, choose target, spring with velocity |
| Soft boundary | Rubber-band mapping + spring settle |
| Standard scroll feel | Platform-appropriate `ScrollPhysics` |
| Shared object between routes | `Hero` |
| Anchored popover | Overlay + composited positioning |
| Native-feeling iOS controls | Cupertino widgets / native platform behavior where available |
| Translucent material | Constrained `BackdropFilter` + readable surface |
| Reduced motion | Short fade or immediate state; remove large movement/overshoot |
| Haptic selection | One haptic at the selection/commit event |
| Smooth rendering | Profile UI and raster work; scope rebuilds and paints |
| Accessible text | TextScaler-aware layout and no fixed text heights |

## Output expectations

When implementing or reviewing an interface:

1. State the design decision and its purpose in one or two lines.
2. Provide the Flutter code or focused changes.
3. Explain the motion/interaction ingredients briefly:
   - interaction model
   - gesture tracking and interruption behavior
   - spring/timing choice
   - velocity handoff or snap logic, if relevant
   - accessibility and platform adaptation
4. Identify what must be checked on a real device:
   - drag tracking and grab offset
   - interruption/reversal
   - velocity handoff and boundary behavior
   - haptic timing
   - reduced motion and text scaling
   - performance on the slowest supported device

Do not pad the answer with a design essay. Make clear decisions, implement them, and name the important trade-offs.

## Tone

Opinionated, practical, and precise. Prefer familiar platform behavior over clever custom motion. Use springs for interactions that need physical continuity, restraint for frequent UI, and custom behavior only when it improves clarity or control.

The goal is not to make Flutter imitate screenshots of iOS. The goal is to make the interface feel coherent, responsive, understandable, and at home on the platform.
