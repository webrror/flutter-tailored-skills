# Worst-Case Data Catalog for Flutter UI

Read this file during Phase 2 of `break-ui-flutter`.

The goal is **realistic adversarial data**, not arbitrary nonsense. Use project schemas, backend constraints, user input rules, localization files, and product behavior to select values. Adapt examples to the model's actual types and limits.

## 1. Text and names

| Case | Example/pattern | What it exposes |
|---|---|---|
| Long realistic full name | `Aleksandra Wiśniewska-Kowalczyk` | Row width, avatar spacing, title wrapping |
| One-character name | `Q` | Empty-looking layouts, avatar initials, orphaned separators |
| Multi-part name | `Mary-Jane van der Meer` | Assumptions about first/last name |
| Combining marks | `Jose\u0301` / NFC equivalent | Text rendering and normalization |
| Non-Latin | `நிலா`, `محمد`, `山田太郎` | Font fallback, directionality, glyph clipping |
| Mixed scripts | `Alex 李-محمد` | Bidi and mixed-script layout |
| Emoji | `👩🏽‍💻`, `🏳️‍🌈` | Grapheme handling, baseline, truncation |
| Leading/trailing spaces | `"  Alex  "` | Normalization and alignment |
| Newlines | Multi-line text where accepted | Height assumptions and overflow |
| Long unbroken token | Realistic URL/token/slug accepted by system | Wrapping and overflow behavior |
| Empty string | `""` | Empty placeholder and spacing |
| Whitespace-only | `"   "` | Validation vs display fallback |
| Long title/description | Maximum accepted or realistic imported text | Wrapping, line limits, card height |

Do not use a 5,000-character name unless the product can genuinely receive it. Prefer the schema limit or a realistic integration/import case.

## 2. Emails, identifiers, and URLs

| Case | Example/pattern | What it exposes |
|---|---|---|
| Long valid email | `bartholomew.fitzgerald@northwind-industries-holdings.example.com` | Row overflow, action reachability |
| Short email/local part | `a@b.co` if accepted | Tight spacing, assumptions |
| Plus addressing | `alex+receipts@example.com` | Parsing/display |
| Long domain/subdomain | Long valid domain within system constraints | Wrapping |
| URL with query | Long but valid URL | Wrapping, overflow, copy action |
| UUID | Full canonical UUID | Identifier display |
| Long external ID | Maximum schema-accepted ID | Fixed width, truncation |
| Leading zeros | `00001234` | Numeric coercion |
| Case variation | Mixed-case identifier where valid | Normalization |
| Invalid identifier | Only if the UI handles invalid/imported data | Error/fallback state |

Inspect server-side constraints and actual identifier formats. Do not assume email maximums or URL limits without evidence.

## 3. Numbers, counts, currency, and percentages

Test each numeric field at:

- `0`
- `1` (singular/plural)
- `2`
- Negative values if domain-valid or possible from bad data
- Maximum schema/database value
- Large realistic values with locale separators
- Decimal precision and rounding boundaries
- Very small fractional values
- Overflow/unknown values if the API can return them
- Null/missing where permitted

Examples:

| Type | Cases |
|---|---|
| Count | 0, 1, 2, 999, 1,000, 1,000,000 or actual max |
| Currency | zero, small amount, large amount, negative/refund, fractional cents if valid |
| Percentage | 0%, 1%, 99.9%, 100%, over/under range only if domain permits |
| Duration | zero, seconds, hours, multi-day |
| Rating | minimum, maximum, no rating, fractional rating |
| Numeric ID | leading zero display, very large integer |
| Compact formatting | values around unit changes (999/1K, 999K/1M) |

Use locale-aware formatting. Check rounding, separators, compact notation, currency symbol placement, and label pluralization.

## 4. Collections and lists

Required collection states:

- Empty (`[]`)
- One item
- Two items
- Typical data
- Large realistic data
- Duplicate items if the domain permits them
- Mixed item lengths and states
- Missing/partial item fields
- Unsorted data if ordering matters
- Long first item and long last item
- Page boundary: exactly page size, page size + 1
- Loading more / exhausted pagination / failed next page

For an unpaginated list, use 1,000+ rows as a stress case unless the product limit is lower. For paginated data, test realistic page boundaries and loading behavior rather than disabling pagination.

Flutter concerns:

- `ListView.builder`, `GridView.builder`, `SliverList`, `SliverGrid` vs eager `Column` construction.
- Scroll performance and memory.
- Stable keys and state retention.
- Nested scroll ownership.
- Empty state and pull-to-refresh behavior.
- List separators at first/last item.
- Sticky headers and grouped lists.

## 5. Missing, null, and partial data

For every nullable/optional field, test:

- `null`
- Missing JSON key
- Empty string
- Whitespace-only string
- Empty collection
- Invalid enum / unknown future enum value
- Partial object with only required fields
- API error or unavailable source
- Stale cached value where applicable

Examples:

- No avatar/photo.
- No subtitle or secondary label.
- No date.
- No status.
- No optional action.
- No description.
- No metadata.
- Missing nested object.
- Unknown status from a newer backend.

Do not test null for non-nullable model fields unless testing deserialization/error handling. Keep the fixture type-correct or explicitly label it as malformed API input.

## 6. Dates, time, and relative labels

Test:

- Just now / boundary before and after minute, hour, day, month, and year transitions.
- Future timestamp if possible due to clock skew.
- Very old timestamp.
- Midnight and date rollover.
- Month-end and year-end.
- Leap day.
- Timezone offset changes and DST where relevant.
- Different locale date order and 12/24-hour preferences.
- Long localized month/day names.
- Missing/invalid timestamp.

Verify the product's timezone policy. Do not fabricate timezone behavior; use deterministic test clocks where the project supports them.

## 7. Media and avatars

Test:

- No image URL/path.
- Invalid URL/path.
- 404 or network failure.
- Slow load.
- Very wide and very tall aspect ratios.
- Small/low-resolution image.
- Large image dimensions.
- Transparent image.
- Animated media if supported.
- Corrupt/unreadable file.
- Avatar fallback initials with one-character and multi-part names.

Flutter-specific checks:

- `Image.errorBuilder` or package-equivalent fallback.
- Explicit dimensions to prevent layout shifts.
- `BoxFit` and clipping.
- Loading placeholder.
- Cache behavior.
- Avoid unbounded image sizes in lists.
- Preserve avatar/media dimensions under `Row` pressure.

## 8. Labels, badges, statuses, and actions

Test:

- Longest supported localized label.
- One-character label.
- Empty label.
- Unknown enum/status.
- Multiple badges in one row.
- Badge plus long title plus trailing menu.
- Long button label.
- Disabled/loading button.
- Action text at large text scale.
- Tooltip/accessibility label longer than visual label.

Check whether labels wrap, truncate, or force layout expansion. Ensure action controls remain reachable and do not shrink into unusable targets.

## 9. Localization and directionality

Test at least:

- English baseline.
- Longest supported locale strings.
- German-like compound words.
- French/Spanish accents.
- Arabic or Hebrew RTL.
- CJK text.
- Mixed LTR/RTL values such as email addresses in RTL UI.
- Plural forms for 0, 1, 2, and large counts.
- Date, number, currency, and percentage formatting.
- Text expansion in buttons, menus, dialogs, tabs, and chips.

Flutter checks:

- `Directionality` and app localization delegates.
- Layout mirroring.
- Directional padding/alignment (`EdgeInsetsDirectional`, `AlignmentDirectional`) where appropriate.
- Correct order of icons and labels in RTL.
- Bidi isolation for embedded identifiers/URLs where needed.

## 10. Forms and validation

For every input:

- Empty.
- Minimum length.
- Maximum accepted length.
- One beyond the maximum.
- Leading/trailing whitespace.
- Pasted content.
- Multi-line input if supported.
- Emoji and combining marks.
- Non-Latin input.
- Invalid format.
- Valid but unusually long input.
- Backend rejection despite frontend acceptance.
- Keyboard visible with bottom actions.
- Validation error text long enough to wrap.
- Focus traversal and submission behavior.

Compare client validators with server constraints. Test imported/API values that bypass the text field's `maxLength`.

## 11. Responsive layout and constraints

Test:

- Smallest supported width.
- Typical phone width.
- Landscape orientation if supported.
- Tablet width.
- Desktop/web width.
- Window resize on desktop/web.
- Large text scaling.
- Keyboard open.
- Safe-area cutouts and insets.
- Split-screen or constrained window where supported.

Flutter-specific failure points:

- `Row`/`Column` overflow.
- Missing `Expanded`/`Flexible` in bounded rows.
- `Expanded` in unbounded scroll constraints.
- Fixed-height cards and dialogs.
- `IntrinsicHeight`/`IntrinsicWidth` overuse.
- `FittedBox` shrinking text below readable size.
- Nested `ListView`/`GridView` scrolling.
- Incorrect `shrinkWrap` use on large lists.
- Dialog/bottom-sheet content not scrollable.
- Fixed bottom actions hidden by keyboard.
- `SafeArea` omissions.
- RTL alignment and directional insets.

## 12. Accessibility and interaction

Test:

- Large system text scale.
- Screen reader labels for truncated content.
- Focus order and keyboard navigation.
- Minimum touch target behavior.
- Long-press/tooltip alternatives.
- Disabled vs loading vs error states.
- High contrast or theme contrast requirements.
- Reduced motion if the UI contains animated state changes.
- Semantics for empty/loading/error states.
- Content not conveyed by color alone.

Use TalkBack/VoiceOver and keyboard testing where supported. A screenshot alone cannot validate accessibility.

## 13. State and failure conditions

Test:

- Loading.
- Empty.
- Success.
- Partial success.
- Error.
- Retry.
- Offline.
- Stale cache.
- Permission denied.
- Expired session.
- Rate limited.
- Server unavailable.
- Duplicate submission.
- Rapid repeated taps.
- Concurrent updates.
- Optimistic update rollback.
- Long-running operation.
- Cancellation.

Ensure states are distinguishable and do not leave stale or contradictory UI.

## 14. Environment and platform

Where applicable, inspect:

- Android small/large screens and platform text settings.
- iOS safe areas, Dynamic Type, VoiceOver.
- Web browser width/zoom, keyboard, pointer, URL routing.
- Desktop window resizing and keyboard navigation.
- Light/dark theme.
- Network quality and offline behavior.
- Font availability/fallback.
- Device pixel ratio and image scaling.

Do not claim platform coverage that was not tested.

## 15. Fixture quality checklist

Before using the fixture, verify:

- [ ] It matches the real model shape and types.
- [ ] Values are plausible or tied to actual source limits.
- [ ] Every rendered field is covered.
- [ ] Long and short values are both represented.
- [ ] Null/missing cases respect model nullability or are clearly malformed-input tests.
- [ ] Counts include 0 and 1.
- [ ] Empty, one, and huge collections exist.
- [ ] Localization/directionality cases are included where relevant.
- [ ] Media failures are represented.
- [ ] Fixture selection is isolated from production.
- [ ] The fixture can be reset reproducibly.
- [ ] The same widget is tested, not a simplified lookalike.

## 16. Common fix patterns (not automatic prescriptions)

| Failure | Possible Flutter fix |
|---|---|
| Text pushes trailing action off-screen | Bound text with `Expanded`/`Flexible`; keep action non-flexible |
| Long text clipped | Wrap, line limit with accessible full value, or detail view |
| Avatar squashes | Fixed constraints, `ClipOval`, `BoxFit`, non-flexible size |
| Badge wraps badly | `Wrap`, max width, deliberate truncation or responsive placement |
| Empty list looks broken | Explicit empty state with useful next action |
| “1 items” | Locale-aware pluralization |
| Huge list janks | Lazy builder/slivers, pagination, bounded loading |
| Fixed card height clips large text | Content-driven height or scrollable region |
| Keyboard covers CTA | Insets-aware layout, scroll-to-focused-field, keyboard-aware bottom area |
| RTL layout is wrong | Directional padding/alignment and mirrored ordering |
| Image failure leaves blank box | Placeholder/error fallback with preserved dimensions |
| Long localized button overflows | Wrap, adapt layout, or use a shorter localized string without losing meaning |
| Null field leaves orphaned punctuation | Conditional composition of labels/separators |
| Unknown enum crashes | Safe fallback state and telemetry where appropriate |

Fixes are contextual. Do not apply a pattern without confirming it preserves the component's intent.
