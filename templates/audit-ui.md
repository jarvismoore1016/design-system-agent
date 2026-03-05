---
description: Audit the UI for visual consistency, token drift, and accessibility issues
allowed-tools: Read, Glob, Grep, Bash, AskUserQuestion, Write
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

### 2. Accessibility (WCAG 2.1 AA)

Check for the following. Tag each finding with its WCAG criterion.

**Color contrast** `[WCAG 1.4.3]` `[WCAG 1.4.11]`:
- Normal text (under 18px regular or 14px bold): must meet 4.5:1 contrast ratio against its background
- Large text (18px+ regular or 14px+ bold): must meet 3:1
- UI components and graphical objects (borders, icons, chart lines): must meet 3:1 against adjacent colors
- Flag any text or UI element where the computed contrast is likely to fail — show the estimated ratio

**Keyboard navigability** `[WCAG 2.1.1]` `[WCAG 2.4.3]`:
- All interactive elements (buttons, links, inputs, selects, checkboxes, custom controls) must be reachable by Tab
- Focus order must follow the visual/logical reading order — flag if a `tabindex` value creates an illogical sequence
- Flag `<div>`, `<span>`, or `<li>` elements with click handlers but no `tabindex="0"` and no keyboard event handler

**Focus visible** `[WCAG 2.4.7]`:
- Every interactive element must show a visible focus indicator
- `outline: none` or `outline: 0` without a replacement `:focus-visible` style is a violation
- Focus indicator must have at least 3:1 contrast against adjacent colors

**No keyboard traps** `[WCAG 2.1.2]`:
- Modals, drawers, and dialogs must trap focus while open AND provide a keyboard-accessible close mechanism (Escape key or a focusable close button)
- Custom dropdown menus must not leave keyboard users unable to exit

**Text alternatives** `[WCAG 1.1.1]`:
- `<img>` elements: must have `alt` attribute — decorative images use `alt=""`, informative images describe the content
- Icon-only interactive elements (buttons, links) must have `aria-label` or visually-hidden text
- SVG icons used as content must have `<title>` or `aria-label`

**Form labels** `[WCAG 1.3.1]`:
- Every `<input>`, `<select>`, and `<textarea>` must have a programmatic label: `<label for="...">`, `aria-label`, or `aria-labelledby`
- `placeholder` text alone is not a label — it disappears on input and has insufficient contrast

**Error identification** `[WCAG 3.3.1]`:
- Form validation errors must be described in text — not only through color or icon
- Error messages must be associated with their input via `aria-describedby` or `aria-errormessage`

**Additional checks**:
- `<button>` or `<a>` with non-descriptive text: "click here", "read more", "here", "link", "button" `[WCAG 2.4.6]`
- Missing `role` on custom interactive elements (e.g., `<div onClick={...}>`) `[WCAG 4.1.2]`
- Missing `aria-expanded`, `aria-controls`, or `aria-haspopup` on disclosure and menu components `[WCAG 4.1.2]`

For each finding, include the WCAG criterion tag:
```
[ACCESSIBILITY] path/to/file.html:87 [WCAG 1.3.1]
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

---

## Audit Log

After presenting the summary table and next steps, write the audit to the persistent log **before** entering interactive fix mode — this ensures findings are recorded even if the user skips fixes entirely.

### 1 — Capture timestamp and audit number

Run these two Bash commands:

```bash
date "+%Y-%m-%d %H:%M"
```
```bash
date "+%Y-%m-%d-%H-%M"
```

The first gives the human-readable date for display (`YYYY-MM-DD HH:MM`). The second gives the filename-safe timestamp (`YYYY-MM-DD-HH-MM`). Store both for use in steps below.

To determine the audit number (N): count `.md` files in `design-system-audits/` that are not named `SUMMARY.md`. If the directory does not exist yet, N = 1.

```bash
ls design-system-audits/*.md 2>/dev/null | grep -v SUMMARY.md | wc -l
```

Add 1 to the result.

### 2 — Create the audit directory

```bash
mkdir -p design-system-audits
```

### 3 — Write the individual audit report file

Write the complete audit to `design-system-audits/{filename-timestamp}.md`. Use this exact structure:

```markdown
# Audit #{N} — {YYYY-MM-DD HH:MM}

**Project**: {project name — use directory name if CLAUDE.md has no project name}
**Scanned**: {N} files, {N} lines
**Scope**: {the path passed as $1, or "Full project" if no argument}
**Previous audit**: {link to the most recent prior audit file, e.g. [Audit #2](2025-02-20-10-14.md), or "None — first audit"}

---

## Findings

{Paste all TOKEN DRIFT, ACCESSIBILITY, INCONSISTENCY, and CONTRAST finding blocks exactly as shown to the user — do not summarize or truncate}

---

## Summary

| Category | Critical | Warning | Suggestion |
|----------|----------|---------|------------|
| Token drift | {N} | {N} | {N} |
| Accessibility | {N} | {N} | {N} |
| Inconsistency | {N} | {N} | {N} |
| Contrast | {N} | {N} | {N} |
| **Total** | **{N}** | **{N}** | **{N}** |

---

## Prioritized Next Steps

{The 3-5 prioritized next steps exactly as shown to the user}

---

## Fix Session

> *No fix session run for this audit.*
```

The Fix Session section starts as a placeholder. It will be updated in place after interactive fix mode completes.

### 4 — Update SUMMARY.md

Read `design-system-audits/SUMMARY.md`. If it does not exist, create it with this header:

```markdown
# Design System Audit Log

Tracks design system health over time. Most recent audit first.

| # | Date | Files | Token Drift | Accessibility | Inconsistency | Contrast | Total | Fixes Applied | Report |
|---|------|-------|-------------|---------------|---------------|----------|-------|---------------|--------|
```

Then insert a new row **at the top of the table** (below the header row), so the most recent audit always appears first:

```
| {N} | {YYYY-MM-DD HH:MM} | {N} files | {C}C {W}W {S}S | {C}C {W}W {S}S | {C}C {W}W {S}S | {C}C {W}W {S}S | {total} | — | [{filename-timestamp}.md]({filename-timestamp}.md) |
```

Use C = Critical count, W = Warning count, S = Suggestion count from the summary table. Leave "Fixes Applied" as `—` for now; it will be updated after the fix session.

---

## Interactive Fix Mode

After presenting the summary table and next steps, enter interactive fix mode.

### Step 1 — Ask which categories to fix

Use AskUserQuestion with `multiSelect: true`:
- question: "Which categories of findings would you like me to fix now?"
- header: "Fix categories"
- options:
  - Token drift (replace hardcoded values with token references)
  - Accessibility (apply ARIA fixes, labels, focus styles)
  - Inconsistency (consolidate duplicate component patterns)
  - Contrast (adjust colors to meet WCAG AA ratios)

If the user selects none, end here. If they close or skip, end here.

### Step 2 — Fix findings one at a time

For each selected category, go through every finding in that category one by one.

Before each fix, show the finding in full and use AskUserQuestion:
- question: "Fix this finding?\n\n[CATEGORY] path/to/file:line\n  Issue: [issue]\n  Fix: [proposed fix]"
- header: "Apply fix?"
- options:
  - Yes — apply this fix
  - Skip — leave it for now
  - Stop — stop fixing this category

If **Yes**: apply the fix to the file. Then show a brief diff summary:
```
Fixed: path/to/file.css:42
  - color: #2563EB
  + color: var(--color-primary)
```

If **Skip**: move to the next finding in the category.

If **Stop**: stop processing the current category. If more categories are selected, move to the next one.

### Step 3 — Category summary

After finishing all findings in a category (or when the user stops), show:
```
Token drift: 4 fixes applied, 2 skipped
```

Then move to the next selected category if any.

### Step 4 — Final summary and log update

After all selected categories are processed, show:
```
Fix session complete.
  Token drift:    4 applied, 2 skipped
  Accessibility:  6 applied, 1 skipped
  Inconsistency:  0 applied, 3 skipped
  Contrast:       1 applied, 0 skipped
```

Then update the audit log to record the fix session results.

**4a — Update the Fix Session section in the individual audit file**

Find the audit file written in the Audit Log step (`design-system-audits/{filename-timestamp}.md`). Replace the Fix Session placeholder:

```markdown
## Fix Session

> *No fix session run for this audit.*
```

With the actual fix session record:

```markdown
## Fix Session

**Categories selected**: Token drift, Accessibility

| Category | Applied | Skipped |
|----------|---------|---------|
| Token drift | 4 | 2 |
| Accessibility | 6 | 1 |
| Inconsistency | 0 | 3 |
| Contrast | 1 | 0 |
| **Total** | **11** | **6** |

### Changes Made

{List each fix that was applied, one per line:}
- `path/to/file.css:42` — Replaced `color: #2563EB` with `var(--color-primary)`
- `path/to/file.html:87` — Added `aria-label="Email address"` to email input
```

If the user selected no categories (or skipped all fixes), update the placeholder to:

```markdown
## Fix Session

> *No fixes applied — user skipped fix mode.*
```

**4b — Update the Fixes Applied column in SUMMARY.md**

Read `design-system-audits/SUMMARY.md` and find the row for this audit (matched by the filename-timestamp link). Update the `Fixes Applied` cell from `—` to the total number of fixes applied (e.g., `11`).

---

## Scope

If `$1` is provided, limit the scan to that path. If not, scan the entire project, excluding `node_modules`, `.git`, `dist`, `build`, and `.next`.
