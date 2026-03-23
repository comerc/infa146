---
name: x-neomorphic-views
description: Use this skill when creating non-interactive UI containers, cards, panels, background surfaces, or any non-button visual container in the Korpu iOS project. Triggers when a user asks for a "card", "panel", "container", "surface", "background view", or any layout wrapper that is not a button. Also triggers when adding text fields, form inputs, or bottom sheets. Ensures correct use of KONeomorphicView hierarchy, NeomorphicViewStyle, style providers, and approved text field and bottom sheet components.
---

# Neomorphic View System

For non-button containers: cards, panels, backgrounds, surfaces.

## Class Hierarchy

```
UIView
└── BaseRoundedView
    └── SurfaceView
        └── BorderedRoundedView
            └── OuterShadowView
                └── DualShadowView
                    └── NeomorphicView
                        └── KONeomorphicView   ← default for all product containers
```

## Default Choice

Use `KONeomorphicView` for all product-level cards, panels, and containers.

Only go lower in the hierarchy when building a new reusable container component that needs a specific subset of neomorphic capabilities.

## Styling Architecture

### NeomorphicViewStyle

An immutable style struct describing visual appearance. All fills, shadows, borders, and corner radii are defined here — never set directly on the view in a ViewController.

### NeomorphicViewStyleProvider

Delegates style selection to the provider. The view requests configuration through the provider and applies it. Views do not know colors, shadows, or fills directly.

## Usage Pattern

```swift
// Standard container card
let cardView = KONeomorphicView()
cardView.styleProvider = SomeNeomorphicStyleProvider()

// Add content inside the view's contentView
cardView.contentView.addSubview(titleLabel)
cardView.contentView.addSubview(bodyLabel)
```

All `Palette.*` color references go inside the `StyleProvider` implementation, not in the ViewController.

## Text Fields

Use only project-approved components — do not create custom UITextField subclasses:

| Component | Use case |
|-----------|----------|
| `KONeomorphicTextField` | Standard neomorphic input field |
| `KOFilledTextField` | Filled / solid background variant |

Configure via:
- `TextFieldStyleProvider` — visual styling (colors, borders, shadows)
- `TextFieldValidator` — validation logic and error states

Do not add custom borders, shadow layers, or background colors directly to text fields in screen code.

## Bottom Sheets

Standard components for all bottom sheet needs:

| Component | Role |
|-----------|------|
| `BaseBottomSheetViewController` | Base class for custom bottom sheets |
| `BottomSheetContentView` | Content wrapper with standard insets and sizing |
| `BottomSheetListViewController` | Pre-built list/selection sheet |

Built-in features (do not reimplement these):
- Automatic height calculation based on content
- Drag to dismiss and tap backdrop to dismiss
- `UITableView` / `UICollectionView` support
- Single or multiple selection modes
- Customizable action buttons

## Rules

### DO
- Default to `KONeomorphicView` for all product cards and containers
- Configure all visual appearance through `NeomorphicViewStyle` via `NeomorphicViewStyleProvider`
- Reference `Palette.*` colors inside style providers, not in ViewControllers
- Use `KONeomorphicTextField` or `KOFilledTextField` for all form inputs
- Use the standard bottom sheet components for all sheet presentations

### DON'T
- Hardcode colors, shadows, corner radii, or borders directly on any view in a ViewController ❌
- Create plain `UIView` subclasses with manually set shadow/border properties when `KONeomorphicView` fits ❌
- Mix neomorphic views with manually styled plain UIViews in the same screen ❌
- Add custom shadow layers or border layers to `KONeomorphicTextField` or `KOFilledTextField` ❌
- Re-implement drag-to-dismiss, height calculation, or selection logic — use the provided bottom sheet components ❌
