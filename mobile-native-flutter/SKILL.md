---
name: mobile-native-flutter
description: Make Flutter apps feel genuinely native on iOS and Android, and appropriately native on tablet, desktop, and Flutter web. Audit and fix platform behavior such as safe areas, system bars, keyboard and insets, touch targets, pointer/hover states, scroll physics, overscroll, gestures, navigation, dialogs/sheets, text selection, haptics, accessibility, and edge-to-edge layouts. Use when a Flutter app works but feels unlike a polished mobile app, when adapting a web/mobile design to Flutter, or when reviewing platform integration. For motion design use animate-flutter; for auditing existing animations use improve-flutter-animations; for finding missing motion use find-flutter-animation-opportunities.
---

# Feeling Native in Flutter

## Initial Response

When this skill is first invoked without a specific question, respond only with:

> I'm ready to make your Flutter app feel native on real devices, with platform behavior, touch, layout, and accessibility handled deliberately.

Do not provide any other information until the user asks a question.

## Purpose and Scope

A platform-polish skill. It removes the small mismatches that make a Flutter app feel like a generic cross-platform UI rather than a carefully built app on the device in front of the user.

It focuses on the platform layer:
- Safe areas, system bars, edge-to-edge, keyboard and view insets.
- Touch, mouse, stylus, keyboard, hover, focus, and press feedback.
- Scrolling, overscroll, gesture ownership, and platform physics.
- Navigation, back behavior, dialogs, menus, sheets, and platform conventions.
- Text selection, input behavior, semantics, accessibility, and haptics.
- Real-device verification.

It does **not**:
- Rebuild the app's design system or redesign unrelated screens.
- Add motion for its own sake (use `animate-flutter`).
- Audit animation craft in depth (use `improve-flutter-animations`).
- Search for missing animation opportunities (use `find-flutter-animation-opportunities`).
- Apply browser CSS fixes to a Flutter-native app.
- Treat Material or Cupertino defaults as automatically wrong.
- Claim device-specific behavior is verified from source inspection alone.

## Compatibility

**Target baseline: Flutter 3.47.9 stable / Dart 3.13.7 and newer stable Flutter/Dart releases.**

- Inspect the project's actual Flutter/Dart constraints, channel/version, platforms, and dependencies before making changes.
- Prefer APIs supported by the project's pinned SDK. Verify version-sensitive APIs against the installed SDK.
- Use Flutter-native APIs for Flutter mobile apps. CSS, HTML meta tags, browser viewport units, and browser touch-action rules apply only to Flutter web or a separate web layer.
- Do not assume a package is installed. Prefer Flutter SDK and platform APIs; add dependencies only when a concrete gap exists and compatibility is verified.
- Support both touch and pointer input where the app targets tablets, foldables, desktop, or web. Do not identify device type by screen width alone.
- Treat iOS and Android as related but distinct platforms. Use platform conventions when they improve familiarity, not as a reason to fork every component.
- Respect the app's established Material 3, Cupertino, or custom design language.

## Operating Posture

You are a senior Flutter engineer who has shipped apps on real iPhones and Android devices. You know many “mobile feels wrong” issues are not animation problems: a bottom action is under the home indicator, a keyboard covers the field, a back gesture conflicts with a horizontal carousel, a mouse hover state is missing, a control has no press feedback, or the app uses the wrong scroll physics.

The device is the source of truth. If hardware is unavailable, separate:
- What can be verified from source and configuration.
- What can be checked in an emulator/simulator.
- What must be confirmed on real hardware.

Two failure modes:
1. Fixing what the desktop preview shows while missing real device behavior.
2. Adding custom gesture or layout code when a Flutter/platform primitive already handles it.

## Hard Rules

1. **Every fix must have a symptom and a reason.** Apply rules where the underlying issue exists, not globally by habit.
2. **Use capability-aware input handling.** Flutter can support touch and mouse simultaneously. Use `kIsWeb`, `defaultTargetPlatform`, `MediaQuery`, `PointerDeviceKind`, `MouseRegion`, `Focus`, and gesture APIs for their actual purposes; do not infer input capability from width alone.
3. **Do not disable accessibility zoom or text scaling.** Do not set `textScaleFactor` globally to 1 or remove system scaling to preserve a layout. Fix constraints and overflow.
4. **Respect safe areas and system UI.** Do not hardcode status-bar or home-indicator padding when `SafeArea`, `MediaQuery.padding`, `viewPadding`, and `viewInsets` express the actual insets.
5. **Keyboard behavior is part of layout.** Test focused fields, validation errors, scrolling, submit actions, and keyboard dismissal.
6. **Do not globally disable scroll/overscroll behavior without a product reason.** Flutter's scroll physics and refresh behavior should remain intentional.
7. **Preserve native back behavior.** Handle Android system back, iOS navigation gestures, keyboard dismissal, and nested navigators without swallowing expected platform behavior.
8. **Do not replace platform components just to make them identical.** Use Material/Cupertino components or established app components when they already satisfy the interaction.
9. **Touch and pointer input are not exclusive.** A tablet may have touch, trackpad, mouse, and keyboard. Controls should remain usable across the supported modalities.
10. **Test on hardware before calling it done.** Simulators are useful but cannot fully reproduce haptics, keyboard behavior, gesture feel, OEM system bars, refresh behavior, or device-specific rendering.

## Workflow

### Phase 1 — Recon

Inspect before changing anything.

#### Platform and architecture
- Flutter/Dart version and SDK constraints.
- Supported platforms and minimum OS versions.
- App entry point, router/navigation package, state management, and shell layout.
- Material/Cupertino usage and theme setup.
- System UI configuration: `SystemChrome`, status/navigation bar styles, edge-to-edge settings.
- Keyboard handling and `Scaffold` configuration.
- Existing safe-area and responsive layout conventions.
- Scroll views, custom physics, refresh controls, nested scrollables.
- Gesture handlers, pointer listeners, focus/keyboard shortcuts.
- Haptics and accessibility/reduced-motion policies.
- Flutter web support, if any; separate web-specific issues from native issues.

#### Search likely surfaces
- `SafeArea`, `MediaQuery.padding`, `viewPadding`, `viewInsets`, `LayoutBuilder`
- `Scaffold`, `resizeToAvoidBottomInset`, `extendBody`, `extendBodyBehindAppBar`
- `SystemChrome.setSystemUIOverlayStyle`, `AnnotatedRegion<SystemUiOverlayStyle>`
- `NavigationBar`, `BottomNavigationBar`, `CupertinoTabScaffold`, `AppBar`, `CupertinoNavigationBar`
- `Navigator`, `Router`, `PopScope`, `BackButton`, `CupertinoPageRoute`
- `showDialog`, `showModalBottomSheet`, `CupertinoModalPopup`, `MenuAnchor`, `PopupMenuButton`
- `ScrollView`, `ListView`, `CustomScrollView`, `PageView`, `RefreshIndicator`, `ScrollPhysics`
- `GestureDetector`, `InkWell`, `InkResponse`, `Listener`, `MouseRegion`, `FocusableActionDetector`
- `TextField`, `TextFormField`, `FocusNode`, `FocusScope`, `TextInputAction`, `keyboardType`
- `Semantics`, `Focus`, `Shortcuts`, `Actions`, `MediaQuery.disableAnimations`
- `HapticFeedback`, `SystemSound`
- `SelectableText`, `SelectionArea`, `Clipboard`
- fixed heights, bottom-positioned controls, manually applied inset padding
- platform branches using `Platform.isIOS`, `defaultTargetPlatform`, or `kIsWeb`

Use read-only targeted search. Do not install packages or modify files during recon.

#### Recon output
Summarize:
- SDK and platform targets.
- Design language and navigation structure.
- System UI, safe-area, keyboard, and scroll conventions.
- Input modalities supported.
- Existing platform-specific components.
- Relevant constraints and documented decisions.

### Phase 2 — Diagnose by symptom

Start from what the user reports. If auditing proactively, inspect each category and report only verified mismatches.

| Symptom | Flutter areas to inspect |
|---|---|
| Content under notch / home indicator | `SafeArea`, `MediaQuery.padding/viewPadding`, edge-to-edge, system bar configuration |
| Bottom bar/action obscured | Scaffold geometry, bottom safe area, keyboard insets, `extendBody` |
| Keyboard covers input or CTA | `viewInsets`, `resizeToAvoidBottomInset`, scroll-to-focus, `FocusScope`, keyboard actions |
| Layout jumps when keyboard opens | Inset ownership, nested `Scaffold`s, manual padding plus automatic resize |
| Back button/gesture feels wrong | `PopScope`, navigator stack, router, nested navigators, predictive back |
| Scroll feels non-native | `ScrollPhysics`, platform defaults, nested scrollables, overscroll policy |
| Pull-to-refresh triggers unexpectedly | `RefreshIndicator`, scroll notification depth, scrollable ownership |
| Carousel fights page scroll | Gesture arena, `PageView`, horizontal scrollables, drag recognizers |
| Tap has no feedback | `InkWell`/`InkResponse`, button style, custom gesture surfaces |
| Hover/focus missing on tablet/desktop | `MouseRegion`, `FocusableActionDetector`, focus states, shortcuts |
| Controls too small | Material/Cupertino target sizes, hit testing, `GestureDetector` bounds |
| Text selection behaves oddly | `SelectableText`, `SelectionArea`, control vs content semantics |
| Dialog/sheet looks wrong | Platform component, safe area, drag behavior, keyboard/insets |
| Status/navigation bar mismatches | `SystemChrome`, `AnnotatedRegion`, contrast, edge-to-edge |
| App feels different on iOS and Android | Platform conventions, route transitions, scroll physics, system UI |
| Works in emulator, fails on phone | Hardware-only verification list: keyboard, gestures, insets, haptics, OEM behavior |

### Phase 3 — Apply the smallest correct fix

When the user asks to implement:
- Make the smallest change that resolves the symptom.
- Prefer a Flutter/platform primitive over custom plumbing.
- Keep the existing architecture and design system.
- Avoid unrelated refactors.
- Preserve semantics, text scaling, keyboard access, and supported input modalities.
- Do not add a dependency unless necessary and approved.
- Explain any platform-specific behavior rather than hiding it behind arbitrary conditionals.

When the user asks for an audit only, do not modify code. Report:
- Symptom and evidence.
- Exact file/line.
- Likely cause.
- Proposed fix.
- What requires device verification.

## The Fixes

### 1. Content under the notch, status bar, or home indicator

Flutter-native apps do not use CSS `viewport-fit` or `env(safe-area-inset-*)`. Use Flutter's safe-area and inset model.

Prefer:
```dart
SafeArea(
  child: pageContent,
)
```

For edge-to-edge layouts, intentionally paint behind system areas while padding interactive content:
```dart
Scaffold(
  extendBodyBehindAppBar: true,
  body: Stack(
    children: [
      const Positioned.fill(child: Background()),
      SafeArea(child: Content()),
    ],
  ),
)
```

Rules:
- Use `SafeArea` for content that must avoid system intrusions.
- Use `MediaQuery.padding` or `viewPadding` when custom geometry genuinely needs the inset values.
- Understand the difference: `padding` reflects areas currently obstructed; `viewPadding` represents persistent system insets; `viewInsets` typically represents the keyboard.
- Avoid adding safe-area padding twice (for example, `SafeArea` plus manually copied top padding).
- For bottom navigation, sheets, and fixed actions, decide whether the component itself owns the bottom inset.
- Verify edge-to-edge behavior on recent Android versions and on iPhones with different sensor/home-indicator layouts.

### 2. System bars do not match the screen

Use `SystemChrome` or `AnnotatedRegion<SystemUiOverlayStyle>` intentionally.

Inspect:
- Status bar icon brightness/contrast against the actual top background.
- Navigation bar contrast and divider behavior on Android.
- Edge-to-edge configuration and OS-version behavior.
- Whether a route changes the system-bar style and restores it on exit.
- Whether scrolling content changes the background behind a transparent status bar.
- Dark/light theme transitions.

Prefer route- or subtree-scoped configuration when styles vary by screen. Avoid a single global style if different screens need different contrast.

Do not assume system bar behavior is identical across Android OEMs or iOS versions. Verify on devices.

### 3. Keyboard covers content or causes layout jumps

Keyboard behavior must be deliberate, not a last-minute padding patch.

Inspect:
- `Scaffold.resizeToAvoidBottomInset`.
- `MediaQuery.viewInsets.bottom`.
- Whether the screen is scrollable when the keyboard opens.
- Whether focused fields can scroll into view.
- Whether a bottom CTA should move above the keyboard, remain behind it, or be hidden.
- Nested `Scaffold`s and duplicated inset handling.
- `TextInputAction`, `FocusNode`, and next/done behavior.
- Validation and error messages that change field height.
- Keyboard dismissal via scroll, explicit action, or tapping outside.

Typical pattern:
```dart
Scaffold(
  resizeToAvoidBottomInset: true,
  body: SafeArea(
    child: SingleChildScrollView(
      padding: const EdgeInsets.all(16),
      child: FormContent(),
    ),
  ),
)
```

This is a starting pattern, not a universal recipe. For fixed bottom actions, use the scaffold's inset behavior or a single explicit `viewInsets` owner; do not stack both and double the offset.

Avoid:
- Hardcoded keyboard heights.
- Assuming the keyboard is always the same height.
- Fixed-height forms that overflow when text scales.
- Dismissing the keyboard on every tap indiscriminately.
- Repeated `MediaQuery.removeViewInsets` usage without a clear layout reason.

### 4. Touch targets and press feedback

Native-feeling controls acknowledge the user's input immediately and clearly.

- Prefer `FilledButton`, `TextButton`, `IconButton`, `InkWell`, or established app controls over raw `GestureDetector` for ordinary actions.
- Preserve minimum interactive target guidance: approximately 48×48 logical pixels for Material touch targets; follow Cupertino/platform conventions where applicable.
- If the visual icon is smaller, enlarge the hit area without distorting the visual.
- Ensure pressed, focused, hovered, disabled, and selected states are distinguishable.
- Use `InkWell` only when its Material ink behavior fits the design and its `Material` ancestor/clip is correct.
- Custom controls need explicit visual press feedback and semantics.
- Do not add a scale animation to every control by default. Use the motion skill if a custom animation is warranted.
- Haptics are optional and sparse; never use them as the only feedback.

Avoid:
- Tiny `GestureDetector` hit regions.
- `onTap` surfaces with no visible state response.
- Double feedback (ripple plus an unrelated strong scale) unless intentionally designed.
- Haptic feedback for every tap, scroll, or frame.

### 5. Hover, pointer, and keyboard states

Flutter supports multiple input modalities. Do not assume a phone-only interaction model if tablets, desktop, or web are targets.

- Use `MouseRegion` for pointer-specific hover behavior.
- Use `FocusableActionDetector`, `Focus`, `Shortcuts`, and `Actions` for keyboard navigation and activation.
- Keep touch press feedback independent of hover.
- Avoid using screen width as the sole test for mouse capability.
- Ensure focus indicators remain visible and are not replaced by hover.
- Test tab order, Enter/Space activation, Escape dismissal, and arrow-key behavior where appropriate.
- Use platform-adaptive cursor behavior for interactive desktop controls.

For touch-only targets, hover may be absent; this is normal. Do not build fake hover states for touch devices.

### 6. Scroll physics and overscroll

Flutter's `ScrollPhysics` controls native-feeling scroll behavior. Prefer platform defaults unless the product intentionally uses a custom experience.

- `BouncingScrollPhysics` is associated with iOS-style overscroll.
- `ClampingScrollPhysics` is associated with Android-style edge behavior.
- `ScrollBehavior` can unify or customize behavior, but do not override it globally without a product-wide reason.
- Avoid hardcoded physics on every scrollable when a consistent app-level policy is intended.
- Test nested scrollables, refresh controls, horizontal carousels, and scroll-to-focus together.
- Use `RefreshIndicator` with the correct scroll notification depth and scrollable configuration.
- Avoid custom gesture listeners that prevent the scrollable from receiving pointer events.
- Preserve expected platform overscroll affordances unless a canvas, sheet, or immersive surface has a clear reason to own them.

### 7. Gestures compete or feel wrong

Flutter's gesture arena determines which recognizer wins. Fix ownership rather than layering arbitrary gesture callbacks.

Inspect:
- Horizontal `PageView` inside vertical scrolling.
- Nested `GestureDetector`s with overlapping drag recognizers.
- `Dismissible` rows inside scrollables.
- Draggable sheets and their scrollable child.
- Back-swipe gestures near screen edges.
- Gesture cancellation and pointer handoff.
- Whether a control responds to tap after a drag.
- Whether custom `Listener` code bypasses gesture semantics.

Prefer:
- `PageView` for page swiping.
- `Dismissible` for standard swipe-to-dismiss list items.
- `DraggableScrollableSheet` for resizable sheet behavior when it fits.
- `RefreshIndicator` for standard pull-to-refresh.
- Native route/sheet patterns where available.

Do not use `GestureDetector` to recreate a standard component unless the product requires a materially different interaction.

### 8. Navigation and back behavior

Navigation is a platform contract.

- Use the app's router/Navigator consistently.
- Preserve Android system back and predictive-back expectations.
- Preserve iOS back-swipe behavior where the route type supports it.
- Use `PopScope` for back interception when necessary; do not casually swallow back events.
- Handle nested navigators and tab stacks intentionally.
- Ensure dialogs, menus, and sheets dismiss in the expected order.
- Avoid custom full-screen transitions that fight platform navigation unless explicitly part of the design.
- Verify deep links, restoration, and back-stack behavior when applicable.

### 9. Dialogs, menus, and sheets

Prefer standard components when they match the intended platform behavior:
- Material `showDialog`, `showModalBottomSheet`, `MenuAnchor`, `PopupMenuButton`.
- Cupertino `CupertinoAlertDialog`, `CupertinoActionSheet`, and Cupertino navigation patterns.

Check:
- Safe-area handling.
- Keyboard insets and text fields.
- Dismiss by barrier, drag, back, Escape, or explicit close as appropriate.
- Scrollable content and large text.
- Sheet drag handle and scroll handoff.
- Whether a sheet should be modal or a route.
- Platform presentation and corner/shape conventions.
- Focus placement and semantics.

Do not rebuild a platform sheet merely to match a screenshot if native interaction behavior is lost.

### 10. Text input and selection

- Use correct `keyboardType` and `TextInputAction`.
- Use autofill hints where appropriate.
- Set capitalization and autocorrect behavior intentionally.
- Keep text selectable when it is content; use `SelectableText` or `SelectionArea`.
- Do not make the entire screen unselectable to prevent selection on buttons.
- Use semantics and labels for custom input controls.
- Support text scaling, long localized strings, and validation states.
- Test password fields, OTP/code entry, email/phone input, and multiline fields on real keyboards.
- Ensure focus traversal works with hardware keyboards.

### 11. Haptics and system feedback

Flutter provides `HapticFeedback` and `SystemSound`, but behavior varies by platform and hardware.

Use sparingly:
- Selection changes where a tactile detent is meaningful.
- A committed drag or snap.
- Important success/error feedback when visual feedback alone is insufficient.

Rules:
- Trigger at the causal interaction moment, not after an unrelated delay.
- One haptic per meaningful action.
- Always pair with visible feedback.
- Respect user/platform settings where applicable.
- Avoid haptics on scroll, entrance animations, repeated rebuilds, or every button tap.
- Verify on physical devices; emulators cannot establish tactile quality.

### 12. Accessibility and reduced motion

- Preserve system text scaling; use flexible constraints and scrollable layouts rather than disabling it.
- Use `MediaQuery.disableAnimations` or the project's established reduced-motion policy.
- Remove or reduce nonessential translation, scale, parallax, and looping effects when reduced motion is enabled.
- Preserve opacity/color changes that communicate state.
- Ensure state changes are understandable without animation.
- Provide semantic labels, roles, and values for custom controls.
- Ensure focus order and keyboard activation.
- Check contrast and focus visibility.
- Test large text, screen readers, and reduced-motion settings.

### 13. Flutter web on mobile (only when web is a target)

Flutter web has browser-specific behavior that does not apply to native Flutter apps. Inspect the web renderer, browser support matrix, and app shell before making web-specific changes.

- Verify viewport and browser chrome behavior in the actual target browsers.
- Inspect `web/index.html`, manifest, theme color, and browser meta tags when symptoms concern browser chrome or installed PWA behavior.
- Safe areas may require web-specific CSS/environment integration in the HTML shell; Flutter `SafeArea` alone may not cover every browser/PWA edge-to-edge configuration.
- Input focus zoom, browser overscroll, text selection, and browser tap behavior are browser concerns; do not assume native Flutter APIs control them.
- Use CSS only in the web layer and keep it scoped to web.
- Test installed PWA/standalone mode separately from a browser tab.
- Never apply browser-only workarounds to iOS/Android native builds.

## Baseline Checklist

Before shipping a mobile-facing Flutter app, verify:

- [ ] Safe areas and edge-to-edge behavior are intentional on all supported device shapes.
- [ ] Status/navigation bars have correct contrast for light and dark surfaces.
- [ ] Bottom navigation and fixed actions avoid system insets.
- [ ] Keyboard does not cover focused fields or essential actions.
- [ ] Text scaling and long content do not overflow.
- [ ] Touch targets are adequate and press feedback is visible.
- [ ] Pointer hover and keyboard focus work on supported non-touch platforms.
- [ ] Scroll physics match the platform/product convention.
- [ ] Pull-to-refresh and nested scrolling do not conflict.
- [ ] Back navigation and gestures behave as expected.
- [ ] Dialogs/sheets handle safe areas, keyboard, and dismissal correctly.
- [ ] Text selection is available for content and suppressed only for controls when appropriate.
- [ ] Haptics are sparse and verified on hardware.
- [ ] Reduced motion and screen-reader behavior are covered.
- [ ] Flutter web-specific behavior is tested separately if web is supported.
- [ ] Real-device verification is complete on representative iOS and Android hardware.

## Never Ship

| Never | Instead |
|---|---|
| Hardcoded status-bar or home-indicator padding | `SafeArea` / `MediaQuery` insets with clear ownership |
| Double safe-area padding | One explicit inset owner |
| Hardcoded keyboard height | `viewInsets` and scroll-to-focus |
| Globally disabling text scaling | Flexible layout and scrollable content |
| Raw gesture handlers for ordinary buttons | Flutter buttons / `InkWell` / established controls |
| Tiny hit regions | Adequate target size and hit area |
| Custom scroll physics everywhere | Platform defaults or a deliberate app-wide policy |
| Swallowing system back casually | Router/Navigator conventions and `PopScope` only when needed |
| Haptics as the only feedback | Visual state plus sparse haptic feedback |
| Treating hover as touch feedback | Press states plus optional pointer hover |
| Device detection from screen width alone | Platform and input-capability-aware behavior |
| Native Flutter fixes applied to web-only bugs | Scoped web-shell/CSS fixes |
| Calling it native based only on emulator screenshots | Real-device verification |

## Output

### When implementing

Make the changes. Then, in a few lines:
- **Symptom and reason** — what was wrong and why.
- **What changed** — file and platform behavior.
- **Device checks** — what was verified and what still needs hardware.

### When auditing

Return a concise table:

| Priority | Symptom | Location | Evidence | Proposed fix | Verification |
|---|---|---|---|---|---|

Separate source-verifiable findings from hardware-only checks. Do not pad the audit with generic best practices that have no observed issue.

### When planning

Provide implementation-ready steps with exact files, scope, and a hardware verification checklist. Do not change source code unless explicitly asked.

## Tone

Opinionated, practical, and brief. Most platform fixes should use a native Flutter primitive or a small targeted change. When the honest answer is “this needs a real device,” say so rather than claiming it is fixed.
