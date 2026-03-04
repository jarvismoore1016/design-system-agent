---
description: Generate structured markdown documentation for mobile UI components — single component or all components at once
allowed-tools: Read, Write, Glob, AskUserQuestion
argument-hint: component-name-or-path | all
---

# Generate Docs: $1

## Mode Detection

Check the value of `$1`:

- **`$1` is `all`** → follow **All Mode** below — document every component in the project
- **Anything else** → treat `$1` as a component name or file path and follow **Single Mode**

---

## All Mode — Document Every Component

Read `CLAUDE.md` for project context (platform, token file, component locations).

### Step 1 — Find all components

Scan platform-specific locations:
- **React Native**: `src/components/`, `components/`, `src/ui/`
- **Flutter**: `lib/widgets/`, `lib/components/`, `lib/ui/`
- **SwiftUI**: project source groups — look for `*View.swift` and component `.swift` files
- **Kotlin Compose**: `app/src/main/java/.../ui/components/`

Deduplicate and list them.

### Step 2 — Confirm the list

Present via AskUserQuestion with `multiSelect: true`:
- question: "Which components would you like to document?"
- header: "Components"
- options: [list all found component names]

### Step 3 — Document each confirmed component

For each confirmed component:
1. Read the component source file
2. Read the token file to identify which tokens the component uses
3. Write `docs/components/[component-name].md` following the Single Mode structure below
4. Create the `docs/components/` directory if it doesn't exist

After each doc:
```
✓ docs/components/app-button.md
✓ docs/components/app-card.md
✓ docs/components/app-badge.md
```

### Step 4 — Summary

```
Documentation complete.

6 components documented:
  docs/components/app-button.md
  docs/components/app-card.md
  docs/components/app-badge.md
  docs/components/app-input.md
  docs/components/app-modal.md
  docs/components/app-spinner.md

Run /audit-ui to check these components for token drift and accessibility issues.
```

---

## Single Mode — Document One Component

## Before You Write

Read `CLAUDE.md` for project context — platform, token naming convention, and component conventions.

Find the component file for `$1`. Check these platform-specific locations:

- **React Native**: `src/components/`, `components/`, `src/ui/`
- **Flutter**: `lib/widgets/`, `lib/components/`, `lib/ui/`
- **SwiftUI**: Project source groups; look for `$1.swift` or `$1View.swift`
- **Kotlin Compose**: `app/src/main/java/.../ui/components/`

Read the component file carefully. Read the token file to confirm which tokens the component uses.

## Documentation to Write

Save the output to `docs/components/[component-name].md`. Create the directory if it doesn't exist.

---

### Title and Status Badge

Start with: `# [Component Name]`

Add a brief status note if relevant (e.g., "Stable — safe to use in production" or "In progress — API may change").

---

### Overview

2-4 sentences covering:
- What this component is
- When to use it
- When NOT to use it (and what to use instead)

---

### Variants

A table or description of every visual/functional variant with the use case for each:

| Variant | When to use |
|---------|-------------|
| Primary | The main action on a screen — use once per context |
| Secondary | Supporting actions that need less visual weight |
| Ghost | Tertiary actions or where the component sits on a colored background |
| Destructive | Irreversible actions — delete, remove, clear |

---

### States

Table showing the visual and behavioral state of the component:

| State | Description |
|-------|-------------|
| Default | Resting appearance |
| Pressed | Touch feedback — describe opacity, scale, or color change |
| Disabled | Not interactive — describe appearance and how it's communicated |
| Loading | If applicable — describe indicator and accessible announcement |

---

### API / Parameters

Use platform-appropriate terminology:

**React Native (props)**:

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `label` | `string` | — | Visible text and accessibility label |
| `onPress` | `() => void` | — | Press handler |
| `variant` | `'primary' \| 'secondary' \| 'ghost' \| 'destructive'` | `'primary'` | Visual style |
| `size` | `'sm' \| 'md' \| 'lg'` | `'md'` | Component size |
| `disabled` | `boolean` | `false` | Disables interaction |
| `loading` | `boolean` | `false` | Shows loading state |
| `accessibilityHint` | `string` | — | Additional context for screen readers |

**Flutter (constructor parameters)**:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `label` | `String` | required | Visible text and semantic label |
| `onPressed` | `VoidCallback?` | required | Tap handler; null disables the button |
| `variant` | `ComponentNameVariant` | `.primary` | Visual style |
| `size` | `ComponentNameSize` | `.md` | Component size |
| `isLoading` | `bool` | `false` | Shows loading state |

**SwiftUI (view modifiers / initializer)**:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `label` | `String` | required | Visible text and accessibility label |
| `action` | `() -> Void` | required | Tap handler |
| `variant` | `ComponentNameVariant` | `.primary` | Visual style |
| `size` | `ComponentNameSize` | `.md` | Component size |
| `isDisabled` | `Bool` | `false` | Disables interaction |
| `isLoading` | `Bool` | `false` | Shows loading state |
| `accessibilityHint` | `String?` | `nil` | Screen reader hint |

**Kotlin Compose (function parameters)**:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `label` | `String` | required | Visible text and content description |
| `onClick` | `() -> Unit` | required | Click handler |
| `modifier` | `Modifier` | `Modifier` | Layout modifier |
| `variant` | `ComponentNameVariant` | `Primary` | Visual style |
| `size` | `ComponentNameSize` | `Md` | Component size |
| `enabled` | `Boolean` | `true` | Enables or disables interaction |
| `isLoading` | `Boolean` | `false` | Shows loading state |

---

### Design Tokens Used

List which tokens from the design system this component uses:

| Token | Role |
|-------|------|
| `colors.primary` / `AppColors.primary` | Background color of primary variant |
| `colors.textOnPrimary` / `AppColors.textOnPrimary` | Label color on primary background |
| `spacing.sm` / `AppSpacing.sm` | Vertical padding |
| `spacing.md` / `AppSpacing.md` | Horizontal padding |
| `radius.md` / `AppRadius.md` | Corner radius |
| `typography.fontSize.base` / `AppTypography.body` | Default font size |

---

### Accessibility

Platform-specific accessibility notes:

**React Native**:
- `accessibilityRole="button"` — announces as button to VoiceOver and TalkBack
- `accessibilityLabel` — set to the visible `label` prop; VoiceOver reads this
- `accessibilityState={{ disabled }}` — communicates disabled state to screen readers
- `accessibilityHint` — optional; use to describe what happens on press when not obvious
- Minimum touch target: 44×44pt enforced via `minHeight: 44, minWidth: 44`

**Flutter**:
- `Semantics(button: true, label: label)` — announces as button with correct label
- When `onPressed` is null, Flutter marks the widget as disabled in semantics automatically
- `CircularProgressIndicator` in loading state should have a `semanticsLabel`

**SwiftUI**:
- `.accessibilityLabel(label)` — explicitly sets what VoiceOver reads
- `.accessibilityHint(hint)` — reads after label on long pause; use for non-obvious actions
- `.accessibilityAddTraits(.isButton)` — ensures correct role announcement
- Minimum touch target: `.frame(minWidth: 44, minHeight: 44)` applied inside the view

**Kotlin Compose**:
- `semantics { contentDescription = label }` — sets TalkBack announcement
- `enabled` parameter propagates to `semantics { disabled = !enabled }` automatically
- Loading state: provide `contentDescription = "Loading"` to the `CircularProgressIndicator`

---

### Gesture and Interaction

(Mobile equivalent of keyboard section)

| Gesture | Behavior |
|---------|----------|
| Tap | Triggers `onPress` / `onPressed` / `action` / `onClick` |
| Long press | No default behavior — add `onLongPress` prop if needed |
| Swipe | Not applicable — use dedicated swipeable components |

Touch feedback:
- **React Native**: Pressable `pressed` state reduces opacity to 0.75
- **Flutter**: ElevatedButton uses Material ink splash by default
- **SwiftUI**: Button uses system highlight; ButtonStyle can customize
- **Kotlin Compose**: Button uses Material ripple via `Indication`

---

### Platform Differences

If the component behaves differently on iOS vs Android (React Native), or has platform-specific implementation notes:

| Aspect | iOS | Android |
|--------|-----|---------|
| Shadow | `shadowColor`, `shadowOffset`, `shadowRadius` | `elevation` |
| Min touch target | 44×44pt (Apple HIG) | 48×48dp (Material) |
| Haptics | Optional `UIImpactFeedbackGenerator` | Optional `HapticFeedbackConstants` |

---

### Do's and Don'ts

**Do:**
- Use one primary button per screen section
- Use a descriptive label — "Save changes", not "Submit"
- Use `disabled` state when action is unavailable; explain why when possible

**Don't:**
- Don't use icon-only buttons without an `accessibilityLabel`
- Don't use `TouchableOpacity` (React Native) — use `Pressable` for better control
- Don't make touch targets smaller than platform minimums
- Don't use the destructive variant for reversible actions

---

### Usage Examples

**Example 1: Basic usage**
```[language]
// React Native
<AppButton label="Save changes" onPress={handleSave} />

// Flutter
AppButton(label: 'Save changes', onPressed: handleSave)

// SwiftUI
AppButton(label: "Save changes", action: handleSave)

// Compose
AppButton(label = "Save changes", onClick = { handleSave() })
```

**Example 2: Variants**
```[language]
// React Native
<AppButton label="Delete account" variant="destructive" onPress={handleDelete} />
<AppButton label="Cancel" variant="ghost" onPress={handleCancel} />

// Flutter
AppButton(
  label: 'Delete account',
  variant: AppButtonVariant.destructive,
  onPressed: handleDelete,
)

// SwiftUI
AppButton(label: "Delete account", variant: .destructive, action: handleDelete)

// Compose
AppButton(
  label = "Delete account",
  variant = AppButtonVariant.Destructive,
  onClick = { handleDelete() }
)
```

**Example 3: Loading state**
```[language]
// React Native
<AppButton label="Saving" loading={isSaving} onPress={handleSave} />

// Flutter
AppButton(label: 'Saving', isLoading: isSaving, onPressed: handleSave)

// SwiftUI
AppButton(label: "Saving", isLoading: isSaving, action: handleSave)

// Compose
AppButton(label = "Saving", isLoading = isSaving, onClick = { handleSave() })
```

---

## After Writing

Tell the user:
- The path to the generated documentation file
- How many sections were included
- Suggest running `/generate-docs all` if they want to document additional components
