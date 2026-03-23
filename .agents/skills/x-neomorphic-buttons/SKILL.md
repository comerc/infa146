---
name: x-neomorphic-buttons
description: Use this skill when adding buttons, creating interactive controls, styling button appearance, configuring button states, or setting up press animations in the Korpu iOS project. Triggers whenever a button is mentioned, a tap target is created, or any UIControl subclass is needed. Enforces the correct button class hierarchy, prevents use of the deprecated NeomorphismButton, ensures styling goes through ButtonStyleProviding and not into ViewControllers, and ensures content layout uses IconTitleContentView. Apply when a user says "add a button", "create a control", "style a button", "configure button states", or "set up button press behavior".
---

# Neomorphic Button System

The Korpu project uses a layered neomorphic button hierarchy.

> **`NeomorphismButton` is DEPRECATED — never use it in new code.**

## Class Hierarchy

```
UIControl
└── BaseButton
    └── SurfaceButton
        └── OuterShadowButton
            └── DualShadowButton
                └── ConfigurableNeomorphicButton
                    ├── PlateNeomorphicButton
                    ├── KONeomorphicButton        ← default choice (90% of screens)
                    └── KOPlateNeomorphicButton   ← when inset press effect is needed
```

## Which Button to Choose

| Button | When to use |
|--------|-------------|
| `KONeomorphicButton` | Standard interactive button. Fixed `cornerRadius = 16`. Uses `NoPressResponder`. |
| `KOPlateNeomorphicButton` | Needs physical "pressed-in" depth effect. `plateBackgroundColor = Palette.Background.main`. |
| `ConfigurableNeomorphicButton` | Custom style via `ButtonStyleProviding` — only when KO variants don't fit. |
| `PlateNeomorphicButton` | Low-level base class — extend only when building a new plate variant. |

Prefer `KONeomorphicButton` or `KOPlateNeomorphicButton` for 90% of product screens.

## ConfigurableNeomorphicButton — Key Properties

The **main reusable button component**. Buttons do not know colors, fonts, or shadows — all visuals come from `ButtonStyleProviding`.

Key properties:
- `styleProvider: ButtonStyleProviding`
- `runtimeShadowModeOverride: ShadowMode?`
- `contentContainerView`
- `iconTitleView: IconTitleContentView`

## PlateNeomorphicButton

Extends `ConfigurableNeomorphicButton` with a **pressed-in (plate) effect**:
- separate `plateContainer` layer placed in `superview`
- inset press animation (visual depth)
- uses `PlateNeoInsetPressResponder`

Use **only when a physical inset effect is required**.

## Core Principle

**Buttons do not know colors, fonts, or shadows.** All visual configuration comes exclusively from a `ButtonStyleProviding` implementation. ViewControllers and Views must not set button appearance directly.

## Layer Responsibilities

| Class | Responsibility |
|-------|---------------|
| `BaseButton` | `isHighlighted`, `isEnabled`, `isSelected`; calls `ButtonPressResponder` on press; provides `stateDidChange()` hook |
| `SurfaceButton` | Background fill (`ButtonFill`: solid or gradient); `cornerRadius` |
| `OuterShadowButton` | Outer shadows (`OuterShadowStyle`); gradient borders (`ButtonBorderStyle`) |
| `DualShadowButton` | Inner shadows (`InnerShadowStyle`); active shadow mode via `ShadowMode` |
| `ConfigurableNeomorphicButton` | Connects to `ButtonStyleProviding`; owns `iconTitleView` |

## ButtonVisualStyle

A single immutable struct describing visual appearance for one control state. Contains:
- background fill (`ButtonFill` — solid or gradient)
- border (`ButtonBorderStyle`)
- shadow mode (`ShadowMode`)
- outer shadows (`OuterShadowStyle`)
- inner shadows (`InnerShadowStyle`)
- text style (gradient or solid, font)
- image tint
- alpha

Never partially override a style — always supply a complete `ButtonVisualStyle`.

## ButtonStyleProviding

```swift
protocol ButtonStyleProviding {
    func style(for state: UIControl.State) -> ButtonVisualStyle
}
```

State resolution order inside the button: `disabled → highlighted → selected → normal`

Any state change triggers `updateAppearanceForCurrentState()` automatically.

## DefaultButtonStyleProvider

- `.normal` → base style
- `.highlighted` → same style with reduced alpha
- `.disabled` → reduced alpha, no text gradients

## Product Style Providers

Ready-to-use providers for standard Korpu designs:
- `KorpuPlatePrimaryStyleProvider`
- `KorpuPlateSecondaryStyleProvider`
- `KorpuGhostStyleProvider`

All product-specific colors, shadows, and fonts live **inside** these providers. Never put them in a ViewController.

When a new visual design is needed, create a new `StyleProvider` — do not subclass the button itself.

## Press Responders

Press responders handle **animations only**. They must never modify control state.

| Responder | Behavior |
|-----------|----------|
| `NoPressResponder` | No animation (used by `KONeomorphicButton`) |
| `DefaultPressResponder` | Light scale / alpha change |
| `PlateNeoInsetPressResponder` | Plate + inset depth animation (used inside plate variants) |

## IconTitleContentView

Reusable container for button content. Buttons must not manage icon/title layout directly — use `iconTitleView`.

Supported layouts:
- text only
- icon only
- icon left + text right
- icon right + text left
- icon top + text bottom

```swift
button.iconTitleView.configure(
    title: "Continue",
    icon: UIImage(systemName: "arrow.right"),
    layout: .iconRight
)
```

## Usage Examples

```swift
// Standard button — preferred
let button = KONeomorphicButton()
button.styleProvider = KorpuPlatePrimaryStyleProvider()
button.iconTitleView.configure(title: "Sign In", layout: .textOnly)

// Plate button — with inset press effect
let plateButton = KOPlateNeomorphicButton()
plateButton.styleProvider = KorpuPlateSecondaryStyleProvider()
plateButton.iconTitleView.configure(title: "Continue", icon: someIcon, layout: .iconLeft)

// Custom styling — only when KO variants don't cover the design
let custom = ConfigurableNeomorphicButton()
custom.styleProvider = MyCustomStyleProvider()
```

## Rules

### DO
- Default to `KONeomorphicButton` or `KOPlateNeomorphicButton`
- Put all colors, fonts, and shadows in a `ButtonStyleProviding` implementation
- Use `IconTitleContentView` (`button.iconTitleView`) for all icon + title layout
- Create a new `StyleProvider` for each distinct visual design

### DON'T
- Use `NeomorphismButton` anywhere in new code ❌
- Set colors, fonts, or shadow properties on a button in a ViewController ❌
- Manage icon/title layout directly on the button — use `iconTitleView` ❌
- Modify control state from within a press responder ❌
- Override `cornerRadius` on `KONeomorphicButton` externally (it is fixed at 16) ❌
- Partially configure a `ButtonVisualStyle` — always supply all fields ❌
