---
description: Generate UI components using the project's design tokens and conventions — single component, all components, or scan for patterns to extract
allowed-tools: Read, Write, Bash, AskUserQuestion, Glob, Grep
argument-hint: component-name | all | scan
---

# Generate Component: $1

## Mode Detection

Check the value of `$1`:

- **`$1` is `all`** → follow **All Mode** below — generate every component in the project
- **`$1` is `scan`** → follow **Scan Mode** below — find unextracted UI patterns
- **Anything else** → treat `$1` as a component name and follow **Single Mode** (the rest of this document)

---

## All Mode — Generate Full Component Library

Read `CLAUDE.md` for project context (stack, token file, conventions).

### Step 1 — Find all components

Scan for component files in standard locations:
- `src/components/`, `components/`, `src/ui/`, `src/lib/`
- Look for files with component-like names: capitalized (`.tsx`, `.jsx`, `.vue`, `.svelte`) or index files inside named directories

Build a deduplicated list of component names.

### Step 2 — Confirm the list

Present the full list via AskUserQuestion with `multiSelect: true`:
- question: "Which components would you like to generate (or regenerate)?"
- header: "Components"
- options: [list all found component names]

### Step 3 — Generate each confirmed component

For each confirmed component:
1. Read the existing file to understand the current API and variants (if it exists)
2. Generate a complete version using the project's tokens and conventions
3. Match all existing props and variants exactly — do not change the component's API
4. Include all interactive states (default, hover, focus, active, disabled)
5. Every value must reference a token — no hardcoded colors, spacing, or sizing
6. Save to the same file location (overwrite) or to `src/components/[name].[ext]` for new components

After each component:
```
✓ Button — src/components/Button.tsx (regenerated)
✓ Card — src/components/Card.tsx (regenerated)
✓ Badge — src/components/Badge.tsx (regenerated)
```

### Step 4 — Summary table

| Component | Status | File |
|-----------|--------|------|
| Button | Generated | src/components/Button.tsx |
| Card | Generated | src/components/Card.tsx |
| Badge | Generated | src/components/Badge.tsx |

---

## Scan Mode — Find Unextracted Patterns

Read `CLAUDE.md` for project context.

### Step 1 — Scan for repeated patterns

Grep across all source files (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`) for:

**Inline styles with repeated values**:
- `style="..."` attributes where the same property+value appears 3+ times across files
- `StyleSheet.create()` blocks with values that appear in multiple component files

**Duplicated DOM/JSX structures**:
- The same sequence of HTML elements (e.g., `<div class="card"> + <img> + <h3> + <p>`) appearing in 3+ different files
- Similar component-like JSX structures that aren't yet extracted (e.g., repeated `<div className="badge ...">` patterns)

**Repeated class name clusters**:
- The same group of utility classes used together in 4+ places (e.g., `"flex items-center gap-2 rounded-full px-3 py-1 text-sm"`)
- CSS class names that follow a component pattern (`.card`, `.shoe-card`, `.product-card`) but are implemented separately

### Step 2 — Present findings

Show each pattern as a suggested component extraction:

```
Suggested component extractions:

1. ProductCard — card structure with image, title, and price appears in 6 files
   Files: ShoeCard.tsx, CollectionItem.tsx, SearchResult.tsx, ...
   Pattern: <div className="card"> <img> <h3> <p className="price"> <button>

2. PageHeader — title + subtitle pattern appears in 4 files
   Files: HomePage.tsx, AboutPage.tsx, CollectionPage.tsx, ContactPage.tsx
   Pattern: <div className="page-header"> <h1> <p className="subtitle">

3. TagBadge — repeated class cluster "badge rounded-full px-2 py-1 text-xs" in 7 places
```

### Step 3 — Ask what to generate

Use AskUserQuestion with `multiSelect: true`:
- question: "Which patterns should I extract into reusable components?"
- header: "Extract"
- options: [list each suggested component name]

### Step 4 — Generate each selected component

For each selected pattern:
1. Extract the shared structure from the source files
2. Replace all hardcoded values with token references
3. Identify the variants present across the implementations and expose them as props
4. Add all interactive states and accessibility attributes
5. Save to `src/components/[name].[ext]`

After generating each:
```
✓ ProductCard — src/components/ProductCard.tsx
  Props: image, title, price, variant ('default' | 'featured'), onPress
  Found in: 6 files — consider replacing inline implementations with this component
```

---

## Single Mode — Generate One Component

## Before You Build

Read `CLAUDE.md` to understand:
- Token file path and naming convention
- Tech stack and output file format
- Existing component patterns to follow (naming, file structure, CSS approach)

Then read the token file so you know exactly what values are available. Every color, spacing value, and size in the generated component must come from the token system.

Find 1-2 existing components in the project and read them. Match their conventions exactly — don't invent a new pattern.

## What to Generate

Build a complete `$1` component that's ready to use without modification.

### Uses tokens for everything

No hardcoded hex values, no magic numbers for spacing or sizing. Reference the token variables directly:
- CSS: `var(--color-primary)`, `var(--spacing-md)`
- SCSS: `$color-primary`, `$spacing-md`
- JS/TS tokens: import from the tokens file

### Includes all interactive states

Every state must be visually distinct:
- **Default** — resting appearance
- **Hover** — cursor change + subtle visual feedback
- **Focus** — clearly visible outline or indicator (never `outline: none` without replacement)
- **Active / Pressed** — feedback during click/press
- **Disabled** — visually muted, `disabled` attribute or `aria-disabled="true"`, no pointer events

### Is accessible by default

- Use the correct semantic HTML element for the role (e.g., `<button>` for actions, `<a>` for navigation)
- Add ARIA attributes where native semantics aren't sufficient
- Support keyboard interaction (Enter and Space for buttons, Tab for focus)
- Focus indicator meets WCAG AA (3:1 contrast against adjacent colors)

### Follows project conventions

- File name matches the existing naming pattern (`Button.tsx`, `button.vue`, `Button.svelte`, etc.)
- Class names follow existing conventions (BEM, camelCase, kebab-case — match what's already there)
- Variant/prop pattern matches existing components

## Output Format by Stack

### Vanilla HTML/CSS

Provide:
1. An HTML snippet showing the component's markup with all variants
2. A CSS block with all states using `var(--token-name)` references

### React (JavaScript)

Provide a `.jsx` file:
- Functional component with `propTypes`
- CSS module import (`.module.css`) with all states
- Default props defined
- Full JSDoc comment at the top

### React (TypeScript)

Provide a `.tsx` file:
- Functional component with TypeScript props interface
- CSS module import (`.module.css`) with all states
- Strict typing — no `any`

### Vue

Provide a `.vue` Single File Component:
- `<template>` with semantic HTML
- `<script setup lang="ts">` with `defineProps`
- `<style scoped>` with token references

### Svelte

Provide a `.svelte` file:
- `<script>` with exported props
- Markup with semantic HTML
- `<style>` with token references

## After the Component

Save the file to the appropriate location based on existing project structure (usually `src/components/`, `components/`, or `src/ui/`).

Add a brief usage example as a comment at the end of the file.

Tell the user:
- Where the file was saved
- What props/variants it supports
- How to import and use it
- Which tokens it references (so they know what to update if the design changes)
