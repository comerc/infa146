---
name: x-typography-manager
description: Use this skill whenever working with fonts in the Korpu iOS project. Triggers when the user wants to set a font, configure a label, style button text, add a new font weight, update text appearance, or modify Typography.swift. Must be applied for ALL font-related changes across both Korpu and Khidi targets. Prevents hardcoded font names, wrong method signatures, optional return types, and fixed-size font constants. Also triggers when a user adds a new label, text field, or text-bearing UI element.
---

# Typography Manager

This skill governs all font configuration for the Korpu/Khidi multi-target iOS project.

## File Locations

```
Korpu/Korpu/Typography.swift   → Korpu target
Korpu/Khidi/Typography.swift   → Khidi target
```

## Architecture

Typography uses a **method-based approach**: each font weight is a `static func` that accepts a `CGFloat` size. This avoids fixed-size constants and allows the same weight to be used at any size throughout the app.

```swift
public enum Typography {
    /// Primary font family — main UI text, buttons, labels, navigation
    public enum Primary { ... }

    /// Secondary font family — decorative elements, quotes, special headings
    public enum Secondary { ... }
}
```

Both targets must define the same enum and method names. Only the underlying font names differ.

## Korpu Typography.swift

```swift
import UIKit

public enum Typography {

    /// Manrope — Primary font family for Korpu
    public enum Primary {

        public static func regular(_ size: CGFloat) -> UIFont {
            UIFont(name: "Manrope", size: size)
                ?? .systemFont(ofSize: size, weight: .regular)
        }

        public static func medium(_ size: CGFloat) -> UIFont {
            UIFont(name: "Manrope-Medium", size: size)
                ?? .systemFont(ofSize: size, weight: .medium)
        }

        public static func semibold(_ size: CGFloat) -> UIFont {
            UIFont(name: "Manrope-SemiBold", size: size)
                ?? .systemFont(ofSize: size, weight: .semibold)
        }

        public static func bold(_ size: CGFloat) -> UIFont {
            UIFont(name: "Manrope-Bold", size: size)
                ?? .systemFont(ofSize: size, weight: .bold)
        }

        public static func extraLight(_ size: CGFloat) -> UIFont {
            UIFont(name: "Manrope-ExtraLight", size: size)
                ?? .systemFont(ofSize: size, weight: .ultraLight)
        }

        public static func light(_ size: CGFloat) -> UIFont {
            UIFont(name: "Manrope-Light", size: size)
                ?? .systemFont(ofSize: size, weight: .light)
        }

        public static func extraBold(_ size: CGFloat) -> UIFont {
            UIFont(name: "Manrope-ExtraBold", size: size)
                ?? .systemFont(ofSize: size, weight: .heavy)
        }
    }

    /// Montserrat — Secondary font family for Korpu
    public enum Secondary {

        public static func regular(_ size: CGFloat) -> UIFont {
            UIFont(name: "Montserrat-Regular", size: size)
                ?? .systemFont(ofSize: size, weight: .regular)
        }

        public static func medium(_ size: CGFloat) -> UIFont {
            UIFont(name: "Montserrat-Medium", size: size)
                ?? .systemFont(ofSize: size, weight: .medium)
        }

        public static func semibold(_ size: CGFloat) -> UIFont {
            UIFont(name: "Montserrat-SemiBold", size: size)
                ?? .systemFont(ofSize: size, weight: .semibold)
        }

        public static func bold(_ size: CGFloat) -> UIFont {
            UIFont(name: "Montserrat-Bold", size: size)
                ?? .systemFont(ofSize: size, weight: .bold)
        }

        public static func extraBold(_ size: CGFloat) -> UIFont {
            UIFont(name: "Montserrat-ExtraBold", size: size)
                ?? .systemFont(ofSize: size, weight: .heavy)
        }

        public static func extraBoldItalic(_ size: CGFloat) -> UIFont {
            UIFont(name: "Montserrat-ExtraBoldItalic", size: size)
                ?? .systemFont(ofSize: size, weight: .heavy)
        }
    }
}
```

## Khidi Typography.swift

```swift
import UIKit

public enum Typography {

    /// SF Pro — Primary font family for Khidi (system font)
    public enum Primary {

        public static func regular(_ size: CGFloat) -> UIFont {
            .systemFont(ofSize: size, weight: .regular)
        }

        public static func medium(_ size: CGFloat) -> UIFont {
            .systemFont(ofSize: size, weight: .medium)
        }

        public static func semibold(_ size: CGFloat) -> UIFont {
            .systemFont(ofSize: size, weight: .semibold)
        }

        public static func bold(_ size: CGFloat) -> UIFont {
            .systemFont(ofSize: size, weight: .bold)
        }

        public static func extraLight(_ size: CGFloat) -> UIFont {
            .systemFont(ofSize: size, weight: .ultraLight)
        }

        public static func light(_ size: CGFloat) -> UIFont {
            .systemFont(ofSize: size, weight: .light)
        }

        public static func extraBold(_ size: CGFloat) -> UIFont {
            .systemFont(ofSize: size, weight: .heavy)
        }
    }

    /// Georgia — Secondary font family for Khidi
    public enum Secondary {

        public static func regular(_ size: CGFloat) -> UIFont {
            UIFont(name: "Georgia", size: size)
                ?? .systemFont(ofSize: size, weight: .regular)
        }

        public static func medium(_ size: CGFloat) -> UIFont {
            UIFont(name: "Georgia", size: size)
                ?? .systemFont(ofSize: size, weight: .medium)
        }

        public static func semibold(_ size: CGFloat) -> UIFont {
            UIFont(name: "Georgia-Bold", size: size)
                ?? .systemFont(ofSize: size, weight: .semibold)
        }

        public static func bold(_ size: CGFloat) -> UIFont {
            UIFont(name: "Georgia-Bold", size: size)
                ?? .systemFont(ofSize: size, weight: .bold)
        }

        public static func italic(_ size: CGFloat) -> UIFont {
            UIFont(name: "Georgia-Italic", size: size)
                ?? .systemFont(ofSize: size, weight: .regular)
        }
    }
}
```

## Usage in UI Code

**Basic Usage:**

```swift
final class ProfileViewController: UIViewController {
    private let titleLabel: UILabel = {
        let label = UILabel()
        // Use Primary font family with size 28 for title
        label.font = Typography.Primary.bold(28)
        label.textColor = Palette.Text.primary
        return label
    }()

    private let subtitleLabel: UILabel = {
        let label = UILabel()
        // Use Primary font family with size 18 for subtitle
        label.font = Typography.Primary.semibold(18)
        label.textColor = Palette.Text.secondary
        return label
    }()

    private let bodyLabel: UILabel = {
        let label = UILabel()
        // Use Primary font family with size 16 for body text
        label.font = Typography.Primary.regular(16)
        label.textColor = Palette.Text.tertiary
        return label
    }()

    private let quoteLabel: UILabel = {
        let label = UILabel()
        // Use Secondary font family for decorative quote
        label.font = Typography.Secondary.italic(18)
        label.textColor = Palette.Text.secondary
        return label
    }()
}
```

**Usage with Buttons:**

```swift
private lazy var continueButton: UIButton = {
    let button = UIButton(type: .system)
    button.setTitle("Continue", for: .normal)
    button.titleLabel?.font = Typography.Primary.semibold(16)
    button.setTitleColor(Palette.Button.textColor, for: .normal)
    return button
}()
```

**Usage in Custom Views:**

```swift
final class CardView: UIView {
    private let titleLabel = UILabel()
    private let descriptionLabel = UILabel()

    private func setupFonts() {
        titleLabel.font = Typography.Primary.bold(20)
        descriptionLabel.font = Typography.Primary.regular(14)
    }
}
```

## Recommended Size Ranges

| Use case | Size | Weight |
|----------|------|--------|
| Large titles | 28–32pt | `bold` / `extraBold` |
| Section headers | 20–24pt | `semibold` / `bold` |
| Body text | 14–16pt | `regular` / `medium` |
| Captions | 12–13pt | `regular` |
| Small labels | 10–11pt | `light` / `regular` |

## When to Use Primary vs Secondary

- **Primary**: Standard UI elements — buttons, labels, text fields, navigation titles
- **Secondary**: Decorative or editorial content — quotes, callouts, special headings, marketing copy

When working with existing code, continue using whichever family is already in use for that element.

## System Font Weight Mapping (for fallbacks)

| Method | `UIFont.Weight` fallback |
|--------|--------------------------|
| `extraLight` | `.ultraLight` |
| `light` | `.light` |
| `regular` | `.regular` |
| `medium` | `.medium` |
| `semibold` | `.semibold` |
| `bold` | `.bold` |
| `extraBold` | `.heavy` |

## Rules

### DO

1. **Always use Typography methods with size parameter** when setting fonts:
   ```swift
   label.font = Typography.Primary.bold(24)
   button.titleLabel?.font = Typography.Primary.medium(16)
   textView.font = Typography.Secondary.regular(14)
   ```

2. **Maintain identical method signatures** across both Typography files:
   - Same enum names (`Primary`, `Secondary`)
   - Same method names (`regular()`, `bold()`, `semibold()`, etc.)
   - Same parameter structure (single `CGFloat` parameter for size)

3. **Always provide fallback fonts** when using custom font names:
   ```swift
   public static func bold(_ size: CGFloat) -> UIFont {
       UIFont(name: "CustomFont-Bold", size: size)
       ?? .systemFont(ofSize: size, weight: .bold)
   }
   ```

4. **Match system font weights** to custom font weights in fallbacks — see System Font Weight Mapping table above.

5. **Add descriptive comments** to Typography enums:
   ```swift
   /// Manrope - Primary font family for Korpu
   /// Used for main UI text, buttons, and standard content
   public enum Primary { ... }

   /// Montserrat - Secondary font family for Korpu
   /// Used for decorative elements, quotes, and special headings
   public enum Secondary { ... }
   ```

6. **Choose appropriate font family** based on use case:
   - **Primary**: Standard UI elements (buttons, labels, text fields, navigation)
   - **Secondary**: Decorative elements (quotes, callouts, special headings, marketing content)

7. **When working with existing code**, keep using whichever family is already in use:
   ```swift
   // If you see Typography.Primary — keep using Primary
   label.font = Typography.Primary.bold(20)

   // If you see Typography.Secondary — keep using Secondary
   label.font = Typography.Secondary.regular(18)
   ```

8. **Recommend appropriate sizes** based on typical UI patterns — see Recommended Size Ranges table above.

### DON'T

1. **Never hardcode font names or sizes** in ViewControllers or Views:
   ```swift
   // ❌ WRONG - hardcoded font
   label.font = UIFont(name: "Manrope-Bold", size: 20)
   label.font = UIFont.systemFont(ofSize: 16, weight: .medium)
   ```

2. **Never use different method signatures** across Typography files:
   ```swift
   // ❌ WRONG - inconsistent signatures
   // Korpu: public static func bold(_ size: CGFloat) -> UIFont
   // Khidi: public static func boldFont(size: CGFloat) -> UIFont
   ```

3. **Never return nil or optional fonts** from Typography methods:
   ```swift
   // ❌ WRONG - optional return type
   public static func bold(_ size: CGFloat) -> UIFont? {
       return UIFont(name: "Manrope-Bold", size: size)
   }

   // ✅ CORRECT - always return non-optional with fallback
   public static func bold(_ size: CGFloat) -> UIFont {
       return UIFont(name: "Manrope-Bold", size: size)
           ?? .systemFont(ofSize: size, weight: .bold)
   }
   ```

4. **Never create font constants** with fixed sizes in Typography:
   ```swift
   // ❌ WRONG - fixed size constant
   public static let titleFont = UIFont(name: "Manrope-Bold", size: 24)

   // ✅ CORRECT - flexible method with size parameter
   public static func bold(_ size: CGFloat) -> UIFont {
       UIFont(name: "Manrope-Bold", size: size) ?? .systemFont(ofSize: size, weight: .bold)
   }
   ```

5. **Never skip font families** that exist in one Typography but not the other — maintain parity between targets.

6. **Never use custom font names** directly in UI code:
   ```swift
   // ❌ WRONG
   label.font = UIFont(name: "Manrope-SemiBold", size: 18)

   // ✅ CORRECT
   label.font = Typography.Primary.semibold(18)
   ```
