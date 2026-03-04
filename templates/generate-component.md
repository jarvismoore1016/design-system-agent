---
description: Generate a new UI component using the project's design tokens and conventions
allowed-tools: Read, Write, Bash
argument-hint: component-name
---

# Generate Component: $1

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
