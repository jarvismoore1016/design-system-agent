---
description: Generate structured markdown documentation for UI components — single component or all components at once
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

Read `CLAUDE.md` for project context (stack, component locations, existing docs folder).

### Step 1 — Find all components

Scan for component files in standard locations:
- `src/components/`, `components/`, `src/ui/`, `src/lib/`
- Look for `.tsx`, `.jsx`, `.vue`, `.svelte`, `.html` files with component-like names (capitalized, or index files inside named directories)

Deduplicate and list them.

### Step 2 — Confirm the list

Present via AskUserQuestion with `multiSelect: true`:
- question: "Which components would you like to document?"
- header: "Components"
- options: [list all found component names]

### Step 3 — Document each confirmed component

For each confirmed component:
1. Read the component source file (and any associated CSS/SCSS file)
2. Read the token file to identify which tokens the component uses
3. Write `docs/components/[component-name].md` following the Single Mode structure below
4. Create the `docs/components/` directory if it doesn't exist

After each doc:
```
✓ docs/components/button.md
✓ docs/components/card.md
✓ docs/components/badge.md
```

### Step 4 — Summary

```
Documentation complete.

8 components documented:
  docs/components/button.md
  docs/components/card.md
  docs/components/badge.md
  docs/components/input.md
  docs/components/modal.md
  docs/components/nav.md
  docs/components/avatar.md
  docs/components/spinner.md

Run /audit-ui to check these components for token drift and accessibility issues.
```

---

## Single Mode — Document One Component

## Before You Write

Read `CLAUDE.md` for project context — tech stack, token naming convention, and component conventions.

Find the component file for `$1`. Check these locations in order:
- Direct file path if `$1` looks like a path (`src/components/Button.tsx`)
- `src/components/[name].[ext]`
- `components/[name].[ext]`
- `src/ui/[name].[ext]`
- Any directory named after the component

Read the component file carefully. If there's a corresponding CSS/SCSS file, read that too.

Read the token file to understand which tokens the component uses.

## Documentation to Write

Save the output to `docs/components/[component-name].md`. Create the directory if it doesn't exist. If the project already has a `docs/` folder somewhere, use that instead.

---

### Title and Status Badge

Start with: `# [Component Name]`

Add a brief status note if relevant (e.g., "Stable — safe to use in production" or "In progress — API may change").

---

### Overview

2-4 sentences covering:
- What this component is
- When to use it
- When NOT to use it (what to use instead in those cases)

---

### Variants

A table or description of every visual/functional variant. For each variant, explain when it should be used, not just what it looks like.

Example:
| Variant | When to use |
|---------|-------------|
| Primary | The main action on a page or section — use once per context |
| Secondary | Supporting actions that need less visual weight |
| Ghost | Tertiary actions or where the button sits on a colored background |
| Destructive | Irreversible actions — delete, remove, clear all |

---

### States

Table showing the visual and behavioral state of the component:

| State | Description |
|-------|-------------|
| Default | Resting appearance |
| Hover | Mouse cursor over element — [describe change] |
| Focus | Keyboard focus — [describe focus indicator] |
| Active | During click or press — [describe change] |
| Disabled | Not interactive — [describe appearance, note `disabled` attribute] |

Add component-specific states if applicable (loading, error, success, selected, expanded, etc.).

---

### Props / API

**For React, Vue, Svelte, Angular components:**

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `variant` | `'primary' \| 'secondary' \| 'ghost' \| 'destructive'` | `'primary'` | Visual style |
| `size` | `'sm' \| 'md' \| 'lg'` | `'md'` | Component size |
| `disabled` | `boolean` | `false` | Disables interaction |
| `onClick` | `() => void` | — | Click handler |
| ... | | | |

**For vanilla HTML components:**

| Class / Attribute | Values | Description |
|-------------------|--------|-------------|
| `.btn` | — | Base class, required |
| `.btn--primary` | — | Primary variant |
| `.btn--sm` | — | Small size modifier |
| `disabled` | boolean attribute | Disables the button |
| ... | | |

---

### Design Tokens Used

List which tokens from the design system this component uses, and briefly explain the role of each:

| Token | Role |
|-------|------|
| `--color-primary` | Background color of primary variant |
| `--color-text-on-primary` | Text color on primary background |
| `--spacing-sm` | Vertical padding |
| `--spacing-md` | Horizontal padding |
| `--radius-sm` | Border radius |
| `--font-size-sm` | Small variant font size |

---

### Accessibility

- **Element**: Which HTML element is used and why (e.g., "`<button>` — provides keyboard interaction and click events natively")
- **ARIA**: List any ARIA attributes used and why
- **Keyboard**: Describe keyboard interaction (e.g., "Enter and Space activate the button")
- **Screen reader**: Describe how it's announced (e.g., "Announced as 'button' with visible label text")
- **Focus**: Describe the focus indicator (e.g., "2px outline using `--color-focus-ring`, always visible")
- **Notes**: Any accessibility gotchas specific to this component

---

### Do's and Don'ts

**Do:**
- [Specific, concrete guidance based on this component's design]
- [e.g., "Use one primary button per page section"]
- [e.g., "Use a descriptive label — 'Save changes', not 'Submit'"]

**Don't:**
- [e.g., "Don't use primary for destructive actions — use the destructive variant"]
- [e.g., "Don't disable buttons to hide unavailable actions — explain why instead"]
- [e.g., "Don't use a button where a link is more appropriate (navigating to a new page)"]

---

### Usage Examples

Provide 2-3 ready-to-use code examples covering the most common cases. Use the project's actual stack format.

**Example 1: Basic usage**
```[language]
[code]
```

**Example 2: With variants**
```[language]
[code]
```

**Example 3: In context (e.g., inside a form)**
```[language]
[code]
```

---

## After Writing

Tell the user:
- The path to the generated documentation file
- How many sections were included
- Suggest running `/generate-docs all` if they want to document additional components
