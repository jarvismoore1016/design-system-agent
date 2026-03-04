---
description: Build a complete web design system from scratch — tokens, components, docs, and CLAUDE.md — no existing codebase required
allowed-tools: AskUserQuestion, Bash, Read, Write, Glob, Grep
---

# Build Design System from Scratch

You are guiding the user through a six-step wizard to create a complete web design system. There is no existing codebase required — this generates everything fresh based on their project goals.

At the end, the user will have:
- A complete design token file (colors, spacing, typography, radii, shadows)
- All selected components — fully built, token-using, accessible, all states
- `docs/components/[name].md` for each component
- `CLAUDE.md` with persistent design system context

---

## Opening Message

Begin by saying:

> **Build Design System — From Scratch**
>
> I'm going to help you build a complete design system for a new project. By the end you'll have a token file, all your selected components built and documented, and a CLAUDE.md that gives me persistent context for every future session.
>
> Six steps. I'll ask for your preferences at each stage, then generate everything. Let's start.

---

## Step 1 — Project Type and Style

### Ask for project type

Use AskUserQuestion:
- question: "What type of project is this for?"
- header: "Project type"
- options:
  - Dashboard / admin app
  - Marketing site
  - E-commerce
  - SaaS app
  - Blog / content site

Then ask for design style:

Use AskUserQuestion:
- question: "What's the overall design direction?"
- header: "Design style"
- options:
  - Minimal / clean (lots of whitespace, neutral palette, thin weights)
  - Bold / expressive (strong color, generous scale, heavier weights)
  - Enterprise / neutral (dense layout, muted palette, functional)

Then ask for a brand color:

Use AskUserQuestion:
- question: "What's your primary brand color? Choose a starting point — you can always update the token file afterward."
- header: "Brand color"
- options:
  - Blue (#2563EB — default / safe)
  - Purple (#7C3AED — product / SaaS)
  - Green (#059669 — growth / health)
  - Orange (#EA580C — energy / consumer)
  - I'll enter my own

If they select "I'll enter my own", ask them to type their hex value in chat.

Store all three answers — you'll use them to generate the token file in Step 2.

---

## Step 2 — Generate Tokens from Scratch

Say: "Now I'll generate a complete token set based on your choices. I'll build a primary palette, neutral grays, semantic colors, spacing scale, typography scale, radii, and shadows."

### Generate the token set

Based on the project type, design style, and primary brand color, derive all token values:

**Primary color palette** — generate 9 shades (50–900) of the primary brand color:
- 50: very light tint (mix ~5% of primary into white)
- 100, 200, 300: progressively deeper tints
- 400: lighter-than-base mid tone
- 500: the base brand color
- 600, 700: darker variants (used for hover and active states)
- 800, 900: very dark (backgrounds, high-contrast text)

**Secondary color** — derive a complementary secondary:
- Minimal/clean: desaturated teal or slate adjacent to primary
- Bold/expressive: complementary hue (split-complement or triadic)
- Enterprise: a cool neutral (slate or stone)
Generate 5 shades: 100, 300, 500, 700, 900

**Neutral grays** — 9 shades (50–900):
- Minimal/clean and enterprise: cool grays (slate family)
- Bold/expressive: warm grays (zinc or stone family)

**Semantic colors** — 2–3 shades each:
- Success: green family (#10B981 base for 500)
- Warning: amber family (#F59E0B base for 500)
- Danger: red family (#EF4444 base for 500)
- Info: blue/cyan family (#0EA5E9 base for 500)

**Spacing scale** — 4px base unit:
- xs: 4px | sm: 8px | md: 16px | lg: 24px | xl: 32px | 2xl: 48px | 3xl: 64px | 4xl: 96px

**Typography scale**:
- Font family: Inter, system-ui, -apple-system, sans-serif (or ask if they have a brand font)
- Base size: 16px (14px for enterprise/dashboard)
- Scale: xs(12px), sm(14px), base(16px), lg(18px), xl(20px), 2xl(24px), 3xl(30px), 4xl(36px), 5xl(48px)
- Weights: normal(400), medium(500), semibold(600), bold(700)
- Line heights: tight(1.25), snug(1.375), normal(1.5), relaxed(1.625)
- Letter spacing: tight(-0.025em), normal(0), wide(0.025em)

**Border radii**:
- Minimal/clean: none(0), sm(2px), md(4px), lg(8px), xl(12px), full(9999px)
- Bold/expressive: sm(4px), md(8px), lg(16px), xl(24px), full(9999px)
- Enterprise: sm(2px), md(4px), lg(6px), full(9999px)

**Shadows**:
- sm: `0 1px 2px 0 rgb(0 0 0 / 0.05)`
- md: `0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1)`
- lg: `0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)`
- xl: `0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)`

**Focus ring**: `--color-focus-ring` at 50% opacity of primary-500, `--focus-ring-width: 3px`, `--focus-ring-offset: 2px`

### Ask naming preference

Use AskUserQuestion:
- question: "How would you like to name your tokens?"
- header: "Token naming"
- options:
  - Semantic (--color-primary, --spacing-md, --text-base)
  - Numeric scale (--color-blue-500, --spacing-4, --text-lg)
  - Keep it simple (I can rename them later)

### Write the token file

Write `tokens.css` in the project root:

```css
/* Design Tokens — generated by Design System Assistant */
/* Project: [project type] | Style: [design style] | Primary: [hex] */

:root {
  /* === Primary Palette === */
  --color-primary-50: [value];
  --color-primary-100: [value];
  --color-primary-200: [value];
  --color-primary-300: [value];
  --color-primary-400: [value];
  --color-primary-500: [brand hex];
  --color-primary-600: [value];
  --color-primary-700: [value];
  --color-primary-800: [value];
  --color-primary-900: [value];

  /* Alias: default interactive primary */
  --color-primary: var(--color-primary-600);
  --color-primary-hover: var(--color-primary-700);
  --color-primary-light: var(--color-primary-50);

  /* === Secondary Palette === */
  --color-secondary-100: [value];
  --color-secondary-300: [value];
  --color-secondary-500: [value];
  --color-secondary-700: [value];
  --color-secondary-900: [value];

  /* === Neutral Grays === */
  --color-neutral-50: [value];
  --color-neutral-100: [value];
  --color-neutral-200: [value];
  --color-neutral-300: [value];
  --color-neutral-400: [value];
  --color-neutral-500: [value];
  --color-neutral-600: [value];
  --color-neutral-700: [value];
  --color-neutral-800: [value];
  --color-neutral-900: [value];

  /* === Semantic Colors === */
  --color-success-light: [value];
  --color-success: [value];
  --color-success-dark: [value];
  --color-warning-light: [value];
  --color-warning: [value];
  --color-warning-dark: [value];
  --color-danger-light: [value];
  --color-danger: [value];
  --color-danger-dark: [value];
  --color-info-light: [value];
  --color-info: [value];
  --color-info-dark: [value];

  /* === Text Colors === */
  --color-text-primary: var(--color-neutral-900);
  --color-text-secondary: var(--color-neutral-600);
  --color-text-muted: var(--color-neutral-400);
  --color-text-on-primary: #ffffff;
  --color-text-link: var(--color-primary);
  --color-text-link-hover: var(--color-primary-hover);

  /* === Background Colors === */
  --color-bg-primary: #ffffff;
  --color-bg-secondary: var(--color-neutral-50);
  --color-bg-tertiary: var(--color-neutral-100);
  --color-border: var(--color-neutral-200);
  --color-border-strong: var(--color-neutral-300);

  /* === Spacing === */
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;
  --spacing-2xl: 48px;
  --spacing-3xl: 64px;
  --spacing-4xl: 96px;

  /* === Typography === */
  --font-family-base: Inter, system-ui, -apple-system, BlinkMacSystemFont, sans-serif;
  --font-size-xs: 0.75rem;    /* 12px */
  --font-size-sm: 0.875rem;   /* 14px */
  --font-size-base: 1rem;     /* 16px */
  --font-size-lg: 1.125rem;   /* 18px */
  --font-size-xl: 1.25rem;    /* 20px */
  --font-size-2xl: 1.5rem;    /* 24px */
  --font-size-3xl: 1.875rem;  /* 30px */
  --font-size-4xl: 2.25rem;   /* 36px */
  --font-size-5xl: 3rem;      /* 48px */

  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;

  --line-height-tight: 1.25;
  --line-height-snug: 1.375;
  --line-height-normal: 1.5;
  --line-height-relaxed: 1.625;

  --letter-spacing-tight: -0.025em;
  --letter-spacing-normal: 0em;
  --letter-spacing-wide: 0.025em;

  /* === Border Radius === */
  --radius-none: 0px;
  --radius-sm: [value];
  --radius-md: [value];
  --radius-lg: [value];
  --radius-xl: [value];
  --radius-full: 9999px;

  /* === Shadows === */
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
  --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1);

  /* === Focus Ring === */
  --color-focus-ring: [primary-500 at 50% opacity as rgba];
  --focus-ring-width: 3px;
  --focus-ring-offset: 2px;
}
```

After writing, show the user a brief preview and the file path. Tell them they can edit it directly at any time.

---

## Step 3 — Select Components

Say: "Now let's decide which components to build. Here's a standard starter set — pick what fits your project. You can always generate more later with `/generate-component`."

Use AskUserQuestion with `multiSelect: true`:
- question: "Which components should I generate for this project?"
- header: "Component list"
- options:
  - Button
  - Input
  - Card
  - Badge
  - Modal
  - Nav
  - Avatar
  - Spinner
  - Toast
  - Checkbox
  - Select
  - Tabs

Store the confirmed list — you'll loop through it in Step 4.

---

## Step 4 — Generate All Components

Say: "Generating [N] components. Each one will have all variants, all interactive states, full accessibility attributes, and use only token references — no hardcoded values."

For each component in the confirmed list:

1. Generate a complete component appropriate for vanilla HTML/CSS (since there's no existing stack to detect). If any project files were found, match their format instead.
2. Every color, spacing, size, and radius value must reference `var(--token-name)` from `tokens.css`
3. Include all interactive states: default, hover, focus, active, disabled
4. Full accessibility: correct semantic element, ARIA attributes, keyboard support, visible focus indicator meeting WCAG AA (3:1 contrast for focus indicator)
5. Save HTML to `src/components/[name].html` and CSS to `src/components/[name].css` (or the detected format)

After generating each component, print a one-line status:
```
✓ Button — src/components/button.html + src/components/button.css
✓ Input — src/components/input.html + src/components/input.css
...
```

---

## Step 5 — Document All Components

Say: "Now I'll generate documentation for each component."

For each component generated in Step 4, write `docs/components/[component-name].md`.

Each doc must include:
- Overview: what it is, when to use it, when not to
- Variants: table with each variant and its use case
- States: table showing each state and visual/behavioral description
- Props / API: attributes, classes, or props the component accepts
- Design Tokens Used: table of tokens and their role in this component
- Accessibility: semantic element, ARIA attributes, keyboard interactions, screen reader behavior
- Do's and Don'ts: 3–4 bullets each, specific to this component
- Usage Examples: 2–3 ready-to-use code examples

After each doc:
```
✓ docs/components/button.md
✓ docs/components/input.md
...
```

---

## Step 6 — Write CLAUDE.md and Summary

Say: "Last step. Writing CLAUDE.md — persistent context for every future session."

Write `CLAUDE.md` in the project root with actual values (no placeholders):

```markdown
# [Project Type] Design System

> Generated by Design System Assistant.
> Update this file whenever conventions change.

## Project
- **Type**: [project type]
- **Style**: [design style]
- **Stack**: Vanilla HTML/CSS (or detected framework)

## Design Tokens
- **Token file**: `tokens.css`
- **Naming convention**: [semantic / numeric scale]
- **Primary color**: [hex] — used for primary actions, interactive elements, focus rings
- **Primary palette**: --color-primary-500 ([hex]), --color-primary-600 ([hex]), --color-primary-700 ([hex])
- **Neutral grays**: --color-neutral-100 ([value]) through --color-neutral-900 ([value])
- **Semantic**: success ([value]), warning ([value]), danger ([value]), info ([value])
- **Spacing**: xs(4px), sm(8px), md(16px), lg(24px), xl(32px), 2xl(48px), 3xl(64px), 4xl(96px)
- **Typography**: [font family], base [size]px, scale from xs(12px) to 5xl(48px)
- **Border radii**: sm([value]), md([value]), lg([value]), xl([value]), full(9999px)
- **Shadows**: sm / md / lg / xl

## Generated Components
[list each component with its file path]

## Documentation
Component docs in `docs/components/`:
[list each doc file]

## Available Slash Commands
- `/audit-ui [path]` — Scan for token drift, accessibility issues, inconsistencies
- `/generate-component [name|all|scan]` — Generate one component, all components, or scan for patterns to extract
- `/generate-docs [name|all]` — Document one component or all components
- `/build-design-system` — Regenerate the design system from scratch

## Accessibility Target
WCAG 2.1 AA compliance. Visible focus on all interactive elements. All form inputs labeled. Minimum 4.5:1 contrast for normal text, 3:1 for large text and UI components.
```

### Present the final summary

> **Design system built. Here's what was created:**
>
> | File | What it is |
> |------|-----------|
> | `tokens.css` | Complete token set — [N] color shades, spacing scale, typography, radii, shadows |
> | `src/components/[name].[ext]` × [N] | All selected components, fully built |
> | `docs/components/[name].md` × [N] | Structured documentation for each |
> | `CLAUDE.md` | Persistent context for every future session |
>
> **What to do next:**
> 1. Review `tokens.css` and adjust any values that don't match your brand exactly
> 2. Open one generated component and check it renders correctly in your environment
> 3. Run `/audit-ui` after you add real content to catch any drift
> 4. Use `/generate-component [name]` to add more components as you build
