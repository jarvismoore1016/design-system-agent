---
description: Interactive guided setup for your design system assistant — creates tokens, audit, generation, and documentation commands tailored to your project
allowed-tools: AskUserQuestion, Bash, Read, Write, Glob, Grep
---

# Design System Assistant Setup

You are guiding the user through a six-step setup process. At every step, explain what you are doing and why **before** you do it. Tone: knowledgeable colleague, not tutorial voice. The user should feel like they're building something together with you, not watching a script run.

At the end, the user will have:
- A design token file (all their existing colors, spacing, and typography — named and organized)
- `/audit-ui` — scans for visual inconsistencies and accessibility issues
- `/generate-component` — builds new components from the token system
- `/generate-docs` — documents components in structured markdown
- `CLAUDE.md` — persistent project context for all future sessions

---

## Opening Message

Begin by saying:

> **Design System Assistant Setup**
>
> I'm going to set up an AI-powered design system assistant for your project. By the end you'll have a token file, three slash commands, and a CLAUDE.md that keeps me informed about your design system in every future session.
>
> Six steps, 2-4 minutes each. I'll explain what I'm doing at each stage and ask for your input on the decisions that matter. Let's go.

---

## Step 1 — Project Detection

Say: "First, let me look at your project to understand what we're working with."

### Scan for tech stack

Check for `package.json`. If found, read it and look for these in `dependencies` and `devDependencies`:
- `react`, `react-dom` → React
- `vue` → Vue
- `svelte`, `@sveltejs/kit` → Svelte
- `@angular/core` → Angular
- `next` → Next.js
- `astro` → Astro
- `nuxt` → Nuxt
- `gatsby` → Gatsby

Check for config files:
- `vite.config.*`, `webpack.config.*` → note bundler
- `tailwind.config.*` → Tailwind CSS in use
- `next.config.*` → Next.js confirmed
- `angular.json` → Angular confirmed

Glob for source file extensions to confirm:
- `.tsx`, `.jsx` → React
- `.vue` → Vue
- `.svelte` → Svelte
- `.html` (with no framework deps) → Vanilla HTML/CSS

Check for CSS preprocessors:
- `.scss` or `.sass` files → SCSS in use
- `.less` files → Less in use
- `.module.css` files → CSS Modules

Check for existing token artifacts:
- `tokens.css`, `tokens.scss`, `tokens.js`, `tokens.ts`, `design-tokens.json`
- `variables.css`, `_variables.scss`
- Any file containing `:root {` with custom properties
- Any file with `$color-` or `--color-` patterns

Count the total number of source files (HTML, CSS, SCSS, JS, TS, JSX, TSX, Vue, Svelte).

### Present findings

Summarize what you found, for example:
> Here's what I found in your project:
> - **Framework**: React with TypeScript
> - **CSS**: SCSS modules
> - **Existing tokens**: None detected
> - **Source files scanned**: 52 files

Use AskUserQuestion:
- question: "Does this look right? I'll use this to customize everything I create."
- header: "Stack check"
- options:
  - Looks right (proceed)
  - Something's off (let me correct it)

If they say something's off, ask:
- question: "What's your primary tech stack?"
- header: "Framework"
- options:
  - Vanilla HTML/CSS
  - React
  - Vue
  - Other (Svelte, Angular, Astro, etc.)

Store the confirmed tech stack — you'll need it in every subsequent step.

---

## Step 2 — Token Extraction and Creation

Say: "Now I'll scan your project for all the design values currently in use. Most projects have the same color appearing ten different ways, spacing values that almost-but-don't-quite match, font sizes with no clear pattern. We're going to name them, consolidate them, and give you a single source of truth."

### Scan for values

Grep across CSS, SCSS, Less, HTML, JS, TS, JSX, TSX, Vue, and Svelte files for:

**Colors**:
- Hex: `#[0-9a-fA-F]{3,8}\b`
- RGB/RGBA: `rgba?\(\s*[\d,\s.%]+\)`
- HSL/HSLA: `hsla?\(\s*[\d,\s.%]+\)`
- Named CSS colors used repeatedly: `\b(white|black|red|blue|green|gray|grey)\b`

**Spacing** (look in `margin`, `padding`, `gap`, `top`, `right`, `bottom`, `left`, `width`, `height` declarations):
- `\b\d+(\.\d+)?(px|rem|em)\b`

**Typography**:
- `font-size:\s*[\d.]+(?:px|rem|em)`
- `font-weight:\s*\d+`
- `font-family:\s*[^;,}]+`
- `line-height:\s*[\d.]+(?:px|rem|em|)`

**Border radius**:
- `border-radius:\s*[\d.]+(?:px|rem|%)`

**Shadows**:
- `box-shadow:\s*[^;]+`

Deduplicate all values. For colors, group similar hues together (multiple blues, multiple grays, etc.). For spacing, sort numerically.

### Present findings

Report what you found:
> Here's what I found in your project:
> - **Colors**: 16 unique values — 5 blue variants, 4 grays, 3 neutrals, 4 others
> - **Spacing**: 10 unique values (8px through 64px)
> - **Font sizes**: 7 unique values (12px through 40px)
> - **Font weights**: 3 (400, 600, 700)
> - **Border radii**: 4 values (4px, 8px, 12px, 24px)
> - **Shadows**: 2 unique values

### Ask for naming preference

Use AskUserQuestion:
- question: "How would you like to name your tokens?"
- header: "Token style"
- options:
  - Semantic (--color-primary, --color-text, --spacing-md)
  - Numeric scale (--color-blue-500, --spacing-4, --text-lg)
  - Keep it simple (I'll rename them later)

### Write the token file

Based on confirmed tech stack and naming preference, generate the token content and write the file:

**Vanilla HTML/CSS** → write `tokens.css`:
```css
/* Design Tokens — generated by Design System Assistant */
:root {
  /* Colors */
  --color-[name]: [value];
  ...

  /* Spacing */
  --spacing-[name]: [value];
  ...

  /* Typography */
  --font-size-[name]: [value];
  --font-weight-[name]: [value];
  --font-family-base: [value];
  ...

  /* Border Radius */
  --radius-[name]: [value];
  ...

  /* Shadows */
  --shadow-[name]: [value];
  ...
}
```

**SCSS project** → write `_tokens.scss`:
```scss
// Design Tokens — generated by Design System Assistant

// Colors
$color-[name]: [value];
...

// Spacing
$spacing-[name]: [value];
...

// Typography
$font-size-[name]: [value];
...
```

**React/Vue/Next.js/JS/TS project** → write `tokens.js` or `tokens.ts`:
```js
// Design Tokens — generated by Design System Assistant

export const colors = {
  [name]: '[value]',
  ...
};

export const spacing = {
  [name]: '[value]',
  ...
};

export const typography = {
  fontSize: { [name]: '[value]', ... },
  fontWeight: { [name]: [value], ... },
  fontFamily: { base: '[value]' },
};

export const radius = { [name]: '[value]', ... };
export const shadows = { [name]: '[value]', ... };
```

After writing, show the user a preview of the file and its path. Tell them they can edit it directly any time.

---

## Step 3 — Audit Command

Say: "Now I'll create your `/audit-ui` command. Run this any time you want a snapshot of where your project stands on visual consistency and accessibility. It checks against the token file we just created, so it knows what values are intentional and what's drift."

Write the file `.claude/commands/audit-ui.md`:

```markdown
---
description: Audit the UI for visual consistency, token drift, and accessibility issues
allowed-tools: Read, Glob, Grep, Bash, AskUserQuestion, Write
---

# UI Audit

Scan the project (or the path provided as $1 if given) for design system issues. Report findings grouped by category with file and line references. Be specific — name the file, the value, and what it should be instead.

## Context

Read CLAUDE.md for project context including:
- Token file location and available token names
- Tech stack and file conventions
- Known design patterns

## What to Check

### 1. Token Drift
Find hardcoded color, spacing, border-radius, and shadow values that duplicate or approximate values in the token file.

For each finding, show:
- File and line number
- The hardcoded value found
- The token it should use instead

### 2. Accessibility (WCAG 2.1 AA)
Check for the following. Tag each finding with its WCAG criterion.

- Color contrast [WCAG 1.4.3]: normal text 4.5:1, large text (18px+ or 14px bold) 3:1
- UI component contrast [WCAG 1.4.11]: borders, icons, and interactive components must meet 3:1
- Form labels [WCAG 1.3.1]: every input, select, textarea must have a programmatic label — placeholder alone is not sufficient
- Focus visible [WCAG 2.4.7]: outline: none without a :focus-visible replacement is a violation
- Keyboard navigability [WCAG 2.1.1]: all interactive elements reachable by Tab; custom controls have tabindex="0" and keyboard handlers
- No keyboard traps [WCAG 2.1.2]: modals trap focus and have a keyboard-accessible close mechanism
- Text alternatives [WCAG 1.1.1]: img elements have alt, icon-only buttons have aria-label
- Non-descriptive link/button text [WCAG 2.4.6]: "click here", "read more", "here" are violations
- Error identification [WCAG 3.3.1]: form errors described in text, not only by color or icon
- Missing ARIA [WCAG 4.1.2]: custom interactive elements missing role, aria-expanded, aria-controls, aria-haspopup

Tag each finding: [ACCESSIBILITY] path/to/file:line [WCAG X.X.X]

### 3. Component Inconsistency
Look for the same UI pattern implemented multiple ways, similar class names suggesting parallel implementations, and inline style attributes that belong in shared classes or tokens.

### 4. Contrast (Flag for Review)
Flag text + background combinations likely to fail WCAG AA. Estimate the ratio when values are known. Note as "needs manual contrast check" when exact values can't be determined statically.

## Output Format

Group findings by category. End with a summary table:

| Category | Critical | Warning | Suggestion |
|----------|----------|---------|------------|
| Token drift | - | - | - |
| Accessibility | - | - | - |
| Inconsistency | - | - | - |
| Contrast | - | - | - |
| **Total** | | | |

Finish with 3-5 prioritized next steps.

## Interactive Fix Mode

After the summary table, use AskUserQuestion with multiSelect: true to ask which categories to fix:
- Token drift
- Accessibility
- Inconsistency
- Contrast

For each selected category, go through findings one by one. Before each fix, show the finding and use AskUserQuestion (options: Yes / Skip / Stop). If Yes: apply the fix and show a brief diff. After each category: show a count of fixes applied vs skipped.
```

After writing the file, tell the user what was created and how to use it. Use AskUserQuestion:
- question: "Would you like to adjust what the audit checks for?"
- header: "Audit scope"
- options:
  - Looks good (continue to next step)
  - Add more checks (I want to flag something specific)
  - Remove some checks (not all of these apply)

If they want adjustments, ask what they'd like to change and edit the command accordingly before moving on.

---

## Step 4 — Component Generation Command

Say: "Now I'll create your `/generate-component` command. This is where things get useful fast — you give it a component name and it builds a complete, accessible, token-using component in your stack's format. No starting from scratch, no remembering what tokens to use."

First, quickly scan the codebase to identify the 2-3 most common UI patterns (look for repeated class names or file names containing "button", "card", "input", "badge", "modal", "nav").

Write the file `.claude/commands/generate-component.md`:

```markdown
---
description: Generate UI components using the project's design tokens and conventions
allowed-tools: Read, Write, Bash, AskUserQuestion, Glob, Grep
argument-hint: component-name | all | scan
---

# Generate Component: $1

## Mode Detection

- $1 is `all` → generate every component (All Mode)
- $1 is `scan` → find unextracted patterns (Scan Mode)
- Anything else → generate one component (Single Mode)

## All Mode

Read CLAUDE.md. Scan src/components/, components/, src/ui/ for component files. Present the full list via AskUserQuestion (multiSelect: true). For each confirmed component: read the existing file, regenerate a complete version using project tokens and conventions, preserve the existing API. Save to the same location. End with a summary table of components generated.

## Scan Mode

Read CLAUDE.md. Grep across source files for repeated patterns: inline styles with the same property+value appearing 3+ times, duplicated JSX/HTML structures in 3+ files, repeated class name clusters used together in 4+ places. Present findings as suggested components to extract. Ask via AskUserQuestion (multiSelect: true) which ones to generate. For each selected: extract the shared structure, replace hardcoded values with tokens, identify variants, add states and accessibility. Save to src/components/.

## Single Mode

Read CLAUDE.md first for:
- Token file path and token naming convention
- Tech stack and output file format
- Existing component patterns to follow

Then read the token file. Find 1-2 existing components and match their conventions.

Create a complete $1 component that:
- Uses tokens for everything — no hardcoded colors, spacing, or sizing
- Includes all interactive states: default, hover, focus (never outline: none), active/pressed, disabled
- Is accessible: correct semantic element, ARIA attributes, keyboard interaction, visible focus indicator
- Follows project conventions: naming, file structure, prop/variant pattern

Output format by stack:
- Vanilla HTML/CSS: HTML snippet + CSS block using var(--token-name)
- React JS/TS: .jsx/.tsx functional component with CSS module
- Vue: Single File Component with template, script setup, style scoped
- Svelte: .svelte file with script, markup, style

Add a usage example as a comment. Tell the user where the file was saved and how to import/use it.
```

After writing, demonstrate by generating one component based on the most common pattern found in the codebase. Show the user the output.

---

## Step 5 — Documentation Command

Say: "Last command. `/generate-docs` produces structured documentation for any component — variants, states, props, token usage, accessibility notes, do's and don'ts, and usage examples. Run it on components as you build or refactor them, and you end up with a style guide almost automatically."

Write the file `.claude/commands/generate-docs.md`:

```markdown
---
description: Generate structured markdown documentation for components — one or all at once
allowed-tools: Read, Write, Glob, AskUserQuestion
argument-hint: component-name-or-path | all
---

# Generate Docs: $1

## Mode Detection

- $1 is `all` → document every component (All Mode)
- Anything else → document one component (Single Mode)

## All Mode

Read CLAUDE.md for project context. Scan src/components/, components/, src/ui/ for component files. Present the full list via AskUserQuestion (multiSelect: true). For each confirmed component: read the source file and token file, write docs/components/[name].md using the Single Mode structure below. Create the docs/components/ directory if it doesn't exist. End with a summary listing X components documented and the path to each file.

## Single Mode

Read CLAUDE.md for project context. Find the component file for $1. Check: direct file path, src/components/[name].[ext], components/[name].[ext], src/ui/[name].[ext]. Read the component file and any associated CSS file. Read the token file.

Write docs/components/[component-name].md. Include:

### Overview
What this component is. When to use it vs. alternatives. 2-4 sentences.

### Variants
Table of all visual/functional variants with use case for each.

### States
Table: Default, Hover, Focus, Active, Disabled, plus any component-specific states (loading, error, success).

### Props / API
JS framework: Prop | Type | Default | Description table.
Vanilla HTML: Class/Attribute | Values | Description table.

### Design Tokens Used
Table of tokens the component references and the role of each.

### Accessibility
- Semantic element used and why
- ARIA attributes applied
- Keyboard interactions
- Screen reader behavior
- Any known gotchas

### Do's and Don'ts
3-5 bullets each. Focus on common misuse patterns specific to this component.

### Usage Examples
2-3 copy-paste ready code examples.

## After Writing

Tell the user the path to the generated file. Suggest running /generate-docs all to document additional components.
```

After writing the file, generate documentation for the component created in Step 4. Show the user what it looks like.

---

## Step 6 — CLAUDE.md and Summary

Say: "Last step. I'm creating a CLAUDE.md file that gives me persistent memory about your design system. Every future session, I'll read this before doing anything, so I always know your token names, your stack, your conventions, and what slash commands are available."

Write `CLAUDE.md` in the project root. Fill in the actual values discovered during the scan — don't use placeholders:

```markdown
# [Project Name] — Design System Context

> This file gives Claude persistent context about this project's design system.
> Update it any time conventions change. The more accurate it is, the better Claude performs.

## Tech Stack
- Framework: [detected framework]
- Language: [JS/TS]
- CSS approach: [CSS modules / SCSS / styled-components / vanilla CSS / Tailwind]
- Build tool: [Vite / Webpack / Next.js / etc.]

## Design Tokens
- **Token file**: `[path to token file]`
- **Naming convention**: [semantic / numeric scale / other]
- **Key tokens**:
  - Colors: [list 5-8 primary color tokens with values]
  - Spacing scale: [list spacing tokens]
  - Typography: [list primary font family, size tokens]
  - Border radius: [list radius tokens]

## Design Conventions
- **Primary color**: [description — e.g., "Blue (#2563EB) for interactive elements"]
- **Spacing**: [description — e.g., "8px base unit, scale: 4, 8, 12, 16, 24, 32, 48, 64"]
- **Typography**: [description — e.g., "Inter for UI, 14px base, 16px body, 24px headings"]
- **Border radius**: [description — e.g., "4px for inputs, 8px for cards, pill for tags"]

## Available Slash Commands
- `/audit-ui [path]` — Checks for token drift, accessibility issues, and component inconsistencies. Omit path to scan the whole project.
- `/generate-component [name]` — Generates a new component using the token system with all states and ARIA attributes.
- `/generate-docs [name]` — Documents a component with variants, states, props, token usage, and examples.

## Project Structure
[Brief description of where components, styles, and assets live]

## Accessibility Target
WCAG AA compliance. Visible focus on all interactive elements. All form inputs labeled.

## Notes
[Any specific patterns, gotchas, or conventions discovered during setup]
```

### Present the summary

After writing CLAUDE.md, give the user a complete summary:

> **Setup complete. Here's what was created:**
>
> | File | Purpose |
> |------|---------|
> | `[token file]` | Design tokens — all your colors, spacing, and typography in one place |
> | `.claude/commands/audit-ui.md` | `/audit-ui` — scan for inconsistencies and accessibility issues |
> | `.claude/commands/generate-component.md` | `/generate-component` — build new components from tokens |
> | `.claude/commands/generate-docs.md` | `/generate-docs` — document components as you build them |
> | `CLAUDE.md` | Project context — I'll read this every session automatically |
>
> **What to do next:**
> 1. Run `/audit-ui` to see a full report on your current codebase
> 2. Start migrating hardcoded values to tokens — the audit will show you where
> 3. Use `/generate-component` when you need new UI pieces
> 4. Use `/generate-docs` to document as you build
> 5. Edit `CLAUDE.md` any time to update conventions or add notes
>
> Your design system assistant is ready.
