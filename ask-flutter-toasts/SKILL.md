---
name: ask-flutter-notifications
description: Guide for implementing, styling, managing, and troubleshooting transient notifications in Flutter. Choose between SnackBar, MaterialBanner, dialogs, platform-native toast APIs, overlay-based toast packages, and custom notification surfaces. Covers success/error/info/warning/loading/progress states, async operations, updating and dismissing notifications, actions, queues, placement, themes, accessibility, lifecycle, and common bugs. Compatible with Flutter 3.47.9 stable / Dart 3.13.7 and newer stable releases.
---

# Working With Notifications in Flutter

## Initial Response

When this skill is first invoked without a specific question, respond only with:

> I'm ready to help you design, implement, style, and troubleshoot notifications in Flutter.

Do not provide any other information until the user asks a question.

## Purpose

A guide for transient in-app notifications in Flutter. Use this skill when implementing feedback messages, toast-like surfaces, snackbars, loading notifications, action messages, or fixing notification behavior.

This is **not a Sonner port**. Sonner is a React library; Flutter has different widget, overlay, context, navigation, lifecycle, and platform APIs. Preserve the original skill's decision-making workflow, but use Flutter-native concepts and the selected package's real API.

For exact package-specific API names, setup, parameters, and limitations, read `API.md` and verify against the project's installed package version and current package documentation.

## Compatibility

**Target baseline: Flutter 3.47.9 stable / Dart 3.13.7 and newer stable releases.**

- Check the project's actual SDK constraint and package versions before writing imports or APIs.
- Prefer stable Flutter SDK features.
- Install packages with `flutter pub add <package>` or the project's package manager; do not guess compatible versions.
- Package APIs can change. Verify exact constructor names, enum values, and lifecycle behavior against the resolved version.
- Do not assume a package works identically on iOS, Android, web, desktop, and tests.
- Do not claim that a toast is a system notification. In-app transient surfaces and OS notifications are different systems.
- Avoid introducing a third-party dependency when `ScaffoldMessenger` or another Flutter primitive already satisfies the requirement.

## First decision: what kind of notification is this?

Choose the least intrusive surface that communicates the event.

| Need | Prefer |
|---|---|
| Brief confirmation or recoverable feedback within a page | `SnackBar` via `ScaffoldMessenger` |
| Persistent, page-level message with optional action | `MaterialBanner` |
| Important decision or blocking confirmation | `AlertDialog` / platform-appropriate dialog |
| Floating toast independent of Scaffold layout | A maintained toast/notification package or a carefully scoped `OverlayEntry` |
| Long-running operation with progress and a meaningful result | Inline progress first; optionally a persistent notification surface |
| System notification while app is backgrounded | Platform/local notification integration, not an in-app toast |
| Validation feedback tied to a field | Inline field error/helper text |
| State that must remain visible until resolved | Persistent banner/inline state, not an auto-dismiss toast |

Do not use a toast for critical information that users must read, act on, or recover from. A transient message should not be the only place where an error or important result is communicated.

## Setup principles

- For `SnackBar`, use `ScaffoldMessenger`; do not add a toast package.
- If using a package, follow its documented root/overlay setup exactly and mount its host once at the appropriate app level.
- Do not mount notification hosts independently on every route unless the package explicitly requires route-local scope.
- Keep notification presentation centralized when the app needs consistent styling, deduplication, queueing, and testability.
- Keep business logic independent from presentation. A repository/service should return a result; UI or an application-level notification coordinator decides whether and how to show feedback.
- Do not call UI presentation APIs from background isolates.
- Avoid storing a `BuildContext` for later use. Use a current context, a documented navigator/overlay key, or a notification service with a well-defined lifecycle.
- Do not use `context` after an async gap unless it is still valid (`context.mounted` / `State.mounted`).
- Do not assume a global key is necessary. Prefer local `ScaffoldMessenger.of(context)` when the call is naturally scoped to a page.

## Choosing the right notification

| Intent | Flutter pattern |
|---|---|
| Neutral, short message | `SnackBar` or package neutral toast |
| Success | Short confirmation; avoid celebratory motion for routine actions |
| Error | Explain what failed and, when possible, how to recover |
| Warning | Use only when attention or a decision is genuinely needed |
| Information | Keep concise and non-blocking |
| Loading | Persistent state with explicit completion/error update or inline progress |
| Progress | Show real progress only when measurable; otherwise use indeterminate progress |
| Undo / Retry / View | Action-bearing `SnackBar` or interactive toast with a clear action |
| Long content | Banner, dialog, or destination page—not a tiny toast |
| Destructive/critical result | Persistent, accessible feedback; never rely only on auto-dismiss |

Use semantic categories consistently. Do not use success styling for neutral events or red for ordinary information.

## Async operations: loading → result

For a request that can take noticeable time:

1. Decide whether the operation needs visible loading feedback at all. Very fast operations may not need a toast.
2. Show one loading surface only if it adds value.
3. Retain a stable notification identity/handle when the chosen package supports updating.
4. Replace loading with success or error rather than stacking a second unrelated message.
5. Dismiss or update the loading state in every completion path, including cancellation and exceptions.
6. Avoid showing success before the operation actually succeeds.
7. Prevent duplicate feedback when the same action is retried, triggered twice, or completed through multiple listeners.
8. For operations with meaningful progress, update a progress indicator at a sensible cadence—not on every frame or every tiny percentage change.

Flutter-native `SnackBar` can be replaced through `ScaffoldMessenger` APIs, but package-specific update-by-id semantics differ. Do not invent a universal `toast.promise` equivalent. Write explicit Dart async handling or use the selected package's documented API.

## Actions and callbacks

- Use actions only when the user has a meaningful next step: Undo, Retry, View, Open, or Dismiss.
- Keep the label short and specific.
- Ensure the action remains available long enough to use.
- Define whether activating the action dismisses the notification.
- Avoid multiple competing actions in a compact toast.
- Do not put essential navigation or irreversible actions behind an auto-dismiss message.
- Ensure action buttons are accessible to screen readers and keyboard/pointer users.
- Guard against callbacks firing after the owning page or object has been disposed.

## Duration, persistence, and dismissal

- Keep routine messages brief and short-lived.
- Increase duration for longer text or action-bearing messages.
- Do not auto-dismiss critical errors, authentication expiry, required decisions, or content users must read.
- Allow manual dismissal when a message persists.
- Support swipe dismissal only when it is discoverable and does not conflict with page scrolling.
- Avoid queueing many low-value notifications. Deduplicate repeated events and prioritize important messages.
- If several notifications occur together, prefer summarizing them or showing the most relevant one rather than flooding the screen.
- When a route changes, decide deliberately whether the notification should remain globally visible or disappear with that route.
- Handle app lifecycle changes if a countdown or progress bar must pause while the app is inactive.

## Placement and layout

- Avoid covering primary actions, navigation controls, text fields, keyboard content, or system gestures.
- Respect `SafeArea`, status bar, display cutouts, home indicator, and keyboard insets.
- Position messages near the relevant context when that improves comprehension; use global placement for app-wide feedback.
- Do not place a toast under a modal, bottom sheet, or route transition where it becomes invisible or inaccessible.
- Account for device size, orientation, text scaling, localization, and long messages.
- Use constraints and flexible layout rather than fixed heights.
- For top-positioned surfaces, ensure they do not collide with app bars or system UI.
- On desktop/web, consider pointer interaction, focus, window resizing, and whether the notification should be anchored to a viewport or route.

## Styling and theming — escalation ladder

Use the simplest customization that meets the design.

1. **Built-in surface:** Start with Material `SnackBar` or the selected package's default style.
2. **Theme-level styling:** Use `ThemeData`, `SnackBarThemeData`, `ColorScheme`, and typography tokens.
3. **Package configuration:** Use supported package parameters for placement, colors, borders, icons, duration, and animation.
4. **Custom builder:** Use a package's custom-widget API when its default shell cannot meet the design.
5. **Custom overlay:** Build a reusable overlay-based host only when requirements cannot be met by existing primitives/packages.

Rules:

- Keep typography, spacing, radius, colors, and elevation consistent with the app.
- Use semantic colors with sufficient contrast in light and dark themes.
- Avoid styling every notification independently; centralize tokens and variants.
- Do not copy a web toast's exact dimensions or CSS behavior into Flutter without adapting for logical pixels, text scaling, safe areas, and platform interaction.
- Do not overuse blur, shadows, gradients, or elaborate entrance animations.
- Make custom notifications work with localization and long text.
- Do not use color or icons as the only indicator of success/error/warning.
- Respect system theme changes when the notification is visible.

## Motion and interaction

Notifications are frequent feedback, so motion should be restrained.

- Prefer a short fade or small translation for routine entry/exit.
- Avoid bounce, large travel, rotation, or celebration for everyday messages.
- Do not animate layout properties continuously when a transform/opacity can achieve the same result.
- Use the package's built-in motion unless there is a clear design reason to replace it.
- Make dismissal interruptible and responsive.
- Do not block background interaction unless the notification is intentionally modal.
- Respect reduced-motion preferences where available; replace large movement with opacity or immediate state changes.
- If a toast has a progress bar, ensure the visual timer matches the actual dismissal timer.
- Pause timers on hover/press only if the package supports it reliably and the interaction makes sense on the target platform.
- Test gesture dismissal against scroll gestures and accessibility alternatives.

## Accessibility

- A transient notification must not be the only way to communicate an important state.
- Use appropriate semantics and live-region announcements where supported by the chosen widget/package.
- Avoid repeatedly announcing the same message due to rebuilds or duplicate events.
- Ensure text scales without clipping.
- Keep touch targets large enough for close/action controls.
- Provide sufficient contrast and meaningful labels.
- Do not require swipe gestures as the only dismissal or action path.
- Avoid rapid sequences of announcements that overwhelm screen-reader users.
- Test with TalkBack and VoiceOver on real devices.
- Respect reduced motion and system text settings.
- Consider keyboard focus and Escape/dismiss behavior on desktop/web.

## Troubleshooting

### Notification does not appear

Check:

- Is the correct `ScaffoldMessenger`, overlay, or package host mounted?
- Is the call made while the relevant widget tree is active?
- Is the notification being immediately dismissed or replaced?
- Is it hidden behind a route, dialog, sheet, keyboard, or system UI?
- Is a `BuildContext` stale after an async gap?
- Does the package require a wrapper, builder, navigator key, or overlay state?
- Is the selected platform supported by the chosen package?
- Is the call happening in a background isolate or non-UI context?

### Notification appears twice

Check:

- Is the host mounted more than once?
- Is the same event handled by both a widget and a global listener?
- Does a build method or effect trigger presentation repeatedly?
- Are retries, stream subscriptions, or state listeners duplicated?
- Is the same async completion handled in multiple places?
- Is deduplication needed for repeated identical events?

### Notification appears behind another surface

Check:

- Overlay ownership and route nesting.
- Navigator/Overlay selection.
- Modal routes and bottom sheets.
- Z-order and insertion timing.
- Safe-area and keyboard placement.
- Whether a root-level host is appropriate.

### Styling is ignored

Check:

- Whether the package uses a default theme/configuration provider.
- Whether local config overrides global config.
- Whether the app is rebuilding with a different theme.
- Whether a custom builder replaces the package's style parameters.
- Whether the desired property is supported in the installed package version.
- Whether text styles/colors are overridden by inherited theme values.

### Dismissal or queue behavior is wrong

Check:

- Notification identity and whether the correct item is being dismissed.
- Auto-close timer vs animation duration.
- Queue policy and max visible count.
- Whether updates reset the timer.
- Whether swipe/drag callbacks conflict with scrolling.
- Whether the app lifecycle pauses timers.
- Whether stale callbacks act on disposed state.

### Async loading toast gets stuck

Check:

- Success, error, cancellation, timeout, and early-return paths.
- Whether the operation throws before the result notification is updated.
- Whether the update uses the correct notification handle/id.
- Whether duplicate requests overwrite the active handle.
- Whether the notification service outlives the operation appropriately.

## Package selection

Do not default to a package without checking the requirement.

- **Flutter SDK:** `ScaffoldMessenger` / `SnackBar` for conventional in-app feedback.
- **`toastification`:** Consider when the app needs customizable floating notifications, queueing, actions, dismissal, custom builders, or global configuration. Verify the resolved version's API.
- **`fluttertoast`:** Consider for simple platform toast behavior or its context-based custom toast approach. Verify platform differences and limitations.
- **Other packages:** Evaluate maintenance, SDK constraints, platform coverage, accessibility, queue/update semantics, animation behavior, and license.
- **Custom `OverlayEntry`:** Last resort. Own insertion/removal, timers, lifecycle, semantics, safe areas, animation, hit testing, and cleanup explicitly.

Do not claim one package is universally best. Choose based on the user's requested behavior, current project dependencies, and target platforms.

## Output expectations

When implementing or troubleshooting:

1. State the selected notification pattern and why, briefly.
2. Provide working Flutter code using the actual chosen API.
3. Include setup only when needed; avoid repeating app-wide setup if it already exists.
4. Explain how loading/result updates, dismissal, and actions work.
5. Mention platform limitations or accessibility considerations that materially affect the implementation.
6. For bugs, identify the likely cause and give the smallest targeted fix.
7. If package API details are uncertain, inspect `API.md` and the installed package version rather than inventing parameters.

Keep answers practical. Do not write a notification architecture essay for a one-line `SnackBar`.

## Tone

Opinionated, concise, and implementation-focused. Use Flutter's built-in primitives when they are enough. Add a package only when it solves a real requirement. Make notifications useful, non-duplicative, accessible, and appropriately transient.
