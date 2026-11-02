---
name: animate-flutter-vocabulary
description: Reverse-lookup glossary for Flutter animation and motion terminology. Use when a user describes an animation or interaction in plain language and wants the precise name to discuss, search for, or prompt an implementation. Maps concepts to Flutter terminology and common widgets/APIs where useful, while distinguishing platform-specific or web-only terms. For naming and explaining motion, not designing or implementing it. Compatible with Flutter 3.47.9 stable / Dart 3.13.7 and newer stable releases.
---

# Flutter Animation Vocabulary

## Initial Response

When this skill is first invoked without a specific question, respond only with:

> I'm ready to put a name to any motion effect you describe in Flutter.

Do not provide any other information until the user asks a question.

## Purpose

Turn a vague description of motion or an effect into the precise term, so the user can describe it to a designer, search for an implementation, or prompt a coding assistant.

This is a **reverse-lookup glossary**, not an animation implementation guide. It names and explains the effect; it does not choose whether the effect should be used, prescribe its duration/curve, or write code unless the user explicitly asks for that next step.

## Compatibility and terminology

**Target baseline: Flutter 3.47.9 stable / Dart 3.13.7 and newer stable releases.**

- Use Flutter terminology and APIs rather than assuming CSS, DOM, SVG, or browser animation behavior.
- Mention a Flutter widget/API only when it is a useful implementation clue, not as a guarantee that it exactly reproduces the visual effect.
- Flutter's animation system commonly uses `AnimationController`, `Animation<T>`, `Tween<T>`, `Curve`, implicit animation widgets, transitions, gestures, and `Hero` animations.
- Platform behavior can differ across Android, iOS, desktop, and web. Call out platform-specific terms when relevant.
- Some vocabulary originates in web animation. If Flutter has no direct equivalent, say so and give the closest Flutter concept as an approximation.
- Do not silently rename a recognized motion-design term merely because Flutter implements it differently.

## Quick Start

The user describes an effect loosely. Return the best matching term in this format:

**Stagger** — Animate several items one after another with a small delay between each, creating a cascade.

If several terms could fit, list the best match first, then 1–2 close alternatives with a one-line distinction.

## Instructions

1. **Read for intent, not keywords.** Users describe what they see or feel (“springy”, “slides off”, “draws itself in”), not the technical name. Map the sensation to the glossary.
2. **Use the glossary's definitions consistently.** Keep definitions concise and stable. You may clarify Flutter-specific implementation terminology after the definition.
3. **Disambiguate close terms.** Examples: pop-in vs bounce, crossfade vs shared-element transition, reveal vs mask, layout animation vs transform, scroll-driven vs scroll-triggered.
4. **Name the visual behavior, not just the Flutter class.** `AnimatedContainer` is an implementation tool; “layout animation” or “color/size transition” is the effect.
5. **When nothing matches exactly, say that.** Name the closest term and mark it as an approximation; do not invent a standard term and present it as canonical.
6. **Stay within this glossary.** If a term genuinely is not included, say so. You may explain it using nearby terms without pretending it is a glossary entry.
7. **Keep naming answers tight.** Lead with the term; expand only if asked.
8. **Do not turn a naming request into implementation advice.** A short Flutter implementation hint is fine when useful; do not provide code unless requested.
9. **Avoid web-only assumptions.** Hover, CSS `clip-path`, CSS fill modes, `will-change`, and browser View Transitions are not Flutter-native terms. Explain the closest Flutter equivalent or mark them web-specific.
10. **Use accessible language.** Define jargon in plain terms; don't assume the user knows animation theory.

## Examples

### Example 1 — feel-based description

User: “What's it called when a popup grows from the button that opened it?”

**Origin-aware animation** — An element appears to expand from its trigger, such as a menu or popup growing from the button that opened it rather than from its own center.

In Flutter, this may be built with an `Overlay`/`CompositedTransformFollower` and a scale/position transition; the term describes the effect, not one specific widget.

### Example 2 — distinguish similar effects

User: “The thumbnail turns into the full image when I open it.”

**Shared-element transition** — The same visual element travels and transforms between two locations or sizes, preserving continuity.

Close alternatives:
- **Morph** — If the shape itself changes smoothly into a different shape.
- **Crossfade** — If one image simply fades out while another fades in at the same location.

Flutter clue: `Hero` is the common built-in tool for shared-element transitions between routes.

### Example 3 — physical feel

User: “When I pull past the end of a list, it stretches a bit and snaps back.”

**Rubber-banding / overscroll effect** — Resistance and snap-back when you drag past a boundary.

Flutter note: the exact feel depends on the platform's `ScrollPhysics` (for example, bouncing vs clamping behavior). “Rubber-banding” is the recognizable motion term.

### Example 4 — Flutter implementation terminology

User: “What is the thing called where a widget animates when its value changes without me writing a controller?”

**Implicit animation** — A widget animates automatically between old and new property values when its configuration changes.

Flutter examples include `AnimatedContainer`, `AnimatedOpacity`, and `TweenAnimationBuilder`. This is an implementation category, not a visual effect by itself.

## Glossary

A curated Flutter-oriented vocabulary. Preserve the distinction between motion terms and the Flutter mechanisms commonly used to implement them.

### Entrances & Exits — how elements appear and disappear

- **Fade in / Fade out** — An element appears or disappears by changing opacity. Flutter clue: `FadeTransition` or `AnimatedOpacity`.
- **Slide in / Slide out** — An element enters or leaves by moving along an axis. Flutter clue: `SlideTransition` or `AnimatedSlide`; offsets are relative to the child's size.
- **Scale in / Scale out** — An element grows or shrinks as it appears or disappears, often paired with a fade. Flutter clue: `ScaleTransition` or `AnimatedScale`.
- **Pop in** — An element appears with a slight overshoot, like it bounces into place. Often implemented with a spring or an overshooting curve.
- **Reveal** — Content is gradually uncovered, often by animating a clip, mask-like effect, or visible region. Flutter clue: `ClipRect`, `ClipPath`, or a custom painter; the exact technique depends on the shape.
- **Enter / Exit animation** — The motion an element plays when it is inserted into or removed from the widget tree. Flutter clue: `AnimatedSwitcher`, `AnimatedList`, or explicit transition widgets.
- **Fade-through** — The outgoing content fades down, then incoming content fades up, creating a brief visual separation between states. Common in Material motion patterns.
- **Shared-axis transition** — Two related views move along a shared horizontal, vertical, or depth axis to communicate their relationship. Common in Material motion patterns.

### Sequencing & Timing — coordinating multiple elements or moments

- **Keyframes** — A sequence of defined values at specific points in an animation, with values interpolated between them. Flutter clue: `TweenSequence`, chained tweens, or a custom animation sequence.
- **Interpolation / Tween** — Calculating intermediate values between a start and end value so motion is continuous. Flutter clue: `Tween<T>` and `Animation<T>`.
- **Stagger** — Animate several items one after another with a small delay between each, creating a cascade. Flutter clue: intervals on a shared controller or per-item delays.
- **Orchestration** — Deliberately coordinating multiple animations so they feel like one composed motion.
- **Delay** — Time before an animation starts. Flutter clue: `Future.delayed` for sequencing or a delayed interval in an animation timeline; avoid timers for frame-by-frame motion.
- **Duration** — How long a time-based animation takes to complete. Flutter clue: `Duration` on controllers or implicit animation widgets.
- **Reverse** — Play an animation back toward its starting value. Flutter clue: `AnimationController.reverse()`.
- **Repeat / Loop** — Run an animation repeatedly, either a fixed number of times or indefinitely. Flutter clue: `AnimationController.repeat()`.
- **Stepped animation** — An animation that changes in discrete jumps rather than continuously. Flutter clue: `StepTween`, discrete curves, or explicitly stepped values.
- **Sequence** — Multiple animation stages played in a defined order, often with different values, curves, or intervals.
- **Fill mode** — In web animation, whether an animation retains its start/end styles before or after playback. Flutter does not use CSS fill-mode semantics; the equivalent is usually the controller's final value and the widget's state after completion.
- **Implicit animation** — A widget automatically animates between old and new values when its configuration changes, without manually managing a controller.
- **Explicit animation** — An animation driven by a developer-managed `AnimationController` and animation objects, allowing precise control over timing, sequencing, repeat, reverse, and interaction.

### Movement & Transforms — changing an element's position, size, or angle

- **Translate** — Move an element along the X or Y axis. Flutter clue: `Transform.translate`, `SlideTransition`, or `AnimatedSlide`.
- **Scale** — Make an element bigger or smaller. Flutter clue: `Transform.scale`, `ScaleTransition`, or `AnimatedScale`.
- **Rotate** — Turn an element around an anchor point. Flutter clue: `Transform.rotate` or `RotationTransition`.
- **Skew** — Slant an element along an axis, shearing it out of its rectangular shape. Flutter clue: `Matrix4` transform; there is no dedicated common skew transition widget.
- **3D tilt / Flip** — Rotate an element in 3D space to create a card-flip or depth effect. Flutter clue: `Transform` with a `Matrix4` rotation and perspective.
- **Perspective** — The strength of a 3D effect, controlled by perspective projection in a transform matrix.
- **Transform origin / Alignment** — The anchor point around which scale or rotation occurs. Flutter clue: `Transform.alignment`.
- **Origin-aware animation** — An element appears to expand or move from its trigger or source location, rather than from an arbitrary center.
- **Parallax offset** — A layer moves at a different rate from another layer, creating an impression of depth; often used in scrolling scenes.
- **Path motion** — An element follows a curved or custom path instead of moving in a straight line. Flutter clue: custom path sampling or a package; not a standard one-widget effect.

### Transitions Between States — connecting one state, view, or element to another

- **Crossfade** — One element fades out as another fades in, in the same location. Flutter clue: `AnimatedCrossFade` or `AnimatedSwitcher` with a fade transition.
- **Continuity transition** — A change that visually connects before and after states so the user keeps track of the same object or context.
- **Morph** — One shape smoothly turns into another shape. Flutter clue: interpolate compatible paths/shapes or use a custom painter; not the same as simply fading between two shapes.
- **Shared-element transition** — An element travels and transforms from one position or size to another, such as a thumbnail expanding into a detail view. Flutter clue: `Hero`.
- **Layout animation** — An element's size or position changes smoothly instead of snapping. Flutter clue: implicit size/position animation, `AnimatedSize`, `AnimatedPositioned`, or animated layout packages.
- **Accordion / Collapse / Expand** — A section smoothly expands or collapses to reveal or hide content. Flutter clue: `SizeTransition`, `AnimatedSize`, or expansion widgets.
- **Direction-aware transition** — Content moves in different directions depending on whether navigation is forward or backward.
- **Page / Route transition** — Motion that connects one route or page to another. Flutter clue: `PageRouteBuilder`, route transition APIs, or the navigator's platform defaults.
- **Shared-axis transition** — Related screens move along a shared axis to signal hierarchy or relationship.
- **Fade-through transition** — A transition that fades outgoing content away before bringing incoming content into focus.
- **Container transform** — A small surface, such as a card or button, expands into a larger destination while maintaining visual continuity. Often associated with Material motion patterns.
- **Hero flight** — The in-between movement of a Flutter `Hero` widget between routes; the implementation term for a shared-element transition.

### Scroll — motion tied to scrolling or navigating between views

- **Scroll reveal** — Elements fade or move into place when they enter the visible viewport. Usually triggered by visibility/scroll position, rather than continuously tied to every scroll pixel.
- **Scroll-driven animation** — Animation progress is directly tied to scroll position. Flutter clue: a scroll controller/listenable updates an animation value; keep work scoped and performant.
- **Scroll-triggered animation** — Scrolling crosses a condition that starts an animation, which then runs independently of the scroll position.
- **Parallax** — Background and foreground move at different speeds while scrolling, creating depth.
- **Collapsing header / App bar** — A header changes size or content as the user scrolls. Flutter clue: slivers such as `SliverAppBar`.
- **Overscroll / Rubber-banding** — Resistance or movement beyond a scroll boundary, followed by a return or spring. Flutter feel depends on `ScrollPhysics` and platform.
- **Page transition** — Motion that plays when moving between routes or pages; not the same as a scroll-driven animation.
- **View transition** — A general term for visually connecting two UI states or pages. The browser View Transitions API is web-specific; Flutter equivalents are route transitions, `Hero`, and custom state transitions.

### Feedback & Interaction — responding to the user's actions

- **Hover effect** — A visual change when a pointer rests over an element. Relevant to Flutter desktop/web; mobile touch has no persistent hover, so use press, focus, or selection feedback instead.
- **Press / Tap feedback** — A subtle visual response when a control is pressed, making it feel responsive. Flutter clue: `InkWell`/`InkResponse` ripple or a scale/color state transition.
- **Ripple** — A circle expands from the tap location to acknowledge a press. Flutter clue: Material ink effects such as `InkWell` or `InkResponse`.
- **Hold to confirm** — A progress effect fills while the user keeps a button pressed; releasing early cancels or resets it.
- **Drag** — Move an element by grabbing it, often with velocity carried into its release. Flutter clue: gesture recognizers and an animation controller/simulation.
- **Drag to reorder** — Dragging an item in a list to a new position while other items move to make room. Flutter clue: `ReorderableListView` or custom drag targets.
- **Swipe to dismiss** — Dragging an element far enough in a direction to remove it. Flutter clue: `Dismissible`.
- **Swipe gesture** — A directional drag used to change state or navigate; distinguish it from swipe-to-dismiss when the element is not removed.
- **Rubber-banding** — Resistance and snap-back when dragging past a boundary; common in iOS scrolling and can be recreated in custom interactions.
- **Shake / Wiggle** — A quick side-to-side movement signaling an error, rejection, or attention.
- **Snap** — An element settles into a discrete position or detent after dragging or scrolling. Flutter clue: snapping scroll physics or a spring/animate-to target.
- **Velocity handoff / Momentum** — The speed and direction from a gesture carry into the settling animation, so a flick feels continuous.
- **Interruptible animation** — An animation can be redirected or retargeted while in progress without an abrupt stop.
- **Detent** — A predefined resting position for a sheet or draggable surface, such as collapsed, medium, or expanded.
- **Gesture-driven animation** — Motion directly tracks a user's finger or pointer, rather than merely starting after the gesture ends.

### Easing — how speed changes over an animation

- **Easing / Curve** — The rate at which an animation changes over time. Flutter clue: `Curves` or a custom `Curve`.
- **Ease-out** — Starts fast and ends slow. Often useful for elements entering or responding to a user action.
- **Ease-in** — Starts slow and ends fast. Can work for exits or motion accelerating away, but may feel sluggish for immediate feedback.
- **Ease-in-out** — Starts slow, speeds up, then slows down. Often useful for movement between on-screen positions.
- **Linear** — Constant rate of change. Useful for progress, rotation, and continuous motion; can feel mechanical for UI movement.
- **Cubic Bézier curve** — A custom easing curve defined by control points. Flutter clue: `Cubic`.
- **Asymmetric easing** — Entry and exit or acceleration/deceleration use different curves, giving movement a more intentional feel.
- **Overshoot curve** — A curve moves beyond its target and returns, creating a bounce-like finish. Flutter clue: elastic/back curves; distinguish from a spring simulation.
- **Curve transform** — Applying a curve to an animation's progress to change its timing profile. Flutter clue: `CurvedAnimation`.

### Spring Animations — physics-based motion

- **Spring** — Motion driven by a physical simulation rather than only a fixed-duration easing curve. Flutter clue: `SpringSimulation`, `SpringDescription`, or spring-based animation APIs.
- **Stiffness / Tension** — How strongly the spring pulls toward its target. Higher stiffness generally feels snappier.
- **Damping** — How quickly oscillation settles. Lower damping allows more bounce; higher damping reduces it.
- **Mass** — How heavy the animated object feels. More mass generally makes motion respond more slowly.
- **Bounce** — A spring-like overshoot and settle that adds playfulness or physicality.
- **Perceptual duration** — How long an animation feels active, even if tiny settling motion continues afterward.
- **Momentum** — Motion that carries velocity from a drag or interruption into the next phase.
- **Velocity** — How fast and in which direction an element is moving. A velocity-aware simulation can preserve the speed of a flick.
- **Interruptible animation** — A spring or controller can be retargeted mid-flight, preserving continuity rather than waiting for completion.
- **Damped oscillation** — Repeated movement around a target that gradually loses amplitude.
- **Critical damping** — The fastest return to equilibrium without oscillation; useful terminology when discussing spring behavior.

### Looping & Ambient Motion — animations that run on their own

- **Marquee** — Text or content scrolls continuously in a loop, often horizontally.
- **Loop** — An animation repeats a set number of times or indefinitely.
- **Alternate / Yoyo** — A loop plays forward and then reverses, instead of jumping back to the start.
- **Orbit** — An element circles around another along a continuous path.
- **Pulse** — A gentle repeating scale, opacity, or color change to draw attention.
- **Float** — A gentle continuous drift up and down, making an element feel weightless.
- **Idle animation** — Subtle motion that plays while an element is waiting for interaction.
- **Breathing animation** — A slow, repeating expansion and contraction, often used for ambient emphasis.
- **Spinner / Indeterminate progress** — Repeating motion that communicates ongoing work when completion time is unknown.
- **Shimmer loop** — A moving highlight repeatedly passes across a placeholder or surface.

### Polish & Effects — small touches and visual treatments

- **Blur** — A blur effect softens an element or background. Flutter clue: `ImageFilter.blur` or a blur widget; large animated blur regions can be expensive.
- **Clip reveal** — Content is revealed by animating a clipping boundary. Flutter clue: animated `ClipRect`/`ClipPath` or custom painting.
- **Mask** — A shape or gradient controls which parts of content are visible, potentially with soft edges. Flutter often needs shader/custom-paint techniques; there is no universal CSS-mask equivalent.
- **Before / After slider** — A draggable divider reveals two overlaid images or views for comparison.
- **Line drawing** — A path draws itself progressively, like an invisible pen tracing it. Flutter clue: animate path metrics or custom painter progress.
- **Text morph** — Text changes with a character-by-character, glyph, or shape transition. This is not the same as a simple crossfade; implementation may require custom text rendering or a package.
- **Skeleton / Shimmer** — A placeholder is shown while content loads, often with a moving sheen.
- **Number ticker** — Digits roll or count up to a new value.
- **Tabular numbers** — Fixed-width digits prevent number columns from shifting as values change; this is typography, not an animation by itself.
- **Typewriter** — Text appears one character at a time, as if being typed.
- **Particle effect** — Many small visual elements move independently, often used for celebration or atmosphere.
- **Confetti** — A burst of falling or scattering particles used to celebrate success.
- **Trail / Motion trail** — A fading trace follows a moving object, suggesting speed or direction.
- **Lottie animation** — A vector-based authored animation played from a Lottie asset. This names a delivery format/implementation, not a motion effect.
- **Custom painter animation** — Motion rendered by repeatedly painting custom geometry; an implementation category rather than a specific effect.

### Performance — terms for smoothness and rendering cost

- **Frame rate (FPS)** — Frames displayed per second. 60fps is a common baseline; high-refresh devices may display 90Hz or 120Hz.
- **Frame budget** — The time available to produce a frame at the display's refresh rate (about 16.7ms at 60Hz, 8.3ms at 120Hz).
- **Jank** — Visible stutter or uneven motion when frames miss their deadlines.
- **Dropped / missed frame** — A frame that fails to arrive on time, causing a hitch or uneven cadence.
- **UI-thread bottleneck** — Frame production is delayed by Dart/framework work such as build, layout, or synchronous computation.
- **Raster-thread bottleneck** — Rendering the layer tree is too expensive, often due to effects, complex painting, or GPU work.
- **Compositing** — Combining painted layers into the final scene. Flutter uses a layer tree and engine compositor; this is not identical to browser CSS compositing.
- **Repaint** — A render object paints again because its visual output changed.
- **Rebuild** — Flutter reruns a widget's `build` method. A rebuild is not automatically expensive; scope and resulting layout/paint work matter.
- **Layout thrash** — Informal term for repeated expensive layout work caused by changing geometry or constraints. Flutter does not use the browser's DOM layout-thrashing model, but repeated layout during animation can still be costly.
- **Repaint boundary** — A layer boundary that can isolate repaint work. Flutter clue: `RepaintBoundary`; useful only when it reduces measured work and its layer cost is acceptable.
- **Offscreen layer / saveLayer** — Rendering content into an intermediate buffer before compositing, which can add memory and raster cost.
- **Shader compilation jank** — A hitch caused by compiling a shader/effect on first use; distinguish from recurring frame cost.
- **Image decode jank** — A hitch caused by decoding or preparing image assets during an interaction.
- **Raster cache** — A cached raster representation of content that may reduce repeated drawing, with memory and invalidation trade-offs.
- **Implicit vs explicit animation cost** — Not a performance ranking by itself; actual cost depends on rebuild, layout, paint, and scene complexity.
- **Hardware acceleration** — A broad platform term for GPU-assisted rendering. Do not assume a Flutter transform or opacity animation is automatically “free” or independently composited.

### Principles to Know — concepts that guide when and how to animate

- **Purposeful animation** — Motion serves a function such as orientation, feedback, explaining relationships, or communicating state, rather than being purely decorative.
- **Anticipation** — A small preparatory movement before the main action, hinting at what is about to happen.
- **Follow-through** — Parts of an element continue or settle slightly after the main motion, adding a sense of weight.
- **Squash & stretch** — Deforming an element as it moves to communicate weight, speed, or flexibility.
- **Perceived performance** — Motion or feedback makes an interface feel faster or more responsive, even when total work is unchanged.
- **Frequency of use** — The more often an animation is encountered, the shorter and subtler it should generally be.
- **Spatial consistency** — Motion preserves an element's identity and spatial relationship across states so users can track where it went.
- **Continuity** — The before and after states are visually connected rather than appearing unrelated.
- **Reduced motion** — Respecting the user's preference for less motion by reducing or removing nonessential movement. Flutter clue: `MediaQuery.disableAnimations` and platform accessibility settings; verify behavior for the target SDK/platform.
- **Motion hierarchy** — More important actions receive clearer motion; secondary elements remain quieter.
- **Motion choreography** — Multiple elements move in a coordinated sequence that supports the user's understanding.
- **Physicality** — Motion responds to gesture, velocity, and constraints in a way that feels tangible.
- **Interruptibility** — The user can change direction or state while motion is in progress without waiting for it to finish.
- **Motion restraint** — Avoiding unnecessary animation, especially for frequent actions or motion that adds no information.

## Flutter-specific mapping notes

Use these notes when a user asks for the Flutter name or implementation clue:

| User describes | Best term | Flutter clue |
|---|---|---|
| A widget automatically animates when a property changes | Implicit animation | `AnimatedContainer`, `AnimatedOpacity`, `TweenAnimationBuilder` |
| A controller drives a custom timeline | Explicit animation | `AnimationController`, `Tween`, `CurvedAnimation` |
| A thumbnail expands into a detail page | Shared-element transition | `Hero` |
| Several items appear one after another | Stagger | Controller intervals / delayed starts |
| A widget fades while being replaced | Crossfade / enter-exit | `AnimatedSwitcher`, `FadeTransition` |
| A section opens smoothly | Expand/collapse / layout animation | `SizeTransition`, `AnimatedSize` |
| A widget follows the finger | Gesture-driven animation | Gesture callbacks + controller/simulation |
| A card can be flicked and settles naturally | Spring / momentum / velocity handoff | `SpringSimulation`, `ScrollSpringSimulation` |
| A list resists at its edge | Overscroll / rubber-banding | `ScrollPhysics` |
| A route animates into another | Page/route transition | `PageRouteBuilder`, route transition APIs |
| A surface expands into a larger destination | Container transform | Custom route/overlay transition; Material motion concept |
| A placeholder has a moving sheen | Shimmer | Gradient animation / package |
| A line draws itself | Line drawing | `PathMetric` + `CustomPainter` |
| A number rolls to a new value | Number ticker | Tween + custom digit transition/package |

## Output format

For a naming request, answer with:

**[Best term]** — [Concise definition grounded in this glossary.]

If useful, add:

**Flutter clue:** `[widget/API/concept]` — [one-line explanation].

If ambiguous:

**Close alternatives:**
- **[Term]** — [when this is the better match].
- **[Term]** — [how it differs].

If no exact term exists:

**Closest term:** [term] — This is an approximation; the exact effect is [plain-language description].

Do not provide code unless the user asks for implementation.

## Tone

Precise, concise, and useful. The goal is to give the user the right words for the motion they have in mind—not to turn every naming question into an animation tutorial.
