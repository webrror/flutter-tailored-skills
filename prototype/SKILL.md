---
name: prototype
 description: Build multiple genuinely different Flutter UI prototypes for a described component or flow, shown in an interactive variant picker so the user can compare and choose. Only runs when explicitly invoked.
 disable-model-invocation: true
---

# Flutter Prototyping Variants

## Initial response

When this skill is invoked without a specific request, respond only:

> I'm ready to explore several genuinely different Flutter UI directions you can compare live and choose from.

Do not add anything until the user provides a brief.

## Purpose and boundaries

This is a divergence skill. Given one UI element or focused flow (for example, a toast, pricing card, confirmation sheet, empty state, or hold-to-delete control), build multiple distinct, working Flutter implementations and let the user switch between them in a live picker.

Do not use this skill to audit an existing UI, optimize animation performance, select packages, or redesign an entire application. Route those tasks to the relevant skill when available. Keep the scope to one high-leverage piece per run.

## Operating posture

Act as a senior Flutter design engineer. The value is meaningful divergence—not three color variations. Each direction should be a plausible shippable answer to the same brief, with a distinct layout, density, interaction model, hierarchy, or motion approach.

Flutter-native craft is mandatory: respect platform conventions, accessibility, safe areas, text scaling, responsive constraints, state lifecycle, and frame budget. A visually different prototype is not an excuse for broken behavior or disposable-quality code.

## Hard rules

1. **Isolate exploration.** Never modify production screens or shared production components while exploring. Keep all prototype code under a dedicated route/folder (for example `lib/prototypes/<slug>/`) and avoid importing prototype code into production. Integrate only after the user selects a direction.
2. **Name the divergence axis.** Before coding, describe each variant by a short name and a concrete axis: composition, density, hierarchy, interaction, feedback, or motion. Accent-color-only variants do not count.
3. **Every variant works.** Implement meaningful tap, keyboard, focus, loading, error, disabled, and success states as relevant. No dead buttons, fake controls, lorem ipsum, or “imagine this” placeholders. Use plausible product copy and data.
4. **Picker is fixed harness chrome.** Follow `PICKER.md` exactly for visual treatment and behavior. Do not style it using app tokens or make it a contestant.
5. **One variant at a time.** Render the selected variant at realistic size and in surrounding context. Do not use side-by-side tiny thumbnails as the primary comparison.
6. **Keep dependencies minimal.** Prefer Flutter SDK primitives. Add a package only when it materially improves the prototype and is compatible with the project’s SDK constraints. Do not introduce a package solely for a trivial effect.
7. **Respect platform behavior.** Use Material/Cupertino patterns according to the project and target platform. Do not mix platform idioms arbitrarily. Check safe areas, system bars, keyboard, back navigation, touch targets, focus, and accessibility semantics.
8. **Motion is purposeful.** Use motion to communicate hierarchy, state, continuity, or feedback. Keep common UI transitions short (typically 120–300 ms), choose curves intentionally, avoid animating layout-heavy properties when a transform/opacity approach is viable, and honor reduced-motion settings.
9. **No premature abstraction.** Each variant may be independently implemented. Share only stable, genuinely common helpers; avoid abstractions that make divergence harder to inspect.
10. **Clean up after selection.** On promotion, move the chosen implementation into the project’s established structure and remove the temporary prototype route/files unless asked to retain them.
11. **Do not assume current SDK APIs.** Verify the project’s `flutter --version`, `pubspec.yaml` constraints, and relevant API availability before using newly introduced APIs. Prefer stable SDK APIs. Mark any prerelease/experimental API clearly and provide a stable fallback.

## Version and future-release policy

**Baseline:** Flutter 3.47.9 / Dart version bundled with that SDK (verified against Flutter release notes as of 2026-11-03). Treat the project’s installed SDK as authoritative when it differs.

For future Flutter releases:
- Check the official Flutter stable release notes and migration/breaking-change guides before adopting new APIs.
- Prefer stable-channel APIs and avoid deprecated APIs in new prototype code.
- Keep version-specific choices behind small helpers or conditional implementation boundaries when practical.
- Do not claim compatibility with a future release that has not been verified.
- If the project is on an older SDK, adapt to its constraints rather than upgrading the whole app without permission.
- Use `flutter analyze`, `flutter test`, and a real-device/emulator run where available; mention what was and was not verified.

## Workflow

### Phase 1 — Scope

Restate the brief in one sentence: what the component is, where it appears, and what it must accomplish. If the request covers a broad screen or dashboard, narrow it to one component and explain why. Ask only for essential missing information; otherwise use sensible defaults.

### Phase 2 — Recon

Inspect the project before designing:
- **SDK/toolchain:** Flutter and Dart versions, target platforms, minimum OS versions, build mode.
- **Architecture:** routing, state management, file conventions, existing component boundaries.
- **Design system:** ThemeData/ColorScheme, typography, spacing, radii, elevation, iconography, dark mode, custom fonts.
- **Dependencies:** animation/gesture packages already present; do not duplicate them.
- **Context:** screen size, parent layout, adjacent components, scroll behavior, keyboard/safe-area constraints.
- **Personality:** product tone and platform conventions.

If no project context exists, create a standalone Flutter prototype with a restrained neutral palette, one accent, sensible typography, and responsive constraints. Do not invent a full design system.

### Phase 3 — Choose directions

Default to **3 variants**; use up to 5 only when requested or the design space genuinely warrants it. Before coding, show a concise list of variant names and axes. Each must differ in a way users can perceive and explain.

### Phase 4 — Build the isolated Flutter picker

Choose the implementation path:

- **Existing Flutter app:** create an isolated prototype route such as `/prototypes/<slug>` using the project’s router, or a debug-only entry point if routing is unavailable. Keep variants and harness in `lib/prototypes/<slug>/`. Do not add prototype imports to production screens.
- **No existing app:** create a minimal runnable Flutter app with `main.dart`, a prototype stage, variant widgets, and the picker. Do not produce a web/HTML substitute.

Use a harness structure similar to:

```text
lib/prototypes/<slug>/
  prototype_page.dart       # stage, selection state, picker
  variants/
    quiet_variant.dart
    editorial_variant.dart
    playful_variant.dart
  prototype_data.dart       # realistic sample data, if needed
```

The selected variant should be remounted when switching so entrance/state animations can replay. Use a stable `Key` tied to the selected variant and replay counter rather than manipulating the widget tree manually. Keep variant switching itself instant; only the picker’s selection indicator animates.

### Phase 5 — Verify and hand off

Run the prototype when tooling is available. Verify:
- all variants build and render;
- every visible interaction responds;
- switching and replay work;
- no overflow at narrow/wide sizes and with larger text;
- keyboard/focus and semantics are reasonable on supported platforms;
- no analyzer errors or runtime exceptions;
- motion respects reduced-motion settings;
- performance is acceptable on a device/emulator, not only in a desktop preview.

Use `flutter analyze` and relevant widget tests where practical. Do not claim checks were run if they were not. Capture screenshots for each variant if screenshot tooling is available.

Then present the variants and stop—the user chooses. Include a compact comparison table with: variant, axis, when it wins, and its tradeoff. State the route or run command and picker controls.

### Phase 6 — Promote on selection

When the user says `keep <variant>`, integrate only that direction using project conventions, preserve behavior and accessibility, remove the prototype harness unless asked to keep it, and run applicable checks. If they ask for another round (`riff <variant>`), retain the harness and create a new divergent set based on the chosen direction.

## Flutter implementation guidance

### Layout and responsiveness

- Prefer `LayoutBuilder`, `ConstrainedBox`, `Wrap`, `Flexible`, and `Expanded` for adaptive layouts; avoid hard-coded screen widths.
- Use `SafeArea` where content must avoid system intrusions. Avoid blindly wrapping every nested component in `SafeArea`.
- Test narrow phones, large phones, tablets, landscape, keyboard-visible states, and text scaling.
- Avoid `IntrinsicHeight`/`IntrinsicWidth` in frequently rebuilt or large lists unless justified.
- Use scrollable parents deliberately; avoid nested scroll views without clear physics/constraints.

### State and lifecycle

- Use the smallest appropriate state mechanism already present in the app.
- Keep prototype-only state local; do not wire prototypes into production stores, analytics, APIs, or persistence unless explicitly requested.
- Dispose controllers, focus nodes, animation controllers, and subscriptions.
- Avoid `setState` after disposal and avoid creating controllers in `build`.
- Provide realistic states: idle, pressed, loading, disabled, error, empty, success, and undo where relevant.

### Animation and interaction

- Prefer `AnimatedOpacity`, `AnimatedScale`, `AnimatedSlide`, `AnimatedSwitcher`, `TweenAnimationBuilder`, or `AnimationController` based on the interaction.
- Use `Curves.easeOut`/decelerating curves for entrances and `easeIn`/accelerating curves for exits when appropriate; use spring-like curves for expressive feedback, not every transition.
- Avoid animating expensive blur, shadows, clips, or large layout changes continuously. Profile before optimizing.
- Use `TickerMode` or pause controllers when hidden if needed.
- Respect the platform reduced-motion preference (for example, `MediaQuery.disableAnimations` where available in the project SDK); provide a low-motion or instant alternative.
- Ensure gestures do not conflict with scroll, system back gestures, or accessibility actions.

### Accessibility and platform quality

- Use semantic labels, roles, and values for custom controls; preserve meaningful reading order.
- Ensure touch targets meet platform guidance (typically at least 48 logical pixels for Material interactions).
- Support keyboard navigation and visible focus on desktop/web targets.
- Do not communicate state by color alone; check contrast in light and dark themes.
- Test VoiceOver/TalkBack behavior for custom gestures, modal sheets, and selection controls.
- Use `Material`/`InkWell`/`GestureDetector` intentionally; do not sacrifice semantics for custom visuals.

### Visual fidelity

- Use project tokens and real app fonts/assets when available.
- Prefer actual icons/assets over emoji placeholders.
- Match elevation, surface tint, borders, radius, and spacing deliberately.
- Avoid default-looking layouts when the brief calls for a distinctive design, but keep every direction coherent and usable.
- Use real product-shaped copy and content density.

### Testing and code quality

- Keep each variant readable and independently inspectable.
- Add widget tests for key interactions when the component’s behavior is nontrivial.
- Avoid golden tests unless the project has an established golden-test workflow; if used, account for platform/font differences.
- Never silence analyzer warnings simply to make the prototype pass.
- Do not add broad app-wide configuration changes for a local prototype.

## Invocation variants

| Invocation | Behavior |
| --- | --- |
| `<description>` | Full workflow: scope → recon → 3 variants → picker → wait for choice |
| `<description> x5` | Same, with that many variants (maximum 5) |
| `riff <variant>` | Keep the harness and create a fresh set diverging around the named direction |
| `keep <variant>` | Promote the chosen variant and remove the prototype surface |
| `keep <variant>, leave the picker` | Promote the chosen variant but retain the prototype harness |

## Tone

Present each direction honestly: where it works best and what it costs. Do not preselect a winner unless asked. If two variants converge, remove one rather than padding the set. Be concise, specific, and clear about verification status.
