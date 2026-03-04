# AppButton

**Status**: Stable — safe to use in production

---

## Overview

`AppButton` is the primary interactive element for triggering actions in the app. Use it for any action a user can take: submitting forms, confirming dialogs, navigating to a flow, or destructive operations like deleting data.

Use a link (`Text` with `onPress` + navigation) when the interaction navigates the user to a new route without triggering an action. Use `AppButton` when something *happens* as a result of the tap.

---

## Variants

| Variant | When to use |
|---------|-------------|
| `primary` | The main action on a screen — use once per content area |
| `secondary` | Supporting actions that need less visual weight than primary |
| `ghost` | Tertiary actions, or when the button sits on a colored background |
| `destructive` | Irreversible actions only: delete, remove, clear all |

---

## States

| State | Description |
|-------|-------------|
| Default | Full opacity, background per variant |
| Pressed | Opacity reduced to 0.75 — immediate visual feedback on touch |
| Disabled | Opacity 0.4, non-interactive, `accessibilityState.disabled = true` |
| Loading | Label replaced with `ActivityIndicator`; button non-interactive; spinner color matches variant foreground |

---

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `label` | `string` | required | Visible text. Also used as the VoiceOver/TalkBack accessible label. |
| `onPress` | `() => void` | required | Called when the button is tapped. Not called when `disabled` or `loading`. |
| `variant` | `'primary' \| 'secondary' \| 'ghost' \| 'destructive'` | `'primary'` | Visual style. |
| `size` | `'sm' \| 'md' \| 'lg'` | `'md'` | Controls padding and font size. |
| `disabled` | `boolean` | `false` | Disables interaction. Applies 0.4 opacity. |
| `loading` | `boolean` | `false` | Shows loading spinner. Disables interaction automatically. |
| `accessibilityHint` | `string` | `undefined` | Screen reader hint; read after a pause following the label. Use for non-obvious actions. |

All other `PressableProps` are forwarded to the underlying `Pressable` component.

---

## Design Tokens Used

| Token | Role |
|-------|------|
| `colors.primary` | Background color of primary variant; label color of secondary/ghost variants |
| `colors.danger` | Background color of destructive variant |
| `colors.textOnPrimary` | Label color on primary and destructive backgrounds |
| `spacing.xs` | Vertical padding — small size |
| `spacing.sm` | Vertical padding — medium size; horizontal padding — small size |
| `spacing.md` | Vertical padding — large size; horizontal padding — medium size |
| `spacing.lg` | Horizontal padding — large size |
| `typography.fontSize.sm` | Font size — small variant |
| `typography.fontSize.base` | Font size — medium variant |
| `typography.fontSize.lg` | Font size — large variant |
| `typography.fontWeight.semibold` | Label font weight |
| `radius.md` | Corner radius — all variants |

---

## Accessibility

- **Role**: `accessibilityRole="button"` — announces as "button" to VoiceOver and TalkBack
- **Label**: `accessibilityLabel` is set to the `label` prop value automatically; VoiceOver reads this in full
- **State**: `accessibilityState={{ disabled: true }}` when the button is disabled or loading — communicated to screen readers
- **Hint**: Optional `accessibilityHint` prop for non-obvious actions, e.g., "Permanently removes your account"
- **Touch target**: `minHeight: 44, minWidth: 44` enforced in `StyleSheet` — meets Apple HIG (44pt) and Material Design (48dp) minimums
- **Loading**: `ActivityIndicator` has `accessibilityLabel="Loading"` so screen readers announce the change in state

**VoiceOver reads**: "[label], button" (default) or "[label], dimmed, button" (disabled)
**TalkBack reads**: "[label], button" or "[label], unavailable" (disabled)

---

## Gesture and Interaction

| Gesture | Behavior |
|---------|----------|
| Tap | Triggers `onPress`. No-op if `disabled` or `loading`. |
| Long press | No default behavior. Pass `onLongPress` via spread props if needed. |

Touch feedback is handled by the React Native `Pressable` component:
- `pressed` state reduces opacity to 0.75 via the `style` callback
- On iOS, tap feedback is handled by opacity change
- On Android, `Pressable` uses the ripple effect by default (override with `android_ripple` prop)

---

## Platform Differences

| Aspect | iOS | Android |
|--------|-----|---------|
| Shadow | `shadowColor`, `shadowOffset`, `shadowOpacity`, `shadowRadius` | `elevation: 2` |
| Min touch target | 44×44pt (Apple HIG) | 48×48dp enforced via `minHeight: 44` which exceeds Android's 48dp minimum at standard density |
| Ripple feedback | Opacity change via `pressed` state | Native ripple from `Pressable` |
| Font rendering | SF Pro system font (default) | Roboto system font (default) |

---

## Do's and Don'ts

**Do:**
- Use one primary button per screen section
- Use `label` text that describes the outcome: "Save changes", "Delete account", "Start free trial"
- Add `accessibilityHint` when the consequence of the action isn't obvious from the label alone
- Use `loading={true}` to indicate async operations — don't just disable the button silently

**Don't:**
- Don't use icon-only `AppButton` — use a specialized `IconButton` component with its own accessible label
- Don't use `disabled` as a way to hide unavailable features — explain why the action is unavailable
- Don't put multiple primary buttons in the same visible area of a screen
- Don't use the destructive variant for reversible actions — "Clear filters" is not destructive; "Delete account" is

---

## Usage Examples

**Example 1: Basic usage**
```tsx
import { AppButton } from '../components/AppButton';

<AppButton label="Save changes" onPress={handleSave} />
```

**Example 2: Variants and sizes**
```tsx
// Secondary with small size
<AppButton
  label="Cancel"
  variant="secondary"
  size="sm"
  onPress={handleCancel}
/>

// Destructive with accessibility hint
<AppButton
  label="Delete account"
  variant="destructive"
  onPress={handleDelete}
  accessibilityHint="Permanently deletes your account and all associated data"
/>
```

**Example 3: In a form context with loading state**
```tsx
import { useState } from 'react';
import { View } from 'react-native';
import { AppButton } from '../components/AppButton';

function CheckoutForm() {
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleSubmit = async () => {
    setIsSubmitting(true);
    try {
      await submitOrder();
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <View>
      {/* ... form fields ... */}
      <AppButton
        label="Place order"
        size="lg"
        loading={isSubmitting}
        onPress={handleSubmit}
      />
    </View>
  );
}
```

---

*Generated by `/generate-docs AppButton` — Design System Assistant (Mobile)*
