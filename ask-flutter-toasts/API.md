# Flutter Notification API Reference

This reference supports `ask-flutter-notifications/SKILL.md`.

**Compatibility target:** Flutter 3.47.9 stable / Dart 3.13.7+.  
**Important:** Package APIs change. Confirm the project's resolved version in `pubspec.lock` and consult that version's documentation before relying on exact signatures.

This is a Flutter-oriented reference, not a Sonner API mapping. There is no universal Flutter `toast.promise()` or universal toast update-by-id API.

## 1. Flutter SDK: SnackBar

Use for ordinary in-app feedback associated with a `Scaffold`.

```dart
ScaffoldMessenger.of(context).showSnackBar(
  SnackBar(
    content: const Text('Profile saved'),
    duration: const Duration(seconds: 3),
    action: SnackBarAction(
      label: 'UNDO',
      onPressed: undoSave,
    ),
  ),
);
```

Relevant APIs:

- `ScaffoldMessenger`
- `ScaffoldMessengerState.showSnackBar`
- `ScaffoldMessengerState.hideCurrentSnackBar`
- `ScaffoldMessengerState.removeCurrentSnackBar`
- `ScaffoldMessengerState.clearSnackBars`
- `SnackBar`
- `SnackBarAction`
- `SnackBarThemeData`

### Replacing/updating a visible SnackBar

`ScaffoldMessengerState.showSnackBar` queues SnackBars by default. To replace the current one, hide/remove it before showing the next, or use `clearSnackBars()` when the queue should be cleared.

```dart
final messenger = ScaffoldMessenger.of(context);
messenger.hideCurrentSnackBar();
messenger.showSnackBar(
  const SnackBar(content: Text('Saved')),
);
```

Use `removeCurrentSnackBar()` for immediate removal; `hideCurrentSnackBar()` allows the exit animation.

### Async result pattern

```dart
Future<void> saveProfile(BuildContext context) async {
  final messenger = ScaffoldMessenger.of(context);

  messenger.showSnackBar(
    const SnackBar(
      content: Text('Saving…'),
      duration: Duration(days: 1),
    ),
  );

  try {
    await repository.saveProfile();

    if (!context.mounted) return;
    messenger.hideCurrentSnackBar();
    messenger.showSnackBar(
      const SnackBar(content: Text('Profile saved')),
    );
  } catch (_) {
    if (!context.mounted) return;
    messenger.hideCurrentSnackBar();
    messenger.showSnackBar(
      const SnackBar(content: Text('Could not save profile')),
    );
  }
}
```

This is a basic pattern, not a universal recommendation to use a long-duration SnackBar for loading. For long operations, inline progress is often better. For overlapping requests, use a dedicated coordinator or package handle so one request cannot dismiss another request's message accidentally.

## 2. MaterialBanner

Use for a persistent page-level message that may contain an action.

```dart
ScaffoldMessenger.of(context).showMaterialBanner(
  MaterialBanner(
    content: const Text('Offline. Some features may be unavailable.'),
    actions: [
      TextButton(
        onPressed: retryConnection,
        child: const Text('RETRY'),
      ),
      TextButton(
        onPressed: () {
          ScaffoldMessenger.of(context).hideCurrentMaterialBanner();
        },
        child: const Text('DISMISS'),
      ),
    ],
  ),
);
```

Relevant APIs include `MaterialBanner`, `showMaterialBanner`, and `hideCurrentMaterialBanner`.

## 3. OverlayEntry (custom implementation)

Use only when built-in surfaces and maintained packages cannot meet the requirements.

Core responsibilities:

- Insert the entry into the correct `OverlayState`.
- Position it with safe-area and keyboard awareness.
- Animate entrance/exit.
- Remove it after dismissal.
- Cancel timers and dispose controllers.
- Prevent duplicate entries and stale callbacks.
- Handle semantics, hit testing, pointer behavior, and route changes.
- Decide whether it should survive navigation.
- Avoid retaining stale `BuildContext` values.

Minimal structural example:

```dart
final overlay = Overlay.of(context);
late final OverlayEntry entry;

entry = OverlayEntry(
  builder: (context) => Positioned(
    top: MediaQuery.paddingOf(context).top + 16,
    left: 16,
    right: 16,
    child: Material(
      elevation: 6,
      borderRadius: BorderRadius.circular(12),
      child: const Padding(
        padding: EdgeInsets.all(16),
        child: Text('Saved'),
      ),
    ),
  ),
);

overlay.insert(entry);

// Later, when dismissed:
entry.remove();
entry.dispose();
```

This is intentionally not a complete toast system. A production implementation needs lifecycle, timer, animation, safe-area, semantics, and deduplication handling.

## 4. Package: toastification

Package page: https://pub.dev/packages/toastification

Consider it when you need customizable in-app notifications, queueing, custom builders, actions, dismissal, or global configuration.

Typical setup pattern (verify the installed version):

```dart
import 'package:toastification/toastification.dart';

MaterialApp(
  builder: (context, child) {
    return ToastificationWrapper(child: child!);
  },
  home: const HomePage(),
);
```

Typical show pattern:

```dart
toastification.show(
  context: context,
  type: ToastificationType.success,
  title: const Text('Profile saved'),
  autoCloseDuration: const Duration(seconds: 4),
);
```

Concepts to verify in the installed version:

- `ToastificationWrapper` for context-free usage.
- `toastification.show` for predefined styles/types.
- `toastification.showCustom` for custom widget content.
- Notification item/handle returned by `show`.
- Dismiss by item or id, dismiss all, and queue behavior.
- `ToastificationType`, `ToastificationStyle`.
- Alignment, animation builder/duration, close button, drag dismissal.
- Global configuration provider and theme overrides.
- Callback behavior and whether updates reset the auto-close timer.

Do not assume a `promise` helper exists. Use explicit Dart async handling and the package's documented item/update/dismiss API.

## 5. Package: fluttertoast

Package page: https://pub.dev/packages/fluttertoast

Consider it for simple platform toast behavior or a context-based custom toast.

Important distinctions:

- Context-free native toast APIs have platform-specific styling/position limitations.
- Context-based custom toast widgets provide more UI control but use Flutter's widget/overlay system.
- Platform support differs by mode.
- Native Android toast customization has OS restrictions; verify current package documentation.
- Check queue, cancel, and custom toast removal behavior for the selected API.

Do not treat `Fluttertoast.showToast` and context-based `FToast` custom surfaces as interchangeable.

## 6. Theme and accessibility

For Material surfaces, inspect:

- `ThemeData`
- `ColorScheme`
- `SnackBarThemeData`
- `MaterialBannerThemeData`
- `TextTheme`
- `MediaQuery`
- `Semantics`
- `SafeArea`

For package notifications, check whether styling inherits the app theme or requires explicit configuration. Test light/dark themes, text scaling, contrast, screen readers, keyboard navigation, and platform behavior.

## 7. Troubleshooting checklist

| Symptom | Inspect |
|---|---|
| SnackBar not visible | Correct `ScaffoldMessenger`, route/scaffold scope, overlay/modal, context validity |
| Duplicate toast | Duplicate host, repeated event/listener, build-time side effect, retries |
| Toast behind sheet/dialog | Overlay ownership, navigator/overlay selection, z-order |
| Styles ignored | Package version, config provider scope, theme inheritance, custom builder |
| Loading never resolves | All async paths, stale id/handle, cancellation, exceptions |
| Dismisses wrong item | Identity/handle, overlapping operations, global dismiss calls |
| Queue floods | Deduplication, max-visible policy, queue strategy |
| Timer mismatch | Auto-close duration vs animation duration, lifecycle pause/resume |
| Layout clipped | Safe area, keyboard inset, text scaling, long localization |
| Gesture conflict | Scroll vs swipe-to-dismiss, hit testing, dismissal direction |
| Test flakes | Timer control, animation duration, host setup, pump/settle strategy |

## Version and dependency hygiene

- Run `flutter pub add <package>` to select a version compatible with the project's SDK.
- Inspect `pubspec.lock` for the resolved version.
- Avoid copying API signatures from a different major version.
- Keep package setup centralized and documented.
- Re-check package maintenance, platform support, and license before introducing a dependency.
