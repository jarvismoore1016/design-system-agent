---
description: Audit the UI for visual consistency, token drift, and accessibility issues
allowed-tools: Read, Glob, Grep, Bash
---

# UI Audit

Scan the project (or the path provided as `$1` if given) for design system issues. Report findings grouped by category with file and line references. Be specific — name the file, the value, and what it should be instead.

## Before You Begin

Read `CLAUDE.md` for project context:
- Token file path and token naming convention
- Tech stack and file conventions
- Known patterns and gotchas

Read the token file to build a reference list of approved values. Any value that matches a token value but appears hardcoded is a drift finding.

## What to Check

### 1. Token Drift

Find hardcoded values in CSS, SCSS, JS, and HTML that duplicate or approximate values from the token file.

Check:
- Color values (`#xxxxxx`, `rgb()`, `hsl()`, `rgba()`) that match or closely approximate a token color
- Spacing values (`margin`, `padding`, `gap`) that match a token spacing value
- Font sizes that match a token font size
- Border radius values that match a token radius
- Shadow values that match a token shadow

For each finding:
```
[TOKEN DRIFT] path/to/file.css:42
  Found: color: #2563EB
  Token: --color-primary
  Fix: Replace with var(--color-primary)
```

### 2. Accessibility

Check for:
- `<input>`, `<select>`, `<textarea>` elements without an associated `<label>` element or `aria-label` / `aria-labelledby` attribute
- `<img>` elements with missing or empty `alt` attributes (decorative images should have `alt=""`, not no attribute)
- `<button>` or `<a>` elements with non-descriptive visible text: "click here", "read more", "here", "link", "button"
- Icon-only interactive elements (buttons, links) without `aria-label` or visually-hidden text
- `outline: none` or `outline: 0` without a replacement `:focus-visible` style
- Missing `role` attributes on custom interactive elements (e.g., a `<div>` with a click handler)
- Missing `aria-expanded`, `aria-controls`, or `aria-haspopup` on disclosure and menu components

For each finding:
```
[ACCESSIBILITY] path/to/file.html:87
  Issue: Input has no associated label
  Element: <input type="email" placeholder="Email">
  Fix: Add <label for="email"> or aria-label="Email address"
```

### 3. Component Inconsistency

Look for:
- The same UI pattern with different implementations across files — e.g., three distinct card structures with different class names (`.card`, `.shoe-card`, `.collection-item`) doing the same job
- Similar class naming patterns suggesting parallel component implementations (`.btn`, `.button`, `.cta-btn`)
- Inline `style=""` attributes that belong in a class or token

For each finding:
```
[INCONSISTENCY] Multiple files
  Issue: Button implemented 3 different ways
  Files: index.html (.hero-btn), collection.html (.filter-btn), detail.html (.back-button)
  Fix: Consolidate into a single Button component with variants
```

### 4. Contrast (Flag for Review)

Flag combinations that likely fail WCAG AA:
- Light gray text on white background (e.g., `#999999` on `#ffffff` ≈ 2.85:1 — fails)
- Muted text colors on light backgrounds
- White text on medium-saturated backgrounds

Note these as needing manual verification — don't claim they fail without checking the actual values.

```
[CONTRAST — CHECK] path/to/file.css:24
  Text: color: #999999 (on white background)
  Estimated ratio: ~2.9:1
  Required: 4.5:1 (normal text) or 3:1 (large text, 18px+ or 14px bold)
  Fix: Use #757575 or darker for AA compliance
```

## Output Format

Structure findings by category with a count per category. Show the most critical issues first within each section.

After all findings, write a summary table:

| Category | Critical | Warning | Suggestion |
|----------|----------|---------|------------|
| Token drift | — | — | — |
| Accessibility | — | — | — |
| Inconsistency | — | — | — |
| Contrast | — | — | — |
| **Total** | | | |

End with **3-5 prioritized next steps** based on the highest-impact findings — not a list of every fix, just the most valuable starting points.

## Scope

If `$1` is provided, limit the scan to that path. If not, scan the entire project, excluding `node_modules`, `.git`, `dist`, `build`, and `.next`.
