# Button

**Status**: Stable — safe to use in production

A button triggers an action. It's the primary way users interact with forms, dialogs, and in-page operations.

---

## Overview

Use `Button` for actions — operations that happen on the current page like saving data, submitting a form, opening a dialog, or triggering a process.

Don't use Button for navigation. If clicking will take the user to a new URL, use a link (`<a>`) styled as needed. The difference matters for screen readers and keyboard users.

---

## Variants

| Variant | When to use |
|---------|-------------|
| `primary` | The single most important action in a given context. Use once per section or dialog. |
| `secondary` | Supporting actions with less visual weight — Cancel next to Save, for example. |
| `ghost` | Tertiary actions, or actions placed on colored backgrounds where a solid button would clash. |
| `destructive` | Irreversible actions — delete, remove, clear all. Signals risk. |

Avoid placing two `primary` buttons next to each other. If both actions are equally important, make one `secondary`.

---

## States

| State | Visual change | Behavior |
|-------|---------------|----------|
| Default | Solid fill, full opacity | Clickable |
| Hover | Darker background (10% shift) | Cursor: pointer |
| Focus | 2px outline using `--color-focus-ring` | Outline visible on keyboard navigation |
| Active / Pressed | 1px downward shift | During click or Space/Enter hold |
| Disabled | 45% opacity | Not clickable, `not-allowed` cursor, `pointer-events: none` |

The focus state is always visible — never suppressed. This is required for keyboard accessibility.

---

## API

### HTML Classes

| Class | Description |
|-------|-------------|
| `.btn` | Base class. Required on every button. |
| `.btn--primary` | Primary variant |
| `.btn--secondary` | Secondary variant |
| `.btn--ghost` | Ghost variant |
| `.btn--destructive` | Destructive variant |
| `.btn--sm` | Small size (32px height) |
| `.btn--md` | Medium size (40px height) — default |
| `.btn--lg` | Large size (48px height) |
| `.btn--icon-only` | Icon-only layout, square aspect ratio, 40×40px minimum |

### Attributes

| Attribute | Values | Description |
|-----------|--------|-------------|
| `disabled` | boolean | Disables the button visually and functionally |
| `aria-disabled` | `"true"` | For cases where you need the button in the tab order but inactive |
| `aria-label` | string | Required on icon-only buttons |
| `type` | `"button"` `"submit"` `"reset"` | Defaults to `"submit"` inside a `<form>` — always set explicitly |

---

## Design Tokens Used

| Token | Role |
|-------|------|
| `--color-primary` | Primary variant background and border |
| `--color-primary-hover` | Primary background on hover/active |
| `--color-primary-subtle` | Ghost variant hover background |
| `--color-text-on-primary` | Text and icon color on primary/destructive backgrounds |
| `--color-destructive` | Destructive variant background |
| `--color-destructive-hover` | Destructive background on hover/active |
| `--color-surface` | Secondary variant background |
| `--color-surface-hover` | Secondary background on hover |
| `--color-border` | Secondary variant border |
| `--color-text` | Secondary and ghost text color |
| `--color-focus-ring` | Focus outline color |
| `--spacing-xs` | Small padding, gap between icon and label |
| `--spacing-sm` | Default vertical padding |
| `--spacing-md` | Default horizontal padding |
| `--spacing-lg` | Large size horizontal padding |
| `--radius-sm` | Border radius |
| `--font-size-sm` | Small variant font size |
| `--font-size-md` | Default font size |
| `--font-size-lg` | Large variant font size |
| `--font-weight-medium` | Button label weight |

---

## Accessibility

**Element**: `<button>` — provides native keyboard interaction (Enter and Space to activate), focus management, and correct ARIA role automatically. Don't use `<div>` or `<span>` as buttons.

**Keyboard**:
- `Tab` / `Shift+Tab` — move focus to and from the button
- `Enter` or `Space` — activate the button
- No additional keyboard handling needed for standard buttons

**Screen reader announcement**: "[Label] button" — the browser appends "button" automatically. Don't include the word "button" in the label.

**Icon-only buttons**: Must have `aria-label` with a descriptive action name. The SVG icon should have `aria-hidden="true"` and `focusable="false"` to prevent double-announcement.

**Disabled state**: Use the native `disabled` attribute where possible — it removes the button from tab order and prevents all interactions. Use `aria-disabled="true"` only if the button must remain focusable (e.g., to show a tooltip explaining why it's disabled).

**Focus indicator**: Always visible. The `outline: none` rule from older resets is not used here. The focus outline uses `--color-focus-ring` and `outline-offset: 2px` for clear separation from the button's own border.

---

## Do's and Don'ts

**Do:**
- Use one `primary` button per logical section or dialog
- Write labels as verbs that describe the action: "Save changes", "Delete account", "Add to cart"
- Set `type="button"` explicitly on buttons inside forms that should not submit
- Always pair a destructive button with a confirmation step (modal or inline prompt)
- Give icon-only buttons an `aria-label` that names the action

**Don't:**
- Don't disable a button to hide functionality — explain why the action is unavailable instead
- Don't use `primary` for destructive actions — use the `destructive` variant so the risk is visually clear
- Don't use a button where a link is semantically correct (navigating to another page)
- Don't use vague labels: "OK", "Submit", "Yes", "Click here"
- Don't remove the focus indicator — it's required for keyboard and accessibility

---

## Usage Examples

**Basic form actions**
```html
<div class="form-actions">
  <button class="btn btn--primary" type="submit">Save changes</button>
  <button class="btn btn--secondary" type="button">Cancel</button>
</div>
```

**Destructive action with confirmation pattern**
```html
<!-- Step 1: Initial state -->
<button class="btn btn--destructive" id="delete-btn" type="button">
  Delete account
</button>

<!-- Step 2: After click — show confirmation inline or in a dialog -->
<div class="confirmation" role="alertdialog" aria-labelledby="confirm-heading">
  <p id="confirm-heading">Are you sure? This cannot be undone.</p>
  <button class="btn btn--destructive btn--sm" type="button">Yes, delete my account</button>
  <button class="btn btn--ghost btn--sm" type="button">Cancel</button>
</div>
```

**Icon-only button**
```html
<button class="btn btn--ghost btn--icon-only" aria-label="Close dialog" type="button">
  <svg aria-hidden="true" focusable="false" width="16" height="16" viewBox="0 0 16 16">
    <path d="M2 2L14 14M14 2L2 14" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
  </svg>
</button>
```

**Button with leading icon**
```html
<button class="btn btn--primary" type="button">
  <svg class="btn__icon" aria-hidden="true" focusable="false" width="16" height="16" viewBox="0 0 16 16">
    <!-- icon path -->
  </svg>
  Add to cart
</button>
```

**Size variants in context**
```html
<!-- Toolbar with small buttons -->
<div class="toolbar" role="toolbar" aria-label="Text formatting">
  <button class="btn btn--ghost btn--sm" type="button">Bold</button>
  <button class="btn btn--ghost btn--sm" type="button">Italic</button>
</div>

<!-- Hero call-to-action with large button -->
<section class="hero">
  <h1>Start your free trial</h1>
  <button class="btn btn--primary btn--lg" type="button">Get started</button>
</section>
```
