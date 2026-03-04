---
description: Audit a mobile UI for token drift, accessibility issues, touch target violations, and platform convention problems
allowed-tools: Read, Glob, Grep, Bash, AskUserQuestion, Write
---

# Mobile UI Audit

Scan the project (or the path provided as `$1` if given) for design system issues. Report findings grouped by category with file and line references. Be specific — name the file, the line, the value, and what it should be instead.

## Before You Begin

Read `CLAUDE.md` for project context:
- Platform (React Native, Flutter, SwiftUI, Kotlin Compose)
- Token file path and token naming convention
- Project structure and file conventions

Read the token file to build a reference list of approved values.

## What to Check

### 1. Token Drift

Find hardcoded values that should reference tokens instead.

**React Native** — grep `.tsx`, `.ts`, `.jsx`, `.js` files:
- Literal hex strings: `'#[0-9a-fA-F]{3,8}'` inside `StyleSheet.create()` or inline styles
- Magic spacing numbers: `{ margin: 16 }`, `{ padding: 24 }` — check against token scale
- Hardcoded font sizes: `fontSize: 14` not using a token
- Hardcoded border radii: `borderRadius: 8` not using a token

**Flutter** — grep `.dart` files:
- Color literals: `Color(0xFF[0-9a-fA-F]{6})` outside the token file
- Hardcoded font sizes: `fontSize: 14`, `fontSize: 16.0`
- Hardcoded spacing: `EdgeInsets.all(20)`, `EdgeInsets.symmetric(horizontal: 16)`
- Magic numbers in `SizedBox`: `SizedBox(height: 24)` not from `AppSpacing`

**SwiftUI** — grep `.swift` files:
- Color constructors: `Color(red:`, `Color(hex:` outside `Theme.swift`
- Hardcoded font sizes: `.font(.system(size: 14))` not using a `Font` extension
- Hardcoded padding: `.padding(20)`, `.padding(.horizontal, 16)`

**Kotlin Compose** — grep `.kt` files:
- Color literals: `Color(0xFF[0-9a-fA-F]{6})` outside the token file
- Hardcoded font sizes: `fontSize = 14.sp`
- Hardcoded dp values: `16.dp`, `24.dp` in padding/size modifiers not from `AppSpacing`

For each finding:
```
[TOKEN DRIFT] path/to/file.tsx:42
  Found: color: '#2563EB' (hardcoded in StyleSheet.create)
  Token: colors.primary
  Fix: Replace with colors.primary from tokens import
```

---

### 2. Accessibility (WCAG 2.1 AA)

Check for the following. Tag each finding with its WCAG criterion.

**Color contrast** `[WCAG 1.4.3]` `[WCAG 1.4.11]`:
- Normal text (under 18pt/sp regular or 14pt/sp bold): must meet 4.5:1 contrast against its background
- Large text (18pt/sp+ regular or 14pt/sp+ bold): must meet 3:1
- UI components (borders, icons, chart elements): must meet 3:1 against adjacent colors
- Flag combinations likely to fail — show estimated ratio when values can be resolved

**Text alternatives** `[WCAG 1.1.1]`:
- Icon-only interactive elements must have an accessible name (React Native: `accessibilityLabel`; Flutter: `Semantics(label:)`; SwiftUI: `.accessibilityLabel()`; Compose: `contentDescription`)
- Images that convey information must have a semantic label; decorative images must be excluded from accessibility tree

**Accessible names and roles** `[WCAG 4.1.2]`:

**React Native** `[WCAG 4.1.2]`:
- `<Pressable>` or `<TouchableOpacity>` without `accessibilityLabel` prop
- `<Image>` without `accessibilityLabel` (when not decorative)
- Missing `accessibilityRole` on interactive elements (`'button'`, `'link'`, `'checkbox'`, etc.)
- Missing `accessibilityHint` on complex or non-obvious interactions
- `accessible={false}` used incorrectly on interactive elements

**Flutter** `[WCAG 4.1.2]`:
- `GestureDetector` or `InkWell` wrapping a non-semantic widget without a `Semantics` ancestor
- `Image.asset()` or `Image.network()` without `semanticsLabel`
- Missing `Semantics` widget on custom interactive elements
- `excludeSemantics: true` used incorrectly on interactive elements

**SwiftUI** `[WCAG 4.1.2]`:
- `.onTapGesture` on non-interactive elements without `.accessibilityLabel()`
- `Image(systemName:)` used in interactive context without `.accessibilityLabel()`
- Missing `.accessibilityHint()` on elements with non-obvious actions
- `.accessibilityElement(children: .ignore)` used to suppress needed accessibility information

**Kotlin Compose** `[WCAG 4.1.2]`:
- `clickable {}` modifier without `semantics { contentDescription = "..." }`
- `Icon()` composable inside a button without `contentDescription`
- Missing `semantics { }` block on custom interactive composables
- `clearAndSetSemantics {}` used to remove needed accessibility information

**Focus and keyboard navigation** `[WCAG 2.1.1]` `[WCAG 2.4.3]` (applies to hardware keyboard and Switch Access):
- Custom interactive elements that are not reachable via external keyboard
- Focus order that doesn't match visual/logical reading order
- Flag any `accessibilityViewIsModal` or `AccessibilityFocusScope` that incorrectly blocks focus

**Form labels** `[WCAG 1.3.1]`:
- Text inputs without a visible label AND an accessible label — `placeholder` text alone is not sufficient
- Form fields without error message association when in error state

**Error identification** `[WCAG 3.3.1]`:
- Validation errors communicated only by color or icon — must also include a text description
- Error messages not associated programmatically with their input field

For each finding, include the WCAG criterion tag:
```
[ACCESSIBILITY] path/to/file.tsx:87 [WCAG 4.1.2]
  Issue: Pressable has no accessibilityLabel
  Element: <Pressable onPress={handleDelete}><Icon name="trash" /></Pressable>
  Fix: Add accessibilityLabel="Delete item" and accessibilityRole="button"
```

---

### 3. Touch Target Size

Minimum touch target requirements:
- **iOS** (React Native iOS / SwiftUI): 44×44 points (Apple HIG)
- **Android** (React Native Android / Kotlin Compose): 48×48dp (Material Design)

**React Native** — flag interactive elements where the computed touch area is too small:
- `<Pressable>` or `<TouchableOpacity>` with `style={{ width: N, height: N }}` where N < 44
- Missing `minWidth`/`minHeight` on icon buttons
- `hitSlop` used to compensate — note it as a workaround, not a fix

**Flutter** — flag:
- `InkWell` or `GestureDetector` wrapping a widget smaller than 48×48
- `IconButton` with `iconSize` below threshold (default is fine, only flag explicit overrides)
- Missing `constraints` on small interactive widgets

**SwiftUI** — flag:
- `.frame(width: N, height: N)` on interactive elements where N < 44
- Icon buttons without `.frame(minWidth: 44, minHeight: 44)` or `.contentShape()`

**Kotlin Compose** — flag:
- Clickable composables without `Modifier.sizeIn(minWidth = 48.dp, minHeight = 48.dp)`
- `IconButton` with a manually overridden size below 48dp

For each finding:
```
[TOUCH TARGET] path/to/file.swift:134
  Issue: Interactive element below minimum touch target size
  Element: Button with .frame(width: 24, height: 24)
  Required: 44×44pt (Apple HIG)
  Fix: Add .frame(minWidth: 44, minHeight: 44) or increase visual size
```

---

### 4. Platform Convention Issues

**React Native**:
- `Platform.OS` checks for styling that could be handled by platform-specific token values
- iOS shadow applied on Android (or vice versa): using `shadowColor`/`shadowOffset` without wrapping in `Platform.OS === 'ios'`
- Missing `<SafeAreaView>` at the root of screen components
- `TouchableNativeFeedback` used on iOS, or `TouchableHighlight` used inconsistently
- `KeyboardAvoidingView` missing on screens with text inputs

**Flutter**:
- `dart:io` `Platform.isIOS` checks for styling that widgets handle automatically
- Non-adaptive widgets where adaptive versions exist (`CupertinoSwitch` vs `Switch.adaptive`)
- Missing `SafeArea` widget at top-level scaffold content
- `MediaQuery.of(context)` called without null check in older-pattern code

**SwiftUI**:
- Hardcoded `UIScreen.main.bounds` instead of `@Environment(\.horizontalSizeClass)`
- Missing `.ignoresSafeArea()` used incorrectly (or missing where needed)
- `@State` used for data that should be in an `@ObservableObject`
- `NavigationView` used instead of `NavigationStack` (deprecated in iOS 16+)

**Kotlin Compose**:
- Missing `WindowInsets` handling for system bars
- `LocalContext.current` called inside composition to get resources (should use `stringResource`)
- Missing predictive back support (`BackHandler` not implemented)
- `rememberCoroutineScope` used in composition without proper lifecycle handling

For each finding:
```
[PLATFORM CONVENTION] path/to/file.tsx:67
  Issue: Shadow properties applied without platform check
  Code: style={{ shadowColor: '#000', shadowOffset: { width: 0, height: 2 } }}
  Fix: Wrap in Platform.OS === 'ios' check; use elevation for Android
```

---

## Output Format

Structure findings by category. Show the most critical issues first within each section.

After all findings, write a summary table:

| Category | Critical | Warning | Suggestion |
|----------|----------|---------|------------|
| Token drift | — | — | — |
| Accessibility | — | — | — |
| Touch targets | — | — | — |
| Platform conventions | — | — | — |
| **Total** | | | |

End with **3-5 prioritized next steps** based on the highest-impact findings.

---

## Interactive Fix Mode

After presenting the summary table and next steps, enter interactive fix mode.

### Step 1 — Ask which categories to fix

Use AskUserQuestion with `multiSelect: true`:
- question: "Which categories of findings would you like me to fix now?"
- header: "Fix categories"
- options:
  - Token drift (replace hardcoded values with token references)
  - Accessibility (apply accessibilityLabel, Semantics, contentDescription, and role fixes)
  - Inconsistency / Touch targets (consolidate patterns, enforce minimum touch target sizes)
  - Contrast / Platform conventions (color fixes and platform-specific convention violations)

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
Fixed: src/components/Button.tsx:42
  - <Pressable onPress={handleDelete}>
  + <Pressable onPress={handleDelete} accessibilityLabel="Delete item" accessibilityRole="button">
```

If **Skip**: move to the next finding in the category.

If **Stop**: stop the current category. Move to the next selected category if any.

### Step 3 — Category summary

After finishing all findings in a category (or on Stop):
```
Accessibility: 5 fixes applied, 2 skipped
```

### Step 4 — Final summary

After all selected categories are processed:
```
Fix session complete.
  Token drift:               3 applied, 1 skipped
  Accessibility:             5 applied, 2 skipped
  Inconsistency/Touch:       2 applied, 0 skipped
  Contrast/Platform:         1 applied, 1 skipped
```

---

## Scope

If `$1` is provided, limit the scan to that path. If not, scan the entire project, excluding `node_modules`, `.git`, `build`, `.dart_tool`, `.gradle`, and other build output directories.
