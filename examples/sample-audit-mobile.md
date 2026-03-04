# Mobile UI Audit Report

**Project**: ShopApp Mobile
**Platform**: React Native (TypeScript)
**Token file**: `src/tokens.ts`
**Scanned**: 28 files (18 screens, 7 components, 3 utility files — 1,843 lines total)
**Date**: Run `/audit-ui` in Claude Code to generate a live version for your project

---

## Token Drift — 11 findings

> Hardcoded colors, spacing, and sizes that should reference the token system

```
[TOKEN DRIFT] src/screens/HomeScreen.tsx:47
  Found: color: '#2563EB' (hardcoded in StyleSheet.create)
  Token: colors.primary
  Fix: Replace with colors.primary from tokens import

[TOKEN DRIFT] src/screens/HomeScreen.tsx:89
  Found: backgroundColor: '#F5F5F5' (hardcoded in StyleSheet.create)
  Token: colors.surface
  Fix: Replace with colors.surface

[TOKEN DRIFT] src/screens/ProductDetailScreen.tsx:134
  Found: fontSize: 14 (hardcoded in StyleSheet.create)
  Token: typography.fontSize.sm
  Fix: Replace with typography.fontSize.sm

[TOKEN DRIFT] src/screens/ProductDetailScreen.tsx:201
  Found: padding: 20 (magic number in StyleSheet.create)
  Token: spacing.md (20)
  Fix: Replace with spacing.md

[TOKEN DRIFT] src/screens/CheckoutScreen.tsx:67
  Found: borderRadius: 8 (hardcoded)
  Token: radius.md (8)
  Fix: Replace with radius.md

[TOKEN DRIFT] src/components/ProductCard.tsx:28
  Found: marginBottom: 16 (hardcoded)
  Token: spacing.sm (16)
  Fix: Replace with spacing.sm

[TOKEN DRIFT] src/components/ProductCard.tsx:45
  Found: color: '#1E293B' (hardcoded — approximates colors.textPrimary)
  Token: colors.textPrimary (#1E293B)
  Fix: Replace with colors.textPrimary

[TOKEN DRIFT] src/components/Badge.tsx:12
  Found: backgroundColor: '#EFF6FF' (hardcoded — approximates colors.primaryLight)
  Token: colors.primaryLight
  Fix: Replace with colors.primaryLight

[TOKEN DRIFT] src/components/Badge.tsx:13
  Found: color: '#1D4ED8' (hardcoded — different blue than colors.primary)
  Note: This is a different value from colors.primary (#2563EB). Decide: update the token or add colors.primaryDark.
  Fix: Align with token system — either add a token or use colors.primary

[TOKEN DRIFT] src/components/SearchBar.tsx:31
  Found: height: 44 (magic number)
  Token: spacing.xxl (44)
  Fix: Replace with spacing.xxl (or add a component-specific token)

[TOKEN DRIFT] src/screens/CartScreen.tsx:89
  Found: shadowColor: '#000000' (hardcoded)
  Token: N/A — add a shadows.sm token to the token file
  Fix: Add shadows.sm to tokens.ts, then reference it here
```

---

## Accessibility — 8 findings

> Issues that affect users relying on VoiceOver, TalkBack, or switch access

```
[ACCESSIBILITY — CRITICAL] src/components/IconButton.tsx:23
  Issue: Pressable has no accessibilityLabel
  Element: <Pressable onPress={onPress}><Icon name="cart" size={24} /></Pressable>
  Fix: Add accessibilityLabel="Open cart" and accessibilityRole="button"

[ACCESSIBILITY — CRITICAL] src/components/ImageCarousel.tsx:67
  Issue: Product images have no accessibilityLabel
  Element: <Image source={{ uri: imageUrl }} style={styles.image} />
  Fix: Add accessibilityLabel={`Product image: ${productName}`}

[ACCESSIBILITY — CRITICAL] src/screens/CheckoutScreen.tsx:112
  Issue: TextInput has no accessibilityLabel
  Element: <TextInput placeholder="Card number" style={styles.input} />
  Fix: Add accessibilityLabel="Credit card number" (placeholder alone is insufficient for screen readers)

[ACCESSIBILITY — WARNING] src/components/ProductCard.tsx:34
  Issue: Pressable is missing accessibilityRole
  Element: <Pressable onPress={() => navigate('ProductDetail', { id })}>
  Fix: Add accessibilityRole="button" and accessibilityLabel={`View ${productName}`}

[ACCESSIBILITY — WARNING] src/screens/HomeScreen.tsx:156
  Issue: FlatList items have no semantic grouping for screen readers
  Detail: Each list item renders multiple text nodes without Pressable-level labeling
  Fix: Add accessibilityLabel to the outermost Pressable of each list item,
       or use accessibilityElements grouping

[ACCESSIBILITY — WARNING] src/components/QuantitySelector.tsx:45
  Issue: Increment/decrement buttons missing accessibilityHint
  Element: <Pressable onPress={increment} accessibilityRole="button" accessibilityLabel="+">
  Fix: Add accessibilityHint="Increases quantity by one" for context

[ACCESSIBILITY — WARNING] src/screens/CartScreen.tsx:78
  Issue: Swipe-to-delete action has no accessible alternative
  Detail: Cart items can be swiped to delete, but no button or menu provides the same action
  Fix: Add a delete button or accessibilityCustomActions with a "Delete item" action

[ACCESSIBILITY — SUGGESTION] src/screens/OrderConfirmationScreen.tsx:34
  Issue: Success state announced only via visual icon change
  Detail: Checkmark icon appears but accessible announcement not triggered
  Fix: Use AccessibilityInfo.announceForAccessibility('Order confirmed!') on screen mount
```

---

## Touch Targets — 4 findings

> Interactive elements smaller than platform minimums (iOS: 44×44pt, Android: 48×48dp)

```
[TOUCH TARGET — CRITICAL] src/components/IconButton.tsx:23
  Issue: Touch target smaller than minimum
  Measured: 24×24pt (icon size only, no padding)
  Required: 44×44pt (Apple HIG) / 48×48dp (Material)
  Fix: Add padding: 12 to the Pressable style, or set minWidth: 44, minHeight: 44

[TOUCH TARGET — WARNING] src/components/Badge.tsx:8
  Issue: Badge is interactive (onPress prop) but small
  Measured: 28×20pt
  Fix: If badge is tappable, ensure minHeight: 44. If display-only, remove onPress.

[TOUCH TARGET — WARNING] src/components/QuantitySelector.tsx:22
  Issue: Increment button at 32×32pt
  Required: 44pt minimum
  Fix: Increase button padding or add minHeight: 44

[TOUCH TARGET — WARNING] src/screens/ProductDetailScreen.tsx:203
  Issue: "Add to wishlist" icon button at 36×36pt
  Fix: Add padding: 4 to reach 44×44pt, or use minWidth/minHeight
```

---

## Platform Convention Issues — 3 findings

> Code that doesn't follow React Native / iOS / Android platform conventions

```
[PLATFORM CONVENTION] src/components/ProductCard.tsx:45
  Issue: Shadow properties applied without platform check
  Code: style={{ shadowColor: '#000', shadowOffset: { width: 0, height: 2 }, elevation: 4 }}
  Note: iOS ignores elevation; Android ignores shadow* properties.
        Applying both is harmless but misleading.
  Fix: Use Platform.select to make intent explicit:
       ...Platform.select({
         ios: { shadowColor: '#000', shadowOffset: { width: 0, height: 2 }, shadowOpacity: 0.1, shadowRadius: 4 },
         android: { elevation: 4 },
       })

[PLATFORM CONVENTION] src/screens/HomeScreen.tsx:12
  Issue: Screen content not wrapped in SafeAreaView
  Detail: On iPhone 14 Pro and newer, content renders under the Dynamic Island
  Fix: Wrap root view with <SafeAreaView style={styles.container}> or use
       react-native-safe-area-context's <SafeAreaProvider> + useSafeAreaInsets()

[PLATFORM CONVENTION] src/screens/CheckoutScreen.tsx:5
  Issue: Screen with TextInput missing KeyboardAvoidingView
  Detail: On iOS, the keyboard will overlap input fields when focused
  Fix: Wrap form content in:
       <KeyboardAvoidingView behavior={Platform.OS === 'ios' ? 'padding' : 'height'}>
```

---

## Summary

| Category | Critical | Warning | Suggestion |
|----------|----------|---------|------------|
| Token drift | 0 | 11 | 0 |
| Accessibility | 3 | 4 | 1 |
| Touch targets | 1 | 3 | 0 |
| Platform conventions | 0 | 3 | 0 |
| **Total** | **4** | **21** | **1** |

---

## Prioritized Next Steps

1. **Fix the 3 critical accessibility issues first** — the icon button, product images, and card number input have no accessible labels. These block users who rely on VoiceOver or TalkBack entirely. Takes under 30 minutes to fix all three.

2. **Fix the small touch target on `IconButton`** — 24×24pt is half the Apple HIG minimum. Add `padding: 12` to the Pressable. This also affects every screen that uses `IconButton`.

3. **Wrap `HomeScreen` and `CheckoutScreen` in `SafeAreaView`** — without this, content is obscured by the Dynamic Island on newer iPhones. 5-minute fix per screen.

4. **Add `KeyboardAvoidingView` to `CheckoutScreen`** — checkout has multiple text inputs and is likely the highest-friction screen in the app. Keyboard overlap during payment entry is a conversion killer.

5. **Migrate token drift in `ProductCard` and `Badge`** — these are shared components rendered everywhere. Fixing their hardcoded values propagates improvements across the whole app.
