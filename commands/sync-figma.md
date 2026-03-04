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

## Mode Selection

Read `CLAUDE.md` first for project context — stack, token file path, and naming conventions.

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
Backup: tokens.css.backup.2024-01-15T14-32-00
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

### Get full component spec

Use `figma_get_component_for_development` with the found component node ID. This returns:
- Full layout spec (dimensions, padding, gap, alignment)
- Typography values (font family, size, weight, line height)
- Color values (fills, strokes, effects)
- A rendered image for reference

### Map to project tokens

For each value in the Figma spec, check if it matches a token from the project's token file:
- If it matches a token exactly → reference the token (e.g., `var(--color-primary)`)
- If it's close but not exact → reference the closest token and add a comment noting the discrepancy
- If it doesn't match any token → use the hardcoded value and note it with `/* MISSING TOKEN */`

### Generate the component

Generate a complete component in the project's stack format (from `CLAUDE.md`):
- Use token references for all matched values
- Include all variants shown in the Figma component (match the component properties)
- Include all states visible in the Figma file (look for variant properties named "State", "Status", etc.)
- Full accessibility attributes appropriate for the stack

Save the file to the standard component location for the project.

### Report what was generated

```
Component generated: Button

File: src/components/Button.tsx

Tokens matched: 8/10 values
  ✓ background → var(--color-primary)
  ✓ padding → var(--spacing-sm) / var(--spacing-md)
  ✓ border-radius → var(--radius-md)
  ⚠ letter-spacing: -0.01em (no token found — value hardcoded, marked /* MISSING TOKEN */)
  ⚠ box-shadow: 0 1px 3px ... (no exact token match — using --shadow-sm, closest match)

Variants: primary, secondary, ghost, destructive
States: default, hover, focus, active, disabled

Review the /* MISSING TOKEN */ comments to decide whether to add new tokens or adjust the design.
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

### Build the component in Figma

Use `figma_execute` to create the component structure:
- Create a Frame for each variant/state combination
- Apply the resolved fills, strokes, and radii
- Add text layers with correct font properties
- Set up Auto Layout matching the component's layout direction, gap, and padding
- Name all layers to match the code component structure (e.g., "Button/Primary/Default")

Group all frames into a component set using `figma.combineAsVariants()`.

Apply naming: component set = component name, each variant = "[Variant=Primary, State=Default]" format.

### Report what was created

```
Component created in Figma: Button

Location: Components section on current page
Variants created: 8 (4 variants × 2 states: default + disabled)
Node ID: 123:456 (for future reference)

Note: Hover and focus states were not created in Figma since those are interaction states.
If you need them for design review, create an "Interactive" page with hover frames.
```
