---
description: Interactive guided setup for your mobile design system assistant — creates tokens, audit, generation, and documentation commands tailored to your mobile platform
allowed-tools: AskUserQuestion, Bash, Read, Write, Glob, Grep
---

# Mobile Design System Assistant Setup

You are guiding the user through a six-step setup process for a mobile project. At every step, explain what you are doing and why **before** you do it. Tone: knowledgeable colleague, not tutorial voice. The user should feel like they're building something together with you, not watching a script run.

At the end, the user will have:
- A design token file (all their existing colors, spacing, and typography — named and organized in platform-appropriate format)
- `/audit-ui` — scans for visual inconsistencies, token drift, accessibility issues, and platform convention violations
- `/generate-component` — builds new components from the token system in the correct platform format
- `/generate-docs` — documents components in structured markdown
- `CLAUDE.md` — persistent project context for all future sessions

---

## Opening Message

Begin by saying:

> **Mobile Design System Assistant Setup**
>
> I'm going to set up an AI-powered design system assistant for your mobile project. By the end you'll have a token file, three slash commands, and a CLAUDE.md that keeps me informed about your design system in every future session.
>
> Six steps, 2-4 minutes each. I'll explain what I'm doing at each stage and ask for your input on the decisions that matter. Let's go.

---

## Step 1 — Platform Detection

Say: "First, let me look at your project to understand what platform we're working with."

### Scan for mobile platform

Check for these signals, in order:

**React Native**:
- `package.json` containing `react-native` in `dependencies` or `devDependencies`
- `metro.config.js` or `metro.config.ts` in the project root
- `.tsx` or `.jsx` source files using `StyleSheet.create()` or importing from `react-native`
- `android/` and `ios/` directories present

**Flutter**:
- `pubspec.yaml` with a `flutter:` section
- `lib/` directory containing `.dart` files
- `flutter_lints` or `flutter_test` in `pubspec.yaml`

**SwiftUI**:
- `.swift` source files present
- `ContentView.swift` exists, or files containing `@main` annotation
- No `react-native` in any `package.json`
- `*.xcodeproj` or `*.xcworkspace` directory present

**Kotlin Compose**:
- `build.gradle` or `build.gradle.kts` with `compose` in the dependencies block
- `.kt` files containing `@Composable` annotations
- `composeOptions` block in any `build.gradle[.kts]`

### Present findings

Summarize what you found, for example:
> Here's what I found in your project:
> - **Platform**: React Native with TypeScript
> - **Existing tokens**: `theme.ts` detected
> - **Source files scanned**: 34 files

Use AskUserQuestion:
- question: "Does this look right? I'll use this to customize everything I create."
- header: "Platform check"
- options:
  - Looks right (proceed)
  - Something's off (let me correct it)

If they say something's off, ask:
- question: "What's your mobile platform?"
- header: "Platform"
- options:
  - React Native
  - Flutter
  - SwiftUI
  - Kotlin Compose

Store the confirmed platform — you'll need it in every subsequent step.

---

## Step 2 — Token Extraction and Creation

Say: "Now I'll scan your project for all the design values currently in use. Mobile codebases tend to have hardcoded colors in StyleSheet.create() calls, magic spacing numbers scattered across files, and font sizes with no scale. We're going to name them, consolidate them, and give you a single source of truth."

### Scan for values by platform

**React Native** — grep across `.tsx`, `.ts`, `.jsx`, `.js` files for:
- Colors: `'#[0-9a-fA-F]{3,8}'`, `rgba?\(\s*[\d,\s.]+\)`, color keys in `StyleSheet.create()`
- Spacing: numeric values in `margin`, `padding`, `gap`, `width`, `height` style properties
- Font sizes: `fontSize:\s*\d+`
- Border radii: `borderRadius:\s*\d+`
- Existing token/theme files: `theme.ts`, `tokens.ts`, `colors.ts`, `spacing.ts`

**Flutter** — grep across `.dart` files for:
- Colors: `Color\(0xFF[0-9a-fA-F]+\)`, `Colors\.\w+`, color properties in `ThemeData`
- Font sizes: `fontSize:\s*[\d.]+`
- Spacing: `EdgeInsets\.(all|only|symmetric)\([\d.]+\)`, `SizedBox\(.*height:`, `SizedBox\(.*width:`
- Existing token files: `app_colors.dart`, `theme.dart`, `tokens.dart`

**SwiftUI** — grep across `.swift` files for:
- Colors: `Color\("[^"]+"\)`, `Color\(red:`, `.foregroundColor(\.`, `Color\.init`
- Fonts: `\.font\(\.system\(size:`, `Font\.system\(size:`
- Spacing: `.padding\(\d+\)`, `.frame\(width:`, `.frame\(height:`
- Existing token files: `Theme.swift`, `Colors.swift`, `Fonts.swift`

**Kotlin Compose** — grep across `.kt` files for:
- Colors: `Color\(0xFF[0-9a-fA-F]+\)`, `Color\([\d,\s]+\)`, colors in `MaterialTheme.colorScheme`
- Font sizes: `fontSize\s*=\s*[\d.]+\.sp`
- Spacing: `[\d.]+\.dp` used in padding/size modifiers
- Existing token files: `Color.kt`, `Theme.kt`, `Tokens.kt`

Deduplicate all values. Group similar colors together. Sort spacing values numerically.

### Present findings

Report what you found:
> Here's what I found:
> - **Colors**: 12 unique values — 3 blue variants, 3 grays, 4 neutrals, 2 others
> - **Spacing**: 8 unique values (4dp through 48dp)
> - **Font sizes**: 6 unique values
> - **Border radii**: 4 values
> - **Existing token file**: None detected

### Ask for naming preference

Use AskUserQuestion:
- question: "How would you like to name your tokens?"
- header: "Token style"
- options:
  - Semantic (colorPrimary, spacingMd, fontSizeBase)
  - Numeric scale (blue500, spacing4, textLg)
  - Keep it simple (I'll rename them later)

### Write the token file

Based on confirmed platform and naming preference, generate and write the appropriate file:

**React Native** → write `tokens.ts` in the project root (or `src/tokens.ts` if a `src/` directory exists):
```typescript
// Design Tokens — generated by Mobile Design System Assistant

export const colors = {
  primary: '#[value]',
  // ...
};

export const spacing = {
  xs: [value],
  sm: [value],
  md: [value],
  lg: [value],
  xl: [value],
};

export const typography = {
  fontSize: {
    sm: [value],
    base: [value],
    lg: [value],
    xl: [value],
    xxl: [value],
  },
  fontWeight: {
    regular: '400',
    medium: '500',
    semibold: '600',
    bold: '700',
  },
};

export const radius = {
  sm: [value],
  md: [value],
  lg: [value],
  full: 9999,
};
```

**Flutter** → write `lib/tokens.dart`:
```dart
// Design Tokens — generated by Mobile Design System Assistant
import 'package:flutter/material.dart';

class AppColors {
  static const Color primary = Color(0xFF[value]);
  // ...
}

class AppSpacing {
  static const double xs = [value];
  static const double sm = [value];
  static const double md = [value];
  static const double lg = [value];
  static const double xl = [value];
}

class AppTextStyles {
  static const TextStyle bodySmall = TextStyle(fontSize: [value]);
  static const TextStyle body = TextStyle(fontSize: [value]);
  static const TextStyle heading = TextStyle(fontSize: [value], fontWeight: FontWeight.w600);
}

class AppRadius {
  static const double sm = [value];
  static const double md = [value];
  static const double lg = [value];
}
```

**SwiftUI** → write `Theme.swift` in the project's main source directory:
```swift
// Design Tokens — generated by Mobile Design System Assistant
import SwiftUI

extension Color {
    static let primary = Color("[value]")  // or Color(hex: "...")
    // ...
}

extension Font {
    static let bodySmall = Font.system(size: [value])
    static let body = Font.system(size: [value])
    static let heading = Font.system(size: [value], weight: .semibold)
}

struct AppSpacing {
    static let xs: CGFloat = [value]
    static let sm: CGFloat = [value]
    static let md: CGFloat = [value]
    static let lg: CGFloat = [value]
    static let xl: CGFloat = [value]
}

struct AppRadius {
    static let sm: CGFloat = [value]
    static let md: CGFloat = [value]
    static let lg: CGFloat = [value]
}
```

**Kotlin Compose** → write `ui/theme/Tokens.kt` (or `app/src/main/java/.../ui/theme/Tokens.kt`):
```kotlin
// Design Tokens — generated by Mobile Design System Assistant
package com.yourapp.ui.theme

import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp

object AppColors {
    val Primary = Color(0xFF[value])
    // ...
}

object AppSpacing {
    val xs = [value].dp
    val sm = [value].dp
    val md = [value].dp
    val lg = [value].dp
    val xl = [value].dp
}

object AppTypography {
    val bodySmall = [value].sp
    val body = [value].sp
    val heading = [value].sp
}

object AppRadius {
    val sm = [value].dp
    val md = [value].dp
    val lg = [value].dp
}
```

After writing, show the user a preview and the file path. Tell them they can edit it directly any time.

---

## Step 3 — Audit Command

Say: "Now I'll create your `/audit-ui` command. Run this any time you want a snapshot of where your project stands on visual consistency, accessibility, and platform conventions. It's tuned specifically for [platform]."

Write `.claude/commands/audit-ui.md` using the content from `templates/audit-ui-mobile.md`, with the `PLATFORM` placeholder replaced by the detected platform name.

After writing the file, use AskUserQuestion:
- question: "Would you like to adjust what the audit checks for?"
- header: "Audit scope"
- options:
  - Looks good (continue)
  - Add more checks
  - Remove some checks

---

## Step 4 — Component Generation Command

Say: "Now I'll create your `/generate-component` command. Give it a component name and it builds a complete, accessible, token-using component in [platform] format — ready to use without modification."

Scan the codebase to find 2-3 existing components as reference patterns. Look for files named Button, Card, Input, or Modal.

Write `.claude/commands/generate-component.md` using the content from `templates/generate-component-mobile.md`, inserting the platform context.

After writing, demonstrate by generating a small Button component based on the tokens from Step 2. Show the output.

---

## Step 5 — Documentation Command

Say: "Last command. `/generate-docs` produces structured documentation for any component — variants, states, API, token usage, accessibility notes, platform-specific gesture behavior, and code examples."

Write `.claude/commands/generate-docs.md` using the content from `templates/generate-docs-mobile.md`.

After writing, generate documentation for the component created in Step 4.

---

## Step 6 — CLAUDE.md and Summary

Say: "Last step. I'm creating a CLAUDE.md file that gives me persistent memory about your design system. Every future session, I'll read this before doing anything."

Write `CLAUDE.md` in the project root with the actual discovered values:

```markdown
# [Project Name] — Mobile Design System Context

> This file gives Claude persistent context about this project's mobile design system.
> Update it any time conventions change.

## Platform
- **Platform**: [React Native / Flutter / SwiftUI / Kotlin Compose]
- **Language**: [TypeScript / Dart / Swift / Kotlin]
- **Min OS target**: [e.g., iOS 16+, Android API 26+]

## Design Tokens
- **Token file**: `[path to token file]`
- **Naming convention**: [semantic / numeric scale / other]
- **Key tokens**:
  - Colors: [list 5-8 primary color tokens with values]
  - Spacing scale: [list spacing tokens with dp/pt values]
  - Typography: [list primary font sizes]
  - Border radius: [list radius tokens]

## Design Conventions
- **Touch targets**: [iOS: 44×44pt / Android: 48×48dp]
- **Primary color**: [description]
- **Spacing**: [base unit and scale description]
- **Typography**: [font family, base size, scale]

## Available Slash Commands
- `/audit-ui [path]` — Checks for token drift, accessibility issues, touch target violations, and platform convention issues
- `/generate-component [name]` — Generates a platform-native component using the token system
- `/generate-docs [name]` — Documents a component with variants, states, API, and examples

## Project Structure
[Brief description of where components, styles, and assets live]

## Accessibility Target
- iOS: WCAG AA + Apple Accessibility Guidelines (VoiceOver, minimum 44×44pt touch targets)
- Android: WCAG AA + Material accessibility guidelines (TalkBack, minimum 48×48dp touch targets)

## Notes
[Any platform-specific patterns, gotchas, or conventions discovered during setup]
```

### Present the summary

After writing CLAUDE.md, give the user a complete summary:

> **Setup complete. Here's what was created:**
>
> | File | Purpose |
> |------|---------|
> | `[token file]` | Design tokens — all your colors, spacing, and typography |
> | `.claude/commands/audit-ui.md` | `/audit-ui` — scan for inconsistencies, a11y issues, touch target violations |
> | `.claude/commands/generate-component.md` | `/generate-component` — build new [platform] components from tokens |
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
> Your mobile design system assistant is ready.
