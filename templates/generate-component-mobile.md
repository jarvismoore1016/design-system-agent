---
description: Generate a new mobile UI component using the project's design tokens and platform conventions
allowed-tools: Read, Write, Bash
argument-hint: component-name
---

# Generate Component: $1

## Before You Build

Read `CLAUDE.md` to understand:
- Platform (React Native, Flutter, SwiftUI, Kotlin Compose)
- Token file path and naming convention
- Existing component patterns to follow

Read the token file so you know exactly what values are available. Every color, spacing value, and size in the generated component must come from the token system.

Find 1-2 existing components in the project and read them. Match their conventions exactly.

## What to Generate

Build a complete `$1` component that's ready to use without modification.

### Uses tokens for everything

No hardcoded hex values, no magic numbers. Reference token values directly:
- **React Native**: `import { colors, spacing, typography, radius } from '../tokens'`
- **Flutter**: `AppColors.primary`, `AppSpacing.md`, `AppRadius.md`
- **SwiftUI**: `Color.primary` (from Token extension), `AppSpacing.md`, `Font.body`
- **Kotlin Compose**: `AppColors.Primary`, `AppSpacing.md`, `MaterialTheme.colorScheme`

### Includes all interactive states

Every state must be visually distinct:
- **Default** — resting appearance
- **Pressed** — feedback during touch (scale, opacity, or color change)
- **Focused** — keyboard/accessibility focus (when platform supports it)
- **Disabled** — visually muted, not interactive

### Is accessible by default

The generated component must:
- Include all required accessibility attributes for the platform (see below)
- Meet minimum touch target requirements (44pt iOS, 48dp Android)
- Have meaningful accessible names, not just visual labels

### Follows project conventions

- File name matches the existing naming pattern
- Uses the same component organization as existing files
- Prop/parameter pattern matches existing components

---

## Output Format by Platform

### React Native

Provide a `.tsx` file:

```typescript
// ComponentName.tsx
import React, { useState } from 'react';
import {
  Pressable,
  Text,
  StyleSheet,
  Platform,
  ActivityIndicator,
} from 'react-native';
import { colors, spacing, typography, radius } from '../tokens';

interface ComponentNameProps {
  label: string;
  onPress: () => void;
  variant?: 'primary' | 'secondary' | 'ghost' | 'destructive';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
  accessibilityHint?: string;
}

export function ComponentName({
  label,
  onPress,
  variant = 'primary',
  size = 'md',
  disabled = false,
  loading = false,
  accessibilityHint,
}: ComponentNameProps) {
  return (
    <Pressable
      onPress={onPress}
      disabled={disabled || loading}
      accessibilityLabel={label}
      accessibilityRole="button"
      accessibilityState={{ disabled: disabled || loading }}
      accessibilityHint={accessibilityHint}
      style={({ pressed }) => [
        styles.base,
        styles[variant],
        styles[size],
        pressed && styles.pressed,
        (disabled || loading) && styles.disabled,
      ]}
    >
      {loading ? (
        <ActivityIndicator
          color={variant === 'primary' ? colors.textOnPrimary : colors.primary}
          accessibilityLabel="Loading"
        />
      ) : (
        <Text style={[styles.label, styles[`${variant}Label`], styles[`${size}Label`]]}>
          {label}
        </Text>
      )}
    </Pressable>
  );
}

const styles = StyleSheet.create({
  base: {
    alignItems: 'center',
    justifyContent: 'center',
    borderRadius: radius.md,
    minHeight: 44, // Apple HIG minimum touch target
    minWidth: 44,
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 1 },
        shadowOpacity: 0.1,
        shadowRadius: 2,
      },
      android: {
        elevation: 2,
      },
    }),
  },
  // Variants
  primary: {
    backgroundColor: colors.primary,
  },
  secondary: {
    backgroundColor: 'transparent',
    borderWidth: 1.5,
    borderColor: colors.primary,
  },
  ghost: {
    backgroundColor: 'transparent',
  },
  destructive: {
    backgroundColor: colors.danger,
  },
  // Sizes
  sm: {
    paddingHorizontal: spacing.sm,
    paddingVertical: spacing.xs,
  },
  md: {
    paddingHorizontal: spacing.md,
    paddingVertical: spacing.sm,
  },
  lg: {
    paddingHorizontal: spacing.lg,
    paddingVertical: spacing.md,
  },
  // States
  pressed: {
    opacity: 0.75,
  },
  disabled: {
    opacity: 0.4,
  },
  // Labels
  label: {
    fontWeight: typography.fontWeight.semibold,
  },
  primaryLabel: {
    color: colors.textOnPrimary,
    fontSize: typography.fontSize.base,
  },
  secondaryLabel: {
    color: colors.primary,
    fontSize: typography.fontSize.base,
  },
  ghostLabel: {
    color: colors.primary,
    fontSize: typography.fontSize.base,
  },
  destructiveLabel: {
    color: colors.textOnPrimary,
    fontSize: typography.fontSize.base,
  },
  smLabel: { fontSize: typography.fontSize.sm },
  mdLabel: { fontSize: typography.fontSize.base },
  lgLabel: { fontSize: typography.fontSize.lg },
});

// Usage:
// <ComponentName label="Save changes" onPress={handleSave} />
// <ComponentName label="Delete account" variant="destructive" onPress={handleDelete} />
// <ComponentName label="Processing" loading onPress={() => {}} />
```

---

### Flutter

Provide a `.dart` file:

```dart
// component_name.dart
import 'package:flutter/material.dart';
import '../tokens.dart';

enum ComponentNameVariant { primary, secondary, ghost, destructive }
enum ComponentNameSize { sm, md, lg }

class ComponentName extends StatelessWidget {
  final String label;
  final VoidCallback? onPressed;
  final ComponentNameVariant variant;
  final ComponentNameSize size;
  final bool isLoading;

  const ComponentName({
    super.key,
    required this.label,
    required this.onPressed,
    this.variant = ComponentNameVariant.primary,
    this.size = ComponentNameSize.md,
    this.isLoading = false,
  });

  @override
  Widget build(BuildContext context) {
    return Semantics(
      label: label,
      button: true,
      enabled: onPressed != null && !isLoading,
      child: _buildButton(context),
    );
  }

  Widget _buildButton(BuildContext context) {
    final isDisabled = onPressed == null || isLoading;

    // Minimum touch target: 48×48dp (Material)
    return ConstrainedBox(
      constraints: const BoxConstraints(minWidth: 48, minHeight: 48),
      child: _buildVariant(context, isDisabled),
    );
  }

  Widget _buildVariant(BuildContext context, bool isDisabled) {
    final padding = _getPadding();
    final content = isLoading
        ? SizedBox(
            width: 20,
            height: 20,
            child: CircularProgressIndicator(
              strokeWidth: 2,
              color: variant == ComponentNameVariant.primary
                  ? Colors.white
                  : AppColors.primary,
            ),
          )
        : Text(label, style: _getLabelStyle());

    switch (variant) {
      case ComponentNameVariant.primary:
        return ElevatedButton(
          onPressed: isDisabled ? null : onPressed,
          style: ElevatedButton.styleFrom(
            backgroundColor: AppColors.primary,
            disabledBackgroundColor: AppColors.primary.withOpacity(0.4),
            padding: padding,
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(AppRadius.md),
            ),
          ),
          child: content,
        );
      case ComponentNameVariant.secondary:
        return OutlinedButton(
          onPressed: isDisabled ? null : onPressed,
          style: OutlinedButton.styleFrom(
            foregroundColor: AppColors.primary,
            side: BorderSide(color: AppColors.primary, width: 1.5),
            padding: padding,
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(AppRadius.md),
            ),
          ),
          child: content,
        );
      case ComponentNameVariant.ghost:
        return TextButton(
          onPressed: isDisabled ? null : onPressed,
          style: TextButton.styleFrom(
            foregroundColor: AppColors.primary,
            padding: padding,
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(AppRadius.md),
            ),
          ),
          child: content,
        );
      case ComponentNameVariant.destructive:
        return ElevatedButton(
          onPressed: isDisabled ? null : onPressed,
          style: ElevatedButton.styleFrom(
            backgroundColor: AppColors.danger,
            disabledBackgroundColor: AppColors.danger.withOpacity(0.4),
            padding: padding,
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(AppRadius.md),
            ),
          ),
          child: content,
        );
    }
  }

  EdgeInsets _getPadding() {
    switch (size) {
      case ComponentNameSize.sm:
        return EdgeInsets.symmetric(
          horizontal: AppSpacing.sm,
          vertical: AppSpacing.xs,
        );
      case ComponentNameSize.md:
        return EdgeInsets.symmetric(
          horizontal: AppSpacing.md,
          vertical: AppSpacing.sm,
        );
      case ComponentNameSize.lg:
        return EdgeInsets.symmetric(
          horizontal: AppSpacing.lg,
          vertical: AppSpacing.md,
        );
    }
  }

  TextStyle _getLabelStyle() {
    final fontSize = switch (size) {
      ComponentNameSize.sm => AppTextStyles.bodySmall.fontSize!,
      ComponentNameSize.md => AppTextStyles.body.fontSize!,
      ComponentNameSize.lg => AppTextStyles.heading.fontSize!,
    };
    return TextStyle(
      fontSize: fontSize,
      fontWeight: FontWeight.w600,
    );
  }
}

// Usage:
// ComponentName(label: 'Save changes', onPressed: handleSave)
// ComponentName(label: 'Delete', variant: ComponentNameVariant.destructive, onPressed: handleDelete)
// ComponentName(label: 'Loading', isLoading: true, onPressed: null)
```

---

### SwiftUI

Provide a `.swift` file:

```swift
// ComponentName.swift
import SwiftUI

enum ComponentNameVariant {
    case primary, secondary, ghost, destructive
}

enum ComponentNameSize {
    case sm, md, lg
}

struct ComponentName: View {
    let label: String
    let action: () -> Void
    var variant: ComponentNameVariant = .primary
    var size: ComponentNameSize = .md
    var isDisabled: Bool = false
    var isLoading: Bool = false
    var accessibilityHint: String? = nil

    var body: some View {
        Button(action: action) {
            Group {
                if isLoading {
                    ProgressView()
                        .progressViewStyle(CircularProgressViewStyle(tint: labelColor))
                        .frame(width: 20, height: 20)
                } else {
                    Text(label)
                        .font(labelFont)
                        .fontWeight(.semibold)
                }
            }
            .padding(padding)
            .frame(minWidth: 44, minHeight: 44) // Apple HIG minimum
        }
        .background(backgroundColor)
        .foregroundColor(labelColor)
        .cornerRadius(AppRadius.md)
        .overlay(
            variant == .secondary
                ? RoundedRectangle(cornerRadius: AppRadius.md)
                    .strokeBorder(Color.primary, lineWidth: 1.5)
                : nil
        )
        .disabled(isDisabled || isLoading)
        .opacity(isDisabled || isLoading ? 0.4 : 1.0)
        .accessibilityLabel(label)
        .accessibilityHint(accessibilityHint ?? "")
        .accessibilityAddTraits(.isButton)
    }

    private var backgroundColor: Color {
        switch variant {
        case .primary: return Color.primary
        case .secondary: return .clear
        case .ghost: return .clear
        case .destructive: return Color.danger
        }
    }

    private var labelColor: Color {
        switch variant {
        case .primary, .destructive: return Color.textOnPrimary
        case .secondary, .ghost: return Color.primary
        }
    }

    private var labelFont: Font {
        switch size {
        case .sm: return Font.bodySmall
        case .md: return Font.body
        case .lg: return Font.heading
        }
    }

    private var padding: EdgeInsets {
        switch size {
        case .sm: return EdgeInsets(
            top: AppSpacing.xs, leading: AppSpacing.sm,
            bottom: AppSpacing.xs, trailing: AppSpacing.sm)
        case .md: return EdgeInsets(
            top: AppSpacing.sm, leading: AppSpacing.md,
            bottom: AppSpacing.sm, trailing: AppSpacing.md)
        case .lg: return EdgeInsets(
            top: AppSpacing.md, leading: AppSpacing.lg,
            bottom: AppSpacing.md, trailing: AppSpacing.lg)
        }
    }
}

// Usage:
// ComponentName(label: "Save changes", action: handleSave)
// ComponentName(label: "Delete", variant: .destructive, action: handleDelete)
// ComponentName(label: "Processing", isLoading: true, action: {})
```

---

### Kotlin Compose

Provide a `.kt` file:

```kotlin
// ComponentName.kt
package com.yourapp.ui.components

import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.semantics.contentDescription
import androidx.compose.ui.semantics.semantics
import com.yourapp.ui.theme.*

enum class ComponentNameVariant { Primary, Secondary, Ghost, Destructive }
enum class ComponentNameSize { Sm, Md, Lg }

@Composable
fun ComponentName(
    label: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    variant: ComponentNameVariant = ComponentNameVariant.Primary,
    size: ComponentNameSize = ComponentNameSize.Md,
    enabled: Boolean = true,
    isLoading: Boolean = false,
) {
    val buttonModifier = modifier
        .sizeIn(minWidth = 48.dp, minHeight = 48.dp) // Material min touch target
        .semantics { contentDescription = label }

    when (variant) {
        ComponentNameVariant.Primary -> Button(
            onClick = onClick,
            modifier = buttonModifier,
            enabled = enabled && !isLoading,
            contentPadding = getPadding(size),
            colors = ButtonDefaults.buttonColors(
                containerColor = AppColors.Primary,
                disabledContainerColor = AppColors.Primary.copy(alpha = 0.4f),
            ),
            shape = MaterialTheme.shapes.small.copy(
                all = androidx.compose.foundation.shape.CornerSize(AppRadius.md)
            ),
        ) {
            ButtonContent(label, isLoading, MaterialTheme.colorScheme.onPrimary, size)
        }

        ComponentNameVariant.Secondary -> OutlinedButton(
            onClick = onClick,
            modifier = buttonModifier,
            enabled = enabled && !isLoading,
            contentPadding = getPadding(size),
            border = ButtonDefaults.outlinedButtonBorder.copy(
                brush = androidx.compose.ui.graphics.SolidColor(AppColors.Primary)
            ),
        ) {
            ButtonContent(label, isLoading, AppColors.Primary, size)
        }

        ComponentNameVariant.Ghost -> TextButton(
            onClick = onClick,
            modifier = buttonModifier,
            enabled = enabled && !isLoading,
            contentPadding = getPadding(size),
        ) {
            ButtonContent(label, isLoading, AppColors.Primary, size)
        }

        ComponentNameVariant.Destructive -> Button(
            onClick = onClick,
            modifier = buttonModifier,
            enabled = enabled && !isLoading,
            contentPadding = getPadding(size),
            colors = ButtonDefaults.buttonColors(
                containerColor = AppColors.Danger,
                disabledContainerColor = AppColors.Danger.copy(alpha = 0.4f),
            ),
        ) {
            ButtonContent(label, isLoading, MaterialTheme.colorScheme.onError, size)
        }
    }
}

@Composable
private fun ButtonContent(
    label: String,
    isLoading: Boolean,
    contentColor: androidx.compose.ui.graphics.Color,
    size: ComponentNameSize,
) {
    if (isLoading) {
        CircularProgressIndicator(
            modifier = Modifier.size(20.dp),
            color = contentColor,
            strokeWidth = 2.dp,
        )
    } else {
        Text(
            text = label,
            fontSize = getFontSize(size),
            fontWeight = androidx.compose.ui.text.font.FontWeight.SemiBold,
        )
    }
}

private fun getPadding(size: ComponentNameSize) = when (size) {
    ComponentNameSize.Sm -> PaddingValues(horizontal = AppSpacing.sm, vertical = AppSpacing.xs)
    ComponentNameSize.Md -> PaddingValues(horizontal = AppSpacing.md, vertical = AppSpacing.sm)
    ComponentNameSize.Lg -> PaddingValues(horizontal = AppSpacing.lg, vertical = AppSpacing.md)
}

private fun getFontSize(size: ComponentNameSize) = when (size) {
    ComponentNameSize.Sm -> AppTypography.bodySmall
    ComponentNameSize.Md -> AppTypography.body
    ComponentNameSize.Lg -> AppTypography.heading
}

// Usage:
// ComponentName(label = "Save changes", onClick = { handleSave() })
// ComponentName(label = "Delete", variant = ComponentNameVariant.Destructive, onClick = { handleDelete() })
// ComponentName(label = "Processing", isLoading = true, onClick = {})
```

---

## After the Component

Save the file to the appropriate location based on the platform convention:
- **React Native**: `src/components/` or `components/`
- **Flutter**: `lib/widgets/` or `lib/components/`
- **SwiftUI**: The main source group, alongside other view files
- **Kotlin Compose**: `app/src/main/java/.../ui/components/`

Tell the user:
- Where the file was saved
- What props/parameters/variants it supports
- How to import and use it
- Which tokens it references
