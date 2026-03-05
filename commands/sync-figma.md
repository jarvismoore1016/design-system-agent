---
description: Sync tokens and components between your codebase and a connected Figma file — pull tokens, compare designs, generate code from Figma, or push components to Figma
allowed-tools: AskUserQuestion, Read, Write, Bash, Glob, Grep
---

# Sync Figma

## Before You Begin — Check Figma Connection

First, verify that Figma MCP tools are available. Check if any of these tools are accessible:
- `figma_get_variables`
- `figma_get_styles`
- `figma_check_design_parity`
- `figma_search_components`
- `figma_execute`

If none of these tools are available, print the following and stop:

```
Figma tools not connected.

To use /sync-figma, you need either:

Option A — Figma Console MCP (recommended):
  1. Install the MCP: https://github.com/GLips/Figma-Context-MCP
  2. Add to your Claude Code MCP settings
  3. Open Figma Desktop with your file
  4. Re-run /sync-figma

Option B — Figma Desktop Bridge plugin:
  1. Open Figma Desktop
  2. Right-click → Plugins → Development → Figma Desktop Bridge
  3. Re-run /sync-figma

Once connected, /sync-figma gives you four modes:
  - Pull tokens from Figma → sync design tokens to your codebase
  - Compare code to Figma → find spacing, color, and typography drift
  - Generate component from Figma → create code from a Figma component spec
  - Create component in Figma → push a code component into your Figma file
```

---

## Code Preferences

Before selecting a mode, read `CLAUDE.md` to check whether code generation preferences are already stored under a `## Code Preferences` heading.

If preferences are found, load them silently and proceed to Mode Selection.

If no preferences are stored, say:

> "Before we start, I want to make sure the code I generate matches exactly how you like to build things. A few quick questions — I'll save your answers to CLAUDE.md so you only need to do this once."

Then ask the following — each as a separate AskUserQuestion call:

**Styling approach**
- question: "How do you style your components?"
- header: "Styling"
- options:
  - CSS Modules (`.module.css` imported per component)
  - Styled Components / Emotion (CSS-in-JS with tagged templates)
  - Tailwind CSS (utility classes in markup)
  - SCSS / Sass (`.scss` files with variables)
  - Vanilla CSS (plain `.css` with class names)
  - Inline styles (style objects directly on elements)

**Language**
- question: "TypeScript or JavaScript?"
- header: "Language"
- options:
  - TypeScript (strict typing, interfaces, no `any`)
  - JavaScript (PropTypes or no type checking)

**Component pattern**
- question: "How do you prefer components to be structured?"
- header: "Pattern"
- options:
  - Standard functional (props in, JSX out — simple and direct)
  - Compound components (parent + sub-components like `<Menu.Item>`)
  - Headless / renderless (logic only, bring your own markup)
  - Simple — no abstraction (minimal, close to the metal)

**Exports**
- question: "How do you export components?"
- header: "Exports"
- options:
  - Named export only (`export function Button`)
  - Default export only (`export default Button`)
  - Both named and default

**Accessibility depth**
- question: "How thorough should accessibility attributes be?"
- header: "Accessibility"
- options:
  - Full WCAG AA (all ARIA attributes, roles, states, keyboard handling — production-ready)
  - Essential only (labels and roles, skip the verbose stuff)

**Tests**
- question: "Should I include a test file with generated components?"
- header: "Tests"
- options:
  - Yes — generate a test file alongside each component
  - No — skip tests

**File structure**
- question: "How should component files be organized?"
- header: "File structure"
- options:
  - Single file (component + styles in one file)
  - Separate files (component file + dedicated CSS/style file)
  - Barrel exports (component in its own folder with an `index` file)

After collecting all answers, write them to `CLAUDE.md` under a `## Code Preferences` heading:

```markdown
## Code Preferences

These preferences are used by /sync-figma and /generate-component when generating code.

- **Styling**: [answer]
- **Language**: [answer]
- **Pattern**: [answer]
- **Exports**: [answer]
- **Accessibility**: [answer]
- **Tests**: [answer]
- **File structure**: [answer]
```

Say: "Got it — saved to CLAUDE.md. You can update these any time by editing that file. Now, what would you like to do?"

---

## Mode Selection

Use AskUserQuestion:
- question: "What would you like to do?"
- header: "Figma sync mode"
- options:
  - Pull tokens from Figma (overwrite or merge token file)
  - Compare code to Figma (find design drift in a component)
  - Generate component from Figma (build code from a Figma component spec)
  - Create component in Figma (push a code component into Figma)

---

## Mode 1 — Pull Tokens from Figma

### Extract tokens from the connected Figma file

Use `figma_get_variables` to extract all design variables from the Figma file.
Use `figma_get_styles` to extract color, text, effect, and grid styles.

Organize the extracted data into categories:
- **Colors**: all color variables/styles with their values
- **Spacing**: numeric variables that look like spacing values (multiples of 4 or 8, named with "spacing", "gap", "padding", etc.)
- **Typography**: text styles — font family, size, weight, line height, letter spacing
- **Border radii**: numeric variables named with "radius", "rounded", "corner"
- **Shadows**: effect styles

### Map to project token format

Read the current token file (from `CLAUDE.md`) to understand the existing format and naming convention.

Map each extracted Figma token to the closest existing token name. For new tokens not in the existing file, derive a name following the same convention.

### Ask before writing

Present a summary of what will change:
```
Found 47 tokens in Figma. Here's what would change in tokens.css:

Changed (12):
  --color-primary: #1D4ED8 → #2563EB (Figma value)
  --spacing-md: 16px → 20px (Figma value)
  --radius-md: 6px → 8px (Figma value)
  ...

New (8):
  --color-surface: #F8FAFC (not in current token file)
  --color-border-subtle: #E2E8F0
  ...

Unchanged (27):
  --color-danger, --spacing-sm, ... (values match)
```

Use AskUserQuestion:
- question: "How would you like to handle the update?"
- header: "Token update"
- options:
  - Overwrite — replace token file with Figma values (creates backup first)
  - Merge — update existing tokens, add new ones, keep unchanged ones
  - Preview only — show me the diff without writing anything

If "Overwrite": copy current token file to `[tokenfile].backup.[timestamp]`, then write the new file.
If "Merge": update only the changed and new values in the existing token file.
If "Preview only": output the diff as a code block and stop.

### Report what changed

After writing, show:
```
Token sync complete.

Updated: 12 tokens
Added:   8 tokens
Kept:    27 tokens (matched Figma values exactly)

Token file: tokens.css
Backup: tokens.css.backup.[timestamp]
```

---

## Mode 2 — Compare Code to Figma

### Identify the target component

If `$1` is provided, use it as the component name or file path.
If not, ask: "Which component would you like to compare?"

Read the component source file. Extract:
- Background colors, text colors, border colors
- Padding, margin, gap values
- Font sizes, font weights, line heights
- Border radii
- Shadow values
- Width, height, min/max constraints

### Run design parity check

Use `figma_check_design_parity` with:
- The component node ID (search by name in Figma if needed using `figma_search_components`)
- The code-side values extracted above

### Report discrepancies

Present findings tagged by severity:

```
Design Parity Report: Button

BLOCKING (must fix for design accuracy):
  [BLOCKING] Background color
    Code: #1D4ED8 (--color-primary)
    Figma: #2563EB
    → Token value is stale. Update --color-primary or run /sync-figma pull.

  [BLOCKING] Border radius
    Code: 4px (--radius-sm)
    Figma: 8px
    → Mismatch. Update --radius-sm or use --radius-md.

COSMETIC (visual difference, lower priority):
  [COSMETIC] Letter spacing
    Code: not set (browser default)
    Figma: -0.01em
    → Minor. Add letter-spacing: var(--letter-spacing-tight).

MATCH:
  ✓ Padding: 16px/8px (matches --spacing-md/--spacing-sm)
  ✓ Font size: 14px (matches --font-size-sm)
  ✓ Font weight: 600 (matches --font-weight-semibold)

Parity score: 71% (5/7 properties match)
```

### Ask what to do about it

Use AskUserQuestion:
- question: "What would you like to do with the blocking issues?"
- header: "Fix approach"
- options:
  - Fix the code side (update component to match Figma)
  - Post a Figma comment (flag the design side for the designer)
  - Do both

If "Fix the code side": apply the fixes to the component file and show a diff summary.
If "Post a Figma comment": use `figma_post_comment` to add a comment on the component node:
  ```
  [Design Drift] Button component — 2 blocking discrepancies found:
  • background-color: code uses #1D4ED8, Figma shows #2563EB
  • border-radius: code uses 4px, Figma shows 8px
  Flagged by /sync-figma on [date].
  ```
If "Do both": apply code fixes and post the comment.

---

## Mode 3 — Generate Component from Figma

### Find the target component in Figma

If `$1` is provided, use it as the component name to search for.
If not, ask: "What's the name of the component in Figma?"

Use `figma_search_components` to find the component.

If multiple matches are found, use AskUserQuestion to let the user pick the right one.

### Validate the Figma component against design best practices

Before generating code, inspect the component structure using `figma_get_component` and flag any best practice issues. Reference the **Design Systems Best Practices** section at the end of this document for the full rule set.

Report any issues found:

```
Design Quality Check: Button

⚠ ISSUES FOUND (may affect code quality):
  [NAMING] Layer "Rectangle 3" should have a semantic name (e.g., "background", "border")
  [AUTO LAYOUT] Frame "Button/Primary" is not using Auto Layout — padding values may be unreliable
  [TOKENS] Fill color #2563EB is hardcoded — not bound to a variable. Code will use the hex directly.
  [VARIANTS] No "State" property found — hover, focus, and disabled states are missing

✓ PASSING:
  Component name follows ComponentName/Variant convention
  All text layers use a defined text style
  Description is set on the component

Code generation will proceed. Values from hardcoded layers will be marked /* NEEDS TOKEN */ in the output.
```

Use AskUserQuestion:
- question: "Proceed with code generation?"
- header: "Proceed?"
- options:
  - Yes — generate the code as-is
  - Fix issues first — I'll update the Figma component and re-run

If "Fix issues first": stop here. The user can address the Figma component and re-run.

### Get full component spec

Use `figma_get_component_for_development` with the found component node ID. This returns:
- Full layout spec (dimensions, padding, gap, alignment)
- Typography values (font family, size, weight, line height)
- Color values (fills, strokes, effects)
- A rendered image for reference

### Map to project tokens

For each value in the Figma spec, check if it matches a token from the project's token file:
- If it matches a token exactly → reference the token (e.g., `var(--color-primary)`)
- If it's close but not exact → reference the closest token and note the discrepancy
- If it doesn't match any token → use the hardcoded value and mark it `/* NEEDS TOKEN */`

### Generate the component using stored preferences

Apply all preferences from `CLAUDE.md` → `## Code Preferences` when generating:

**Styling approach**:
- CSS Modules → generate a `[Name].module.css` file alongside the component; reference classes via `styles.className`
- Styled Components / Emotion → define styled elements at the top of the file using tagged template literals
- Tailwind → apply utility classes directly in JSX; no separate style file
- SCSS → generate a `[Name].scss` file; import and reference class names
- Vanilla CSS → generate a `[Name].css` file with BEM-style class names
- Inline styles → use `style={{}}` objects on elements; define style objects as constants at the top

**Language**:
- TypeScript → define a `Props` interface with strict types; no `any`; use `FC<Props>` or explicit return type
- JavaScript → use PropTypes for validation; `defaultProps` for defaults

**Component pattern**:
- Standard functional → `function ComponentName(props) { return <JSX /> }`
- Compound → export a parent component and attach sub-components as properties (`ComponentName.Item = ...`)
- Headless → export a hook containing logic; let the caller provide markup
- Simple → minimal code, no abstraction, direct and readable

**Exports**:
- Named → `export function ComponentName`
- Default → `export default ComponentName`
- Both → named export in the file body + `export default` at the end

**Accessibility depth**:
- Full WCAG AA → include `role`, `aria-label`, `aria-expanded`, `aria-controls`, `aria-describedby`, `aria-disabled`, keyboard handlers (`onKeyDown`, `onKeyUp`), focus management; every interactive element fully annotated
- Essential only → include `role` and `aria-label` where semantics aren't sufficient; skip verbose state tracking

**Tests** (if enabled):
- Generate a `[Name].test.tsx` (or `.test.jsx`) file alongside the component
- Include: render test, variant tests, interaction tests (click/keyboard), accessibility test using `@testing-library/jest-dom`

**File structure**:
- Single file → component and styles in one file
- Separate files → component file + dedicated style file in the same directory
- Barrel exports → create a folder `[ComponentName]/` containing `index.ts`, component file, and style file

Generate a complete component:
- Include all variants shown in the Figma component (match the component properties)
- Include all states visible in the Figma file (look for variant properties named "State", "Status", etc.)
- Apply accessibility depth per preferences
- All token-matched values use token references; hardcoded values are marked `/* NEEDS TOKEN */`

Save to the standard component location for the project (or inside `[ComponentName]/` for barrel structure).

### Report what was generated

```
Component generated: Button

Files:
  src/components/Button/index.ts
  src/components/Button/Button.tsx
  src/components/Button/Button.module.css
  src/components/Button/Button.test.tsx    ← tests included per preference

Preferences applied:
  Styling:       CSS Modules
  Language:      TypeScript
  Pattern:       Standard functional
  Accessibility: Full WCAG AA
  Tests:         Yes

Tokens matched: 8/10 values
  ✓ background → var(--color-primary)
  ✓ padding → var(--spacing-sm) / var(--spacing-md)
  ✓ border-radius → var(--radius-md)
  ⚠ letter-spacing: -0.01em (no token — marked /* NEEDS TOKEN */)
  ⚠ box-shadow: 0 1px 3px ... (closest match: --shadow-sm)

Variants: primary, secondary, ghost, destructive
States: default, hover, focus, active, disabled

Design issues carried over from Figma:
  ⚠ Layer "Rectangle 3" → named "background" in code (assumed)
  ⚠ No "Loading" state in Figma — not generated (add manually if needed)
```

---

## Mode 4 — Create Component in Figma

### Read the source component

If `$1` is provided, treat it as a file path and read that file.
If not, ask: "What's the path to the component file you want to push to Figma?"

Read the component file. Extract:
- Component name and variants
- All visual states
- Color values (resolve to hex by looking up token values in the token file)
- Spacing, font, and radius values (resolve token references to their actual values)

### Find or create the Components section

Use `figma_execute` to:
1. Check the current page for a Section named "Components" or "Design System"
2. If found, place the new component inside it
3. If not found, create a new Section named "Components" and place inside it

### Build the component applying design systems best practices

Use `figma_execute` to create the component. Apply every rule from the **Design Systems Best Practices** section below. This is not optional — every component created by this command should be a correct, production-quality Figma component.

**Layer structure**:
- Name every layer semantically. No default Figma names ("Rectangle 47", "Frame 12", "Group 3").
- Use descriptive names that map to code: `background`, `label`, `icon`, `border`, `container`, `content`
- Group related layers. Use frames over groups where layout control is needed.

**Auto Layout**:
- Apply Auto Layout to every frame that contains children. No absolute positioning unless it's truly an overlay (e.g., a badge on an avatar).
- Set explicit horizontal and vertical padding using resolved token values.
- Set gap using the spacing scale.
- Set alignment (primary axis + counter axis) to match the component's layout intent.
- Set "Hug contents" on the component frame so it sizes to its content by default.

**Variants and states**:
- Create a variant for each visual variant in the code (primary, secondary, ghost, destructive, etc.) using a `Variant` property.
- Create a `State` property with values: Default, Hover, Pressed, Focused, Disabled.
- If the component has a loading state in code, add `Loading` to the State property.
- Name each variant frame: `[Variant=Primary, State=Default]` — Figma's standard format.
- Ensure every Variant × State combination exists as a frame. If a combination isn't visually distinct, it's still needed (duplicate the closest one).

**Component properties**:
- Add a **Text** property for any visible label or content that changes between uses. Bind it to the text layer.
- Add a **Boolean** property (`Show Icon`, `Show Badge`, etc.) for any element that can appear or be hidden. Bind it to the layer's visibility.
- Add an **Instance Swap** property for any slot where a different component (like an icon) can be swapped in.
- Do not expose properties for things that shouldn't be changed by component consumers.

**Token binding**:
- Apply styles or variables to every fill, stroke, text style, and effect — no hardcoded hex values.
- If a value doesn't have a matching token, use the closest available token and add a Figma comment noting the discrepancy.
- Typography layers must use a defined text style, not manually set font properties.

**Component naming**:
- Name the component set: `ComponentName` (e.g., `Button`, `Input`, `Badge`)
- Name individual variants using the property format: `[Variant=Primary, State=Default]`
- For nested components (sub-components used inside this one), prefix with the parent: `Button/Icon`, `Button/Label`

**Constraints and resizing**:
- Set horizontal resizing to "Fill container" for components that should stretch (inputs, cards, nav items).
- Set horizontal resizing to "Hug contents" for components with a natural width (buttons, badges, avatars).
- Set the component's constraints relative to its parent so it behaves correctly when the parent resizes.

**Description**:
- Set a description on the component set explaining: what it is, when to use it, when not to use it.
- Set descriptions on each component property explaining what it controls.

**Accessibility annotation**:
- Add a text note layer named `_a11y` (hidden, not visible in exports) or use a Figma annotation kit to document:
  - The semantic role (button, input, checkbox, etc.)
  - Required ARIA attributes
  - Keyboard interaction
  This layer is a design handoff artifact — it doesn't appear in production.

### Report what was created

```
Component created in Figma: Button

Location: Components section on current page
Node ID: 123:456

Structure:
  Component set: Button
  Variants: Primary, Secondary, Ghost, Destructive (4)
  States: Default, Hover, Pressed, Focused, Disabled (5)
  Frames created: 20 (4 variants × 5 states)

Properties exposed:
  Label (Text) → bound to label layer
  Show Icon (Boolean) → bound to icon layer visibility
  Icon (Instance Swap) → bound to icon slot

Best practices applied:
  ✓ All layers semantically named
  ✓ Auto Layout on all frames
  ✓ Token variables bound to all fills and text styles
  ✓ Component description set
  ✓ Accessibility annotation layer added

Note: Hover and focus states show the visual treatment but are not interactive in Figma.
For prototype interactions, wire the Default state to the Hover state using Figma's interaction panel.
```

---

## Design Systems Best Practices Reference

This section defines the standards applied in Mode 3 (validation) and Mode 4 (creation). These are the rules of a production-quality Figma design system.

---

### Component Architecture

**Single responsibility**: Each component does one thing. A `Button` renders a button. It does not also handle navigation logic, data fetching, or layout decisions.

**Atomic structure**: Build components from smaller primitives where it makes sense — an `IconButton` is a `Button` with an `Icon` slot, not a new standalone component.

**Explicit API**: Every aspect of a component that a consumer can change must be a component property. Anything that shouldn't change should be locked. Consumers of the component should never need to break into its internals.

---

### Naming Conventions

**Component names**: PascalCase, noun-first. `Button`, `InputField`, `NavigationBar`, `UserAvatar`. Not `BlueButton`, `ClickableText`, or `Component1`.

**Variant property**: Named `Variant` or `Type`. Values are PascalCase: `Primary`, `Secondary`, `Ghost`, `Destructive`.

**State property**: Named `State`. Values: `Default`, `Hover`, `Pressed`, `Focused`, `Disabled`, `Loading`, `Error`, `Success`.

**Layer names**: lowercase, hyphenated. `background`, `label`, `icon-left`, `icon-right`, `border`, `container`, `prefix`, `suffix`. Never `Frame 1`, `Rectangle`, `Group`, `Ellipse 47`.

**Nested component names**: `[Parent]/[Child]`. A label inside a button component: `Button/Label`. An icon slot: `Button/Icon`.

---

### Spacing and Layout

**4pt grid**: All spacing values (padding, gap, size, offset) must be multiples of 4. Preferred values: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64.

**Auto Layout everywhere**: Any frame with children should use Auto Layout unless it's a true overlay (tooltips, floating badges). Frames using absolute positioning for layout purposes are a red flag.

**No magic numbers**: Every numeric value (padding, gap, border radius, size) must come from the token/variable system. An 18px padding where the scale has 16px and 20px is a sign of pixel pushing, not systematic design.

**Consistent touch targets**: Interactive components (anything that becomes a button or input in code) must have a minimum frame size of 44×44pt for iOS targets or 48×48dp for Android/Material targets.

---

### Token Usage

**All fills must be bound**: Solid fills → color variable. No hex values directly on a layer.

**All text must use a text style**: Font family, size, weight, line height must come from a defined text style, not manual overrides.

**All effects (shadows) must use an effect style**: No manually entered shadow values.

**Semantic tokens over primitive tokens**: Use `color/interactive/primary` or `--color-primary`, not `color/blue/600` or `#2563EB`. Semantic tokens survive rebrand; primitive tokens don't.

---

### Variant and State Structure

**Every variant needs every state**: If a component has 4 variants and 5 states, there are 20 frames. Missing combinations cause inconsistent behavior in prototypes and make code generation ambiguous.

**State is always the last property**: In Figma's variant panel, properties are listed in the order they appear. Put `State` last so it toggles most easily in the properties panel.

**Disabled is a state, not a variant**: `Disabled` belongs in the `State` property, not its own separate `Variant`. Every variant should have a disabled state.

**Interactive states must be visually distinct**: Hover, focused, and pressed states must look different from Default — even if subtly. A state that looks identical to Default is a missing state, not a valid state.

---

### Component Properties

**Text properties for all editable content**: If a text layer will say different things in different uses, it must be a Text property. Consumers should never need to detach a component to change a label.

**Boolean properties for optional elements**: If a sub-element (icon, badge, suffix, helper text) is sometimes visible and sometimes not, it must be a Boolean property. Visibility should not require a new variant.

**Instance Swap for icon slots**: Icon slots should be Instance Swap properties pointing to an icon library component, not hardcoded icon instances. This lets consumers swap icons without breaking anything.

**Avoid over-exposing properties**: Only expose what genuinely varies between uses. If a color or size never changes between instances, don't make it a property — lock it inside the component.

---

### Documentation

**Component description (required)**: Every component set must have a Figma description covering: what it is, when to use it, when NOT to use it, and which component to use instead in those cases.

**Property descriptions (recommended)**: Each component property should have a description explaining what it controls and any non-obvious constraints.

**Changelog annotation (optional)**: For mature design systems, maintain a text layer named `_changelog` inside the component with version history.

---

### Accessibility

**Every interactive component needs an accessibility annotation** covering:
- Semantic role: what this becomes in HTML/code (button, link, input, checkbox, etc.)
- Accessible name: where the name comes from (visible label, aria-label, aria-labelledby)
- State communication: how disabled, expanded, selected, checked states are communicated to screen readers
- Keyboard interaction: Tab to reach, Enter/Space to activate, Escape to dismiss, arrow keys for groups

Use Figma's built-in accessibility annotation kit or a dedicated annotation component. Annotations should be in a separate `Annotations` page or a hidden layer group that does not export.

**Color alone cannot communicate meaning**: Any status (error, success, warning, disabled) communicated by color must also use a text label, icon, or pattern. This is a WCAG 1.4.1 requirement.

**Text contrast**: All text in components must meet WCAG AA: 4.5:1 for normal text, 3:1 for large text (18pt+ or 14pt bold+). Run a contrast check before finalizing token values.
