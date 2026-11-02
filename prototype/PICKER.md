# Flutter Prototype Picker

**SDK baseline:** Flutter 3.47.9 stable (Dart bundled with that SDK), verified 2026-11-03. Treat this as a reference baseline only; inspect the project’s installed SDK and confirm APIs against official docs when implementing. The picker should use stable Flutter APIs and avoid experimental/deprecated APIs unless explicitly requested.

The picker is fixed harness chrome, not a design contestant. Keep its appearance consistent across prototypes and do not restyle it using app theme tokens. The Flutter implementation should visually approximate the original fixed dark-glass pill while respecting platform safe areas.

## Appearance

- Floating dark pill, horizontally centered, normally 24 logical pixels above the bottom safe-area edge.
- Dark translucent surface, subtle inner hairline, soft shadow, rounded to full pill shape.
- Compact system typography (approximately 13 logical pixels).
- Variant labels are buttons; selected label has a slightly brighter translucent background and white text.
- A narrow divider separates variants from replay, shown only if at least one variant has motion worth replaying.
- If the prototype occupies bottom-center (toast, dock, bottom sheet), place the picker near the top safe-area edge instead. Do not otherwise reposition it.
- Picker must remain above prototype content but must not overlap system UI or the keyboard.

## Flutter structure

Use SDK widgets first. Keep the picker self-contained and prototype-only; do not add dependencies for keyboard handling, selection, blur, or URL state unless the project already uses them and they provide a clear benefit. The implementation should compile against the project SDK constraint.

Use a `Stack` with the prototype stage filling available space and the picker positioned using `SafeArea` + `Align`/`Padding`. Prefer a `Material` surface with a dark translucent color, `ClipRRect` and `BackdropFilter` only if the blur is visually important and inexpensive. Provide a non-blurred fallback for platforms where blur is unsupported or too costly.

Suggested conceptual structure (adapt idiomatically to the project):

```dart
Stack(
  children: [
    Positioned.fill(
      child: KeyedSubtree(
        key: ValueKey('$selectedIndex-$replayNonce'),
        child: variants[selectedIndex].builder(context),
      ),
    ),
    SafeArea(
      child: Align(
        alignment: pickerAtTop
            ? Alignment.topCenter
            : Alignment.bottomCenter,
        child: PrototypePicker(...),
      ),
    ),
  ],
)
```

The code above is illustrative; use a real `WidgetBuilder`/variant model appropriate to the project. Avoid a `GlobalKey` per variant unless a genuine need exists.

## Behavior contract

The picker is a comparison tool, not a production navigation pattern. Keep its behavior predictable and scoped to the prototype route.

- Number keys `1–N` switch variants; Left/Right arrows move to previous/next; `R` replays the current variant.
- Ignore shortcuts when focus is in a text field, editable control, or when a modifier is held. Use `Focus`/`Shortcuts`/`Actions` or a scoped keyboard handler appropriate to the project and Flutter SDK.
- Tapping a variant selects it. Exactly one item is selected and exposes selected state through semantics.
- Switching remounts the selected variant so entrance animations rerun. The variant itself switches instantly; do not animate the whole stage.
- Replay remounts the current variant without changing selection.
- Selection persists in the URL as `?v=2` for Flutter Web when practical. For native targets, keep selection in memory unless the user asks for persistence. Do not add local storage or a persistence package for a prototype picker.
- Initial selection is URL value when valid, otherwise variant 1. Avoid an entrance animation on the picker indicator during initial placement.
- Keep keyboard controls scoped to the prototype route and avoid intercepting application-wide shortcuts.
- The replay control is conditional; omit it for entirely static variants.

## Implementation notes

- Use a `StatefulWidget` or the project’s existing state approach for `selectedIndex` and `replayNonce`.
- Use `AnimatedContainer` or a small `AnimationController` for the selected pill indicator (about 250 ms, strong ease-out). This animation is picker-only.
- For a simple and robust implementation, individual buttons may use selected-state `AnimatedContainer` rather than a measured sliding highlight, provided the overall visual treatment and behavior remain consistent.
- Use `Semantics`, `Tooltip`, and focusable controls. Provide a visible focus outline for keyboard users.
- Handle resize/orientation changes without stale measurements. If using a measured sliding indicator, recalculate after layout.
- Respect reduced-motion preferences by making the indicator change immediate.

## Verification checklist

Verify on the actual target platform(s) where possible. Flutter Web URL synchronization is optional and must not compromise native behavior.

- [ ] Every variant is reachable by tap and keyboard.
- [ ] Selection state is exposed to screen readers.
- [ ] `R` replays only when replay is available.
- [ ] Shortcut handling does not fire while editing text or when modifiers are held.
- [ ] Picker does not cover the component under review; top placement is used when needed.
- [ ] Picker respects safe areas and keyboard visibility.
- [ ] Selection indicator does not animate on initial load or when reduced motion is enabled.
- [ ] Variant switching is instant and the selected variant remounts.
