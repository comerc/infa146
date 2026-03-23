---
name: x-palette-manager
description: Use this skill when working with colors, images, shadows, or any visual styling in the Korpu iOS project. Triggers whenever the user wants to add a new color, change a background, update an image asset, configure shadows, modify any visual appearance, or add component styling. This skill MUST be used for ALL palette and asset catalog changes across both the Korpu and Khidi build targets. Apply it even if the user mentions only one target — always verify whether both need updating. Also triggers when refactoring #if KHIDI / #if KORPU conditional compilation blocks into Palette references.
---

# Palette & Asset Catalog Manager

This skill governs all visual asset and palette management for the Korpu/Khidi multi-target iOS project.

## Target Detection

Determine scope before making any changes:

| Signal | Target | Action |
|--------|--------|--------|
| User says "Korpu only" | Korpu | Update only Korpu |
| User says "Khidi only" | Khidi | Update only Khidi |
| No target specified | **Both** | Update Korpu AND Khidi |
| File path contains `/Korpu/` | Korpu | Korpu context |
| File path contains `/Khidi/` | Khidi | Khidi context |
| Asset name has `khidi` prefix | Khidi | Khidi-specific |
| Asset name has no prefix | Korpu | Korpu-specific |
| User shares `#if KHIDI` / `#if KORPU` code | Both | Refactor to Palette for both |
| Module in `KorpuModules/` | Korpu | Korpu-specific feature |
| Module in `KhidiModules/` | Khidi | Khidi-specific feature |

**Default: work with BOTH targets simultaneously.**

Always state upfront which targets are being updated. Warn explicitly when modifying only one.

When analyzing existing code, use asset references to identify the target:

```swift
// Clearly Khidi-specific
UIColor(resource: .khidiBackgroundMain)
UIImage(resource: .khidiCheckboxOff)

// Clearly Korpu-specific
UIColor(resource: .backgroundMain)
UIImage(resource: .checkboxOff)

// Target-agnostic (uses Palette) ✅
Palette.Card.backgroundColor
```

## Asset Catalog Structure

```
Korpu/
├── KorpuAssets.xcassets/          // Korpu target
│   ├── Colors/
│   │   ├── backgroundMain.colorset
│   │   └── ...
│   └── Images/
│       └── checkboxOff.imageset
└── KhidiAssets.xcassets/          // Khidi target
    ├── Colors/
    │   ├── khidiBackgroundMain.colorset
    │   └── ...
    └── Images/
        └── khidiCheckboxOff.imageset
```

## Palette File Locations

```
Korpu/Korpu/Palette.swift   → Korpu target
Korpu/Khidi/Palette.swift   → Khidi target
```

## Color Naming Conventions

- **Korpu**: descriptive name, no prefix — `backgroundMain`, `textSecondary`, `buttonPrimary`, `borderLight`
- **Khidi**: `khidi` prefix — `khidiBackgroundMain`, `khidiTextSecondary`, `khidiButtonPrimary`, `khidiBorderLight`
- **Enum and property names in Palette.swift**: IDENTICAL across both targets (same enum names, same static property names)

## Complete Workflow for Adding New Styling

### Step 1: Add to Asset Catalogs

Add to **both** asset catalogs before referencing them in Palette:
- `KorpuAssets.xcassets`: e.g. `cardBackground`, `cardBorder`
- `KhidiAssets.xcassets`: e.g. `khidiCardBackground`, `khidiCardBorder`

For images: provide @1x, @2x, @3x PNG. Use PNG (not SVG) for assets with shadows or complex visual effects.

Color appearance variants:
- **Any Appearance** — default (light mode)
- **Dark Appearance** — add if dark mode is supported

### Step 2: Update Both Palette Files

Enum structure must be identical. Only the asset references differ:

```swift
// Korpu/Palette.swift
// MARK: - Card Styling
public enum Card {
    // Card background
    public static let backgroundColor = UIColor(resource: .cardBackground)

    // Card border
    public static let borderColor = UIColor(resource: .cardBorder)
    public static let borderWidth: CGFloat = 1

    // Card shadow
    public static let shadowColor = UIColor.black.withAlphaComponent(0.1)
    public static let shadowOffset = CGSize(width: 0, height: 2)
    public static let shadowRadius: CGFloat = 8
    public static let shadowOpacity: Float = 0.2

    // Card corner radius
    public static let cornerRadius: CGFloat = 12

    // Card icon (if needed)
    public static let icon = UIImage(resource: .cardIcon)
}
```

```swift
// Khidi/Palette.swift — identical enum/property names, different assets
// MARK: - Card Styling
public enum Card {
    // Card background
    public static let backgroundColor = UIColor(resource: .khidiCardBackground)

    // Card border
    public static let borderColor = UIColor(resource: .khidiCardBorder)
    public static let borderWidth: CGFloat = 2

    // Card shadow
    public static let shadowColor = UIColor.black.withAlphaComponent(0.15)
    public static let shadowOffset = CGSize(width: 0, height: 4)
    public static let shadowRadius: CGFloat = 12
    public static let shadowOpacity: Float = 0.3

    // Card corner radius
    public static let cornerRadius: CGFloat = 16

    // Card icon (if needed)
    public static let icon = UIImage(resource: .khidiCardIcon)
}
```

### Step 3: Use in UI Code (Target-Agnostic)

```swift
// Works automatically for both targets — no #if blocks needed
view.backgroundColor = Palette.Card.backgroundColor
layer.shadowColor = Palette.Card.shadowColor.cgColor
layer.shadowOffset = Palette.Card.shadowOffset
layer.shadowRadius = Palette.Card.shadowRadius
layer.shadowOpacity = Palette.Card.shadowOpacity
layer.borderColor = Palette.Card.borderColor.cgColor
layer.borderWidth = Palette.Card.borderWidth
layer.cornerRadius = Palette.Card.cornerRadius
imageView.image = Palette.Card.icon
```

## Complete Example: Adding Card Component

**1. Add colors to KorpuAssets.xcassets:**
- Create `cardBackground` color set → #FFFFFF
- Create `cardBorder` color set → #E0E0E0

**2. Add colors to KhidiAssets.xcassets:**
- Create `khidiCardBackground` color set → #F5F5F5
- Create `khidiCardBorder` color set → #D0D0D0

**3. Add images to KorpuAssets.xcassets (if needed):**
- Add `cardIcon.imageset` with PNG files (@1x, @2x, @3x)

**4. Add images to KhidiAssets.xcassets (if needed):**
- Add `khidiCardIcon.imageset` with PNG files (@1x, @2x, @3x)

**5–6. Update both Palette.swift files** (see Step 2 above)

**7. Use in CardView.swift:**

```swift
final class CardView: UIView {
    private let iconImageView = UIImageView()

    override init(frame: CGRect) {
        super.init(frame: frame)
        setupStyling()
    }

    private func setupStyling() {
        // All styling from Palette - no #if blocks, no hardcoded values
        backgroundColor = Palette.Card.backgroundColor

        layer.shadowColor = Palette.Card.shadowColor.cgColor
        layer.shadowOffset = Palette.Card.shadowOffset
        layer.shadowRadius = Palette.Card.shadowRadius
        layer.shadowOpacity = Palette.Card.shadowOpacity

        layer.borderColor = Palette.Card.borderColor.cgColor
        layer.borderWidth = Palette.Card.borderWidth

        layer.cornerRadius = Palette.Card.cornerRadius

        iconImageView.image = Palette.Card.icon
    }
}
```

## Gradient Pattern

```swift
// Korpu/Palette.swift
public enum Gradient {
    public static let gradientTitleStart = UIColor(resource: .gradientTop)
    public static let gradientTitleEnd = UIColor(resource: .gradientBottom)
}

// Khidi/Palette.swift
public enum Gradient {
    public static let gradientTitleStart = UIColor(resource: .gradientTitleStart)
    public static let gradientTitleEnd = UIColor(resource: .gradientTitleEnd)
}

// Usage in a view:
let colors: [UIColor] = [Palette.Gradient.gradientTitleStart, Palette.Gradient.gradientTitleEnd]
button.fill = .verticalGradient(colors.map { $0 })
```

## Image + Shadow Pattern (e.g. Checkbox)

```swift
// Korpu/Palette.swift
public enum Checkbox {
    // Images
    public static let imageOff = UIImage(resource: .checkboxOff)
    public static let imageOn = UIImage(resource: .checkboxOn)

    // Shadow
    public static let shadowColor = UIColor.black.withAlphaComponent(0.15)
    public static let shadowOffset = CGSize(width: 0, height: 2)
    public static let shadowRadius: CGFloat = 4
    public static let shadowOpacity: Float = 0.3
}

// Khidi/Palette.swift
public enum Checkbox {
    // Images
    public static let imageOff = UIImage(resource: .khidiCheckboxOff)
    public static let imageOn = UIImage(resource: .khidiCheckboxOn)

    // Shadow
    public static let shadowColor = UIColor.black.withAlphaComponent(0.25)
    public static let shadowOffset = CGSize(width: 0, height: 3)
    public static let shadowRadius: CGFloat = 6
    public static let shadowOpacity: Float = 0.4
}

// Usage:
button.setImage(Palette.Checkbox.imageOff, for: .normal)
button.setImage(Palette.Checkbox.imageOn, for: .selected)
button.imageView?.layer.shadowColor = Palette.Checkbox.shadowColor.cgColor
button.imageView?.layer.shadowOffset = Palette.Checkbox.shadowOffset
button.imageView?.layer.shadowRadius = Palette.Checkbox.shadowRadius
button.imageView?.layer.shadowOpacity = Palette.Checkbox.shadowOpacity
button.imageView?.layer.masksToBounds = false
```

## Button Fill Pattern

```swift
// Korpu/Palette.swift
public enum ButtonStyling {
    // Plate buttons setup - always comment what the config relates to
    public static let plateButtonFill = UIColor(resource: .backgroundMain)
    public static let plateButtonTextColor = UIColor(resource: .textSecondary)
}

// Khidi/Palette.swift
public enum ButtonStyling {
    // Plate buttons setup
    public static let plateButtonFill = UIColor(resource: .khidiBackgroundMain)
    public static let plateButtonTextColor = UIColor(resource: .khidiPrimaryLight)
}

// Usage in code — automatically adapts to each target:
button.fill = .solid(Palette.ButtonStyling.plateButtonFill)
```

## Refactoring #if Blocks

When you encounter conditional compilation for styling:

```swift
// ❌ Before — to be refactored
#if KHIDI
let image = UIImage(resource: .khidiCheckboxOff)
#else
let image = UIImage(resource: .checkboxOff)
#endif
```

Steps:
1. Add the appropriate entry to both `Palette.swift` files
2. Replace the `#if` block with `Palette.Checkbox.imageOff`

## Rules

### DO
- Add colors/images to **both** asset catalogs before referencing them in Palette
- Use identical enum and property names across both `Palette.swift` files
- Reference only `Palette.*` in ViewControllers and Views
- Configure color appearance variants when needed: **Any Appearance** (default) and **Dark Appearance** (if supporting dark mode)
- Create identical enum structures in both palettes with the same naming:
  - Same enum names (e.g., `Checkbox`, `ButtonStyling`, `Card`)
  - Same static property names (e.g., `imageOff`, `shadowColor`, `backgroundColor`)
- Add comprehensive comments to each section indicating what component the configuration relates to
- Group related configurations together in the same enum:
  ```swift
  public enum Checkbox {
      // Images
      public static let imageOff = ...
      public static let imageOn = ...
      // Shadows
      public static let shadowColor = ...
      public static let shadowOffset = ...
      // Colors (if needed)
      public static let tintColor = ...
  }
  ```
- Include shadow config (color, offset, radius, opacity) whenever components use shadows
- Use PNG for images with shadows or visual effects

### DON'T

1. **Never reference colors that don't exist in Asset Catalogs:**
   ```swift
   // ❌ WRONG - color doesn't exist in xcassets
   public static let buttonColor = UIColor(resource: .nonExistentColor)
   ```

2. **Never create local enums in ViewControllers or Views** for styling configuration:
   ```swift
   // ❌ WRONG - local enum in VC
   enum CheckboxImages {
       static let off = UIImage(resource: .checkboxOff)
   }
   ```

3. **Never use `#if Target` blocks** in ViewControllers or Views for styling:
   ```swift
   // ❌ WRONG - conditional compilation for styling
   #if KHIDI
   let image = UIImage(resource: .khidiCheckboxOff)
   #else
   let image = UIImage(resource: .checkboxOff)
   #endif
   ```

4. **Never hardcode colors, images, or shadow values** directly in UI code:
   ```swift
   // ❌ WRONG - hardcoded values
   view.backgroundColor = UIColor(red: 0.2, green: 0.3, blue: 0.4, alpha: 1.0)
   layer.shadowColor = UIColor.black.cgColor
   layer.shadowRadius = 4
   ```

5. **Never use different property names** across target palettes for the same logical property:
   ```swift
   // ❌ WRONG - different naming
   // Korpu: public static let checkboxImageOff
   // Khidi: public static let imageOff
   ```

6. **Never skip adding colors to one of the Asset Catalogs** — both targets must have their colors defined.
