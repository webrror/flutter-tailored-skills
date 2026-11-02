# Plan Template

Every plan written by `improve-flutter-animations` follows this structure. The executor may have no context from the audit or conversation. Include all relevant facts and exact targets.

```markdown
# NNN — <Short imperative title>

- **Status**: TODO
- **Commit**: <short git commit hash, or unavailable>
- **Severity**: HIGH | MEDIUM | LOW
- **Category**: <audit category>
- **Estimated scope**: <n files; rough effort>
- **Platforms**: <affected platforms>

## Problem

Explain what is wrong, where, and why it matters to the product's motion.

Cite every location as `path/to/file.dart:123`. Include the current code verbatim:

```dart
// lib/widgets/example.dart:14 — current
...
```

State the user-visible consequence and the evidence supporting the finding.

If the current code no longer matches the excerpt or line reference, STOP and report drift. Do not improvise a different fix.

## Target behavior

Describe the intended behavior in concrete terms:
- Purpose of the motion.
- Trigger and state transition.
- Properties that change.
- Curve and duration, or spring/physics approach.
- Interruptibility and dismissal behavior.
- Reduced-motion behavior.
- Platform differences, if any.

Use exact values from `AUDIT.md` or existing project tokens. If a value is a proposal requiring feel validation, label it as such.

## Implementation steps

1. <Exact change in exact file>
2. <Reuse/update the project's motion token or helper>
3. <Handle state, keys, lifecycle, and accessibility>
4. <Keep changes within the stated scope>

### Scope boundaries

- Change: <specific files/components>
- Do not change: <unrelated UI, architecture, dependencies, or behavior>
- Dependencies: <existing helpers/packages; no new dependency unless justified>

## Verification

- **Mechanical**: <exact commands available in this repo: analyze, tests, targeted tests; expected result>.
- **Accessibility**: <system reduced-motion setting, text scale, keyboard/focus or semantics checks>.
- **Feel check**: run the interaction and confirm:
  - <observable curve/origin/response behavior>
  - <interrupt or reverse mid-flight>
  - <gesture velocity/handoff, if applicable>
  - <no layout jump, clipping, or stale replay>
  - <reduced motion drops movement while preserving useful state feedback>
- **Performance**: profile in profile/release mode on representative hardware when the finding concerns jank, expensive effects, or frame cost.
- **Done when**: <machine-checkable and/or directly observable completion criteria>.

## Risks / notes

<Platform caveats, uncertainty, or decisions that must not be guessed>
```

## Plan author rules

- One plan per finding. Merge only when findings share the same files and fix pattern.
- Plans must be self-contained; never refer to “the audit above.”
- Use exact file paths and current excerpts.
- Pull values from `AUDIT.md` and the repo's existing tokens; never approximate silently.
- The feel check is mandatory. Mechanically correct motion can still feel wrong.
- Include real-device verification for gesture feel and performance-sensitive motion.
- Do not ask the executor to install dependencies unless the plan explicitly justifies it and the user approves.
- Do not include unrelated cleanup.
- After writing plans, create/update `plans/README.md` with plan number, title, severity, status, recommended order, and dependencies.
