---
description: Audit a mobile UI for token drift, accessibility issues, touch target violations, and platform convention problems
allowed-tools: Read, Glob, Grep, Bash
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

### 2. Accessibility

**React Native**:
- `<Pressable>` or `<TouchableOpacity>` without `accessibilityLabel` prop
- `<Image>` without `accessibilityLabel` (when not decorative)
- Missing `accessibilityRole` on interactive elements (use `'button'`, `'link'`, `'checkbox'`, etc.)
- Missing `accessibilityHint` on complex interactions
- `accessible={false}` used incorrectly on interactive elements
- Icon-only pressables with no `accessibilityLabel`

**Flutter**:
- `GestureDetector` or `InkWell` wrapping a non-semantic widget without a `Semantics` ancestor
- `Image.asset()` or `Image.network()` without `semanticsLabel`
- Missing `Semantics` widget on custom interactive elements
- `excludeSemantics: true` used incorrectly

**SwiftUI**:
- `.onTapGesture` on non-interactive elements without `.accessibilityLabel()`
- `Image(systemName:)` used in interactive context without `.accessibilityLabel()`
- Missing `.accessibilityHint()` on elements with non-obvious actions
- Incorrect `.accessibilityElement(children: .ignore)` on compound elements

**Kotlin Compose**:
- `clickable {}` modifier without `semantics { contentDescription = "..." }`
- `Icon()` composable inside a button without `contentDescription`
- Missing `semantics { }` block on custom interactive composables
- `clearAndSetSemantics {}` used to remove needed accessibility information

For each finding:
```
[ACCESSIBILITY] path/to/file.tsx:87
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

## Scope

If `$1` is provided, limit the scan to that path. If not, scan the entire project, excluding `node_modules`, `.git`, `build`, `.dart_tool`, `.gradle`, and other build output directories.
