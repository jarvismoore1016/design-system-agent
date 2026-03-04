# UI Audit Report

**Project**: Storefront UI
**Stack**: Vanilla HTML + CSS
**Scanned**: 6 files (4 HTML pages, 1 shared CSS, 847 lines total)
**Date**: Run `/audit-ui` in Claude Code to generate a live version for your project

---

## Token Drift — 14 findings

> Colors, spacing, and sizes hardcoded instead of using token values

```
[TOKEN DRIFT] pages/home.html:134
  Found: color: #2563EB
  Token: --color-primary
  Fix: Replace with var(--color-primary)

[TOKEN DRIFT] pages/home.html:198
  Found: background-color: #F5F5F5
  Token: --color-surface
  Fix: Replace with var(--color-surface)

[TOKEN DRIFT] pages/catalog.html:89
  Found: color: #2980B9
  Note: Approximates --color-primary but is a different value (no exact match)
  Fix: Decide — use var(--color-primary) or add a new token --color-secondary

[TOKEN DRIFT] pages/catalog.html:214
  Found: padding: 20px
  Token: --spacing-md (20px)
  Fix: Replace with var(--spacing-md)

[TOKEN DRIFT] pages/detail.html:67
  Found: font-size: 14px
  Token: --font-size-sm (14px)
  Fix: Replace with var(--font-size-sm)

[TOKEN DRIFT] pages/detail.html:312
  Found: border-radius: 8px
  Token: --radius-md (8px)
  Fix: Replace with var(--radius-md)

[TOKEN DRIFT] pages/checkout.html:44
  Found: box-shadow: 0 2px 8px rgba(0,0,0,0.1)
  Token: --shadow-sm
  Fix: Replace with var(--shadow-sm)

... and 7 more (run /audit-ui for full list)
```

---

## Accessibility — 9 findings

> Issues that affect users who rely on screen readers, keyboard navigation, or contrast

```
[ACCESSIBILITY — CRITICAL] pages/checkout.html:67
  Issue: Input has no label
  Element: <input type="email" placeholder="Email address">
  Fix: Add <label for="email-input">Email address</label> and id="email-input" to the input,
       or add aria-label="Email address" to the input element

[ACCESSIBILITY — CRITICAL] pages/checkout.html:78
  Issue: Input has no label
  Element: <input type="tel" placeholder="Phone number">
  Fix: Same as above — use <label> element or aria-label

[ACCESSIBILITY — CRITICAL] pages/checkout.html:91
  Issue: Input has no label
  Element: <select name="country"><option>Select country</option>...</select>
  Fix: Add <label for="country-select">Country</label>

[ACCESSIBILITY — WARNING] shared.css:8
  Issue: Focus outline removed globally
  Code: * { outline: none; }
  Fix: Remove this rule. Instead, style :focus-visible on interactive elements
       to provide a visible, on-brand focus indicator

[ACCESSIBILITY — WARNING] pages/catalog.html:203
  Issue: Icon-only button has no accessible label
  Element: <button class="sort-btn"><svg>...</svg></button>
  Fix: Add aria-label="Sort results" to the button

[ACCESSIBILITY — WARNING] pages/home.html:88
  Issue: Non-descriptive link text
  Element: <a href="/catalog">Read more</a>
  Fix: Use descriptive text like "Browse the full catalog" or add
       aria-label="Read more about our product catalog"

[ACCESSIBILITY — WARNING] pages/home.html:156
  Issue: Image missing alt text
  Element: <img src="/hero.jpg" class="hero-image">
  Fix: Add descriptive alt="[describe the image content]"
       or alt="" if purely decorative

[ACCESSIBILITY — WARNING] pages/detail.html:189
  Issue: Tab order is non-logical
  Detail: "Back" button appears after the main content in DOM order but is
          visually first on the page
  Fix: Reorder DOM to match visual order, or use tabindex carefully

[ACCESSIBILITY — SUGGESTION] pages/catalog.html:234
  Issue: Color-coded status (green/red dots) with no text alternative
  Fix: Add a text label or aria-label to each status indicator
```

---

## Component Inconsistency — 5 findings

> The same UI pattern implemented multiple ways across the codebase

```
[INCONSISTENCY] Multiple files
  Issue: Navigation implemented 4 different ways
  Files:
    - pages/home.html: <nav class="main-nav">
    - pages/catalog.html: <header class="site-header"><nav>
    - pages/detail.html: <div class="navbar">
    - pages/checkout.html: <nav class="top-nav">
  Impact: Each has different HTML structure, class names, and active state styles
  Fix: Extract a single navigation component or shared nav partial

[INCONSISTENCY] Multiple files
  Issue: Button appears in 3 different implementations
  Files:
    - pages/home.html: .hero-btn (48px height, 24px border-radius, #2563EB)
    - pages/catalog.html: .filter-btn (32px height, 4px border-radius, #2980B9)
    - pages/detail.html: .back-button (36px height, 6px border-radius, #1E90FF)
  Impact: No consistent button component — each has different height, radius, and color
  Fix: Create a single Button component with size and variant props

[INCONSISTENCY] Multiple files
  Issue: Card component implemented differently per page
  Files:
    - pages/home.html: .product-card (padding: 16px, radius: 8px, shadow: 0 2px 8px...)
    - pages/catalog.html: .catalog-item (padding: 20px, radius: 12px, shadow: 0 1px 4px...)
  Impact: Same UI pattern, different visual result — not a deliberate variant, just drift
  Fix: Create a Card component with a single implementation

[INCONSISTENCY] pages/checkout.html
  Issue: Form inputs styled inconsistently within a single page
  Detail: Text inputs use border-radius: 4px, selects use border-radius: 0px,
          checkboxes have no custom styling
  Fix: Create shared input styles applied consistently across all form element types

[INCONSISTENCY] pages/home.html, pages/detail.html
  Issue: Section headings use different sizes for the same visual level
  Detail: home.html section heading: 26px | detail.html section heading: 24px and 28px
  Fix: Establish a heading scale in tokens and apply consistently
```

---

## Contrast — 3 items flagged for review

> Manual verification recommended — these combinations may not meet WCAG AA

```
[CONTRAST — CHECK] shared.css:34
  Text: color: #999999 on white (#FFFFFF) background
  Estimated ratio: ~2.85:1
  Required: 4.5:1 for normal text
  Status: FAILS — below threshold
  Fix: Use #767676 or darker (minimum for AA on white)

[CONTRAST — CHECK] pages/home.html:144
  Text: color: #AAAAAA on #F5F5F5 background
  Estimated ratio: ~2.32:1
  Status: FAILS — well below threshold
  Fix: Darken the text color significantly, or use on a darker background

[CONTRAST — CHECK] pages/catalog.html:89
  Text: color: #CCCCCC (placeholder text) on white input
  Estimated ratio: ~1.6:1
  Note: Placeholder text is not required to meet contrast ratios by WCAG,
        but this is extremely low and will affect readability for many users
  Fix: Use #767676 or darker for placeholder text
```

---

## Summary

| Category | Critical | Warning | Suggestion |
|----------|----------|---------|------------|
| Token drift | 0 | 14 | 0 |
| Accessibility | 3 | 5 | 1 |
| Inconsistency | 0 | 5 | 0 |
| Contrast | 0 | 2 | 1 |
| **Total** | **3** | **26** | **2** |

---

## Prioritized Next Steps

1. **Fix the 3 critical accessibility issues first** — unlabeled form inputs in checkout.html block users who rely on screen readers. This is a legal risk in many jurisdictions. Takes 10 minutes to fix.

2. **Remove `outline: none` from shared.css** — this single rule removes all keyboard focus indicators across the entire site. Add `:focus-visible` styles to buttons and inputs instead. High impact, low effort.

3. **Fix the two confirmed contrast failures** — `#999999` and `#AAAAAA` on light backgrounds affect readability for users with low vision. Update to `#767676` minimum.

4. **Consolidate the button** — three button implementations are the source of most token drift and visual inconsistency. Creating one Button with variants pays down technical debt across the whole project.

5. **Unify the navigation** — four nav implementations means four things to maintain. Extract a shared nav partial and update all pages to use it.
