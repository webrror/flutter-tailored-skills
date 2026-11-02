---
name: break-ui-flutter
description: Stress-test Flutter widgets, screens, and flows with realistic worst-case data and environments. Map every rendered value, create edge-case fixtures, expose Demo/Worst case/Empty/One/Huge states through a development-only switch, inspect actual layouts, and report each break with a targeted fix. Use when asked to break, stress-test, find edge cases, or try worst-case data in a Flutter UI. For visual redesign use a design skill; for animation review use an animation skill.
---

# Breaking UI in Flutter

## Initial Response

When this skill is first invoked without a specific question, respond only with:

> I'm ready to stress-test your Flutter UI with realistic worst-case data and show you what breaks.

Do not provide any other information until the user asks a question.

## Purpose

An adversarial UI-testing skill. It does one thing: take a Flutter component or screen that appears correct with tidy demo data, find realistic worst cases for every value it renders, expose those cases beside the demo state, inspect what breaks, and report each issue with a targeted fix.

It does **not** redesign the component, critique visual taste, or review motion. It tests whether the existing UI survives plausible production data, state, screen sizes, localization, and accessibility settings.

This is a Flutter adaptation of the `break-ui` workflow. Keep its intent and discipline; replace web-specific implementation details with Flutter widgets, constraints, fixtures, routes, and tests.

## Compatibility

**Target baseline: Flutter 3.47.9 stable / Dart 3.13.7 and newer stable releases.**

- Inspect the project's actual Flutter/Dart constraints and dependencies before using APIs.
- Prefer stable SDK APIs and project conventions.
- Do not assume the project uses Material 3, a particular state-management library, or a router.
- Do not introduce a new package just to build a fixture toggle.
- Verify behavior on the target platforms and layouts; a passing widget test does not prove the screen is visually correct on device.
- Treat source data, validation schemas, backend constraints, and localization resources as evidence. Do not invent product limits.
- If a source has no defined limit, record it as **unbounded / not found**, not as an assumed limit.

## Operating Posture

Be the most demanding realistic user of this component. Production data is rarely as tidy as demo data: names vary, optional values disappear, counts grow, labels are translated, media fails to load, and collections become empty or large.

Two failure modes matter:

1. **Nonsense data.** Arbitrary thousands-of-character strings prove little if the system would never accept them. Use values that a real user, import, backend, or integration can plausibly produce—or the maximum the source actually accepts.
2. **Only testing long text.** The common shipped bugs are often a one-character name, missing avatar, count of exactly one, empty list, null subtitle, translated status badge, large number, or a list that becomes slow.

## Hard Rules

1. **Map every rendered value before building fixtures.** Include labels, counts, relative timestamps, badges, status text, button labels derived from data, tooltips, avatars, images, and collection length.
2. **Ground limits in evidence.** Inspect Dart models, JSON serialization, API schemas, form validators, `maxLength`, database constraints/migrations, and backend types. Note frontend/backend mismatches.
3. **Keep fixtures type-correct.** Worst-case fixtures must use the same model types and file conventions as the existing demo data.
4. **Do not contaminate production data paths.** Fixture selection must be development-only or isolated in a test/prototype route. Never ship a user-accessible debug toggle accidentally.
5. **Swap data at the boundary.** Keep the component under test unchanged where possible; inject the selected fixture through its existing data/model boundary.
6. **Do not hide failures with arbitrary clipping.** Recommend wrap, ellipsis, scrolling, constraints, or redesign based on the content's purpose. Preserve access to important content and actions.
7. **Do not fix before reporting unless the user explicitly asks for fixes.** First establish what breaks and provide a clear report. If the invocation includes `+ fix`, fix all non-ambiguous issues after documenting them.
8. **Test realistic viewport and text-scale extremes.** Include narrow widths, large text scaling, keyboard/insets when relevant, and target platform differences.
9. **No fake pass claims.** Distinguish inspected in a running app/device from inferred by code review or covered only by widget tests.
10. **Every issue gets a location.** Provide `file:line` when available, otherwise the widget/class/function and a clear locator.

## Workflow

### Phase 1 — Map the UI

Inspect the target widget/screen and trace every displayed value to its source.

Create a compact inventory:

| Field/value | Source | Validation/limit | Worst-case scenario |
|---|---|---|---|
| Display name | model/API | schema or “unbounded” | maximum realistic name |
| Email/identifier | model/API | backend constraint | long/unbroken value |
| Count | integer field | numeric range | 0, 1, large value |
| Avatar/media | URL/path | optional? | null, invalid, slow/failing |
| Collection | query/state | pagination/limit | empty, one, large |
| Label/status | enum/localization | translated strings | longest supported locale |

Look for:

- Dart model fields, nullable/non-nullable types, default values, `fromJson`/serialization.
- API/OpenAPI/GraphQL schemas and backend validation.
- Form validators and `maxLength`.
- Database column types, migrations, and server-side limits.
- `Text`, `RichText`, `TextField`, `InputDecorator`, `Tooltip`, badges, chips, buttons, menus, and accessibility labels.
- `Row`, `Column`, `Flex`, `Expanded`, `Flexible`, `Wrap`, `ListView`, `GridView`, `SliverList`, and nested scrollables.
- Fixed widths/heights, `AspectRatio`, `FittedBox`, clipping, overflow handling, and assumptions about text scale.
- Empty/loading/error/partial states and fallback images.
- Any derived value: pluralized text, formatted numbers, dates, durations, percentages, relative times, initials, or status labels.

Completion criterion: every value visible in the target UI appears in the inventory with a source and either a grounded limit or “unbounded / not found.”

### Phase 2 — Build the worst-case fixture

Read `CATALOG.md` now. It contains realistic edge-case patterns for text, identifiers, numbers, collections, dates, media, missing fields, localization, environment, accessibility, and performance.

Create fixtures shaped exactly like the existing demo data. Prefer a fixture file next to the current sample/mock data, using the project's established conventions.

Use realistic values:

- A long but plausible full name.
- A long email or identifier that the backend accepts.
- A short one-letter name.
- Missing/null optional values where the model permits them.
- Count 0, 1, and a large formatted count.
- Long localized labels and mixed scripts.
- Invalid or unavailable media.
- Empty and one-item collections.
- A realistically large collection based on the actual pagination/performance behavior.

Spread cases across rows. Do not put every failure into the first row; that makes it impossible to inspect the list's overall behavior.

Required fixture states:

- **Demo** — current representative data.
- **Worst case** — realistic adversarial values across fields.
- **Empty** — zero items and no-results behavior.
- **One** — one item; verify singular/plural and lone-item layout.
- **Huge** — realistic upper-bound or stress collection. For an unpaginated list, use 1,000+ items to expose performance/scroll issues, unless the app's real limit is lower. If the UI is paginated, test the page boundary and a realistic large dataset instead of defeating pagination.

Additional states may be warranted: loading, error, partial/missing data, offline, or media failure.

### Phase 3 — Add a development-only fixture switch

The user must be able to compare the same component under each fixture without changing its implementation.

Flutter-specific options:

- **Existing app with a dev server:** add a development-only route or debug panel that hosts the component and selects fixture data.
- **Flutter web:** optionally support a query parameter such as `?data=worst` when the project’s routing setup makes this straightforward. Keep the URL state so reloads preserve the selected fixture.
- **Mobile/desktop:** use a dev-only prototype route, debug menu, or segmented control.
- **Standalone reproduction:** create a minimal Flutter app/screen with the component and fixtures.

Use a neutral, unobtrusive control labeled `Demo / Worst case / Empty / One / Huge`. Add extra states only when useful.

Rules:

- The switch is test chrome, not part of the design under test.
- Keep it visually plain and out of the component's way.
- Switching should be instant; do not animate the content during stress testing.
- Keep fixture selection at the data boundary.
- Guard debug-only access with the project's established debug/dev mechanism (for example `kDebugMode`, a dev route, or a test-only harness).
- Do not expose worst-case fixtures in production builds.
- Avoid writing fixture selection into production user preferences.
- If adding a route, ensure it is not included in production navigation.

A minimal pattern may use an enum and injected model:

```dart
enum FixtureMode { demo, worstCase, empty, one, huge }

class StressHarness<T> extends StatelessWidget {
  const StressHarness({
    super.key,
    required this.mode,
    required this.fixtures,
    required this.builder,
  });

  final FixtureMode mode;
  final Map<FixtureMode, T> fixtures;
  final Widget Function(BuildContext context, T data) builder;

  @override
  Widget build(BuildContext context) {
    return builder(context, fixtures[mode]!);
  }
}
```

Adapt this to the project's architecture. Do not force a generic harness into a simple screen if a local fixture switch is clearer.

### Phase 4 — Inspect and try to break it

Run every fixture through the actual UI.

Inspect at minimum:

- Narrow phone width and the project's smallest supported layout.
- Typical phone width.
- Tablet/desktop width when supported.
- Large text scaling / accessibility text size.
- Long localized strings and non-Latin scripts.
- Light and dark themes if both are supported.
- Keyboard-visible state for forms.
- Empty, loading, error, and partial data.
- Image load failure and missing media.
- Scroll behavior and large collections.
- Touch targets and whether actions remain reachable.
- Screen-reader semantics for truncated or hidden content.

Look specifically for Flutter failures:

- `RenderFlex overflowed` yellow/black stripes.
- `A RenderFlex overflowed by ... pixels` in logs.
- `BoxConstraints` violations or unbounded constraints.
- Text clipping, ellipsis that hides essential content, unintended line count changes.
- `Expanded`/`Flexible` misuse in unbounded or nested layouts.
- Fixed-height cards that fail at large text scale.
- `ListView`/`GridView` nested scrolling conflicts.
- Sliver constraints and viewport issues.
- Rebuilds or eager construction that make large collections janky.
- `Image.network` failure states, missing dimensions, layout shifts.
- Dialog, bottom sheet, or menu content exceeding available height.
- Keyboard covering fields/actions.
- Accessibility labels that omit the full value when visual text is truncated.

Use Flutter DevTools, debug overflow indicators, logs, widget tests, golden tests where appropriate, and physical/emulator devices. Do not rely only on a screenshot at the default viewport.

### Phase 5 — Report, then stop

Do not silently redesign or apply fixes unless requested. Deliver the report in the format below.

## Required Output Format

### Part 1 — What broke

One row per distinct issue, worst first.

Severity:

- **Broken** — content unreadable, action unreachable, wrong data shown, crash/assertion, or core flow unusable.
- **Ugly** — content remains usable but visibly wrong: squashed avatar, awkward wrap, misaligned badge, clipped decoration.
- **Fragile** — works for current fixture but is one realistic step from breaking: missing limit/fallback, unsupported null case, fixed size assumption.

| # | Severity | Field/state | Worst-case value | What happened | Fix |
|---|---|---|---|---|---|

Every row must include a `file:line` location when available, or a clear widget/function locator.

### Part 2 — Decisions for you

List only issues with more than one valid product/design decision, one line each.

Examples:

- **Long title:** wrap or ellipsis? Recommend wrapping because the title is meaningful and the card has room.
- **Empty role:** hide the row or show “Not assigned”? Recommend explicit empty state because users need to distinguish missing data from loading.
- **Huge collection:** paginate or virtualize? Recommend pagination if the backend supports it; otherwise use lazy slivers and bounded loading.

Do not block obvious fixes behind a question.

### Part 3 — What held up

List the worst cases the UI already handles correctly. This confirms the test was meaningful and identifies what should not be changed.

### Close

State:

- Where the fixture switch lives (`route`, file path, or test harness).
- Which states it supports.
- Which viewport/text-scale/platform combinations were checked.
- What was not verified (if anything).

End with:

> Say `fix all` or `fix 1, 3` and I'll apply those fixes and re-check the fixtures.

## Invocation Variants

| Invocation | Behavior |
|---|---|
| `<widget or screen>` | Full workflow: map → fixtures → dev switch → inspect → report, then stop |
| `<widget or screen> + fix` | Same workflow, then apply all unambiguous fixes and re-verify |
| `fix all` / `fix 1, 3` | Apply named fixes from the last report and re-check every affected state |
| `data only <widget or screen>` | Build the fixture set and switch/harness without the full report |
| `review fixture <path>` | Inspect an existing fixture set for coverage gaps and type/limit realism |

## Flutter-specific Fix Guidance

Choose fixes based on the content's purpose; do not reflexively add ellipsis.

- For a `Row`, give text a bounded flexible region (`Expanded`/`Flexible`) and keep actions/icons from shrinking.
- For long text, decide between wrapping, ellipsis, line limits, or a detail view. Preserve full content through semantics or a secondary interaction when truncating.
- For badges/chips, allow sensible wrapping or constrain text based on the design; do not let a badge force the whole row wider.
- For avatars/media, preserve aspect ratio and size (`SizedBox`, `ClipOval`, `BoxFit`) and provide a fallback.
- For dynamic-height content, avoid hardcoded heights that fail under text scaling; prefer intrinsic content layout where feasible.
- For large collections, use lazy builders/slivers, pagination, or virtualization appropriate to the data source.
- For numbers, format with locale-aware APIs and test separators, compact notation, and extremes.
- For dates/times, test timezone boundaries, relative-time rollover, and localized formatting.
- For empty/error/loading states, make them explicit and distinct.
- For forms, align frontend validation with backend constraints and test pasted/imported values, not just typed input.
- For nested scrollables, define the intended scroll owner and physics; avoid accidental unbounded constraints.
- For responsive layouts, test breakpoints and minimum usable widths rather than only one device.

## Tone

Matter-of-fact, never smug. The component was built against friendly data, which is normal. Name the break, show the value that caused it, explain the user-visible consequence, and give the fix. If nothing breaks, say so. A short report on a sturdy UI is a good result.
