# Repository Guidelines

## Project Structure & Module Organization

iOS app lives in `Korpu/`; entry point is `Korpu/App/MainModule.swift`.

Minimal deployment version of iOS is 18.0.

Feature code is grouped in `Korpu/Modules` with a template trio (`KOModule`, `KOPresenter`, `KOViewController`) and feature folders like `Main` for routers and screens.

Assets sit in `Korpu/KorpuAssets.xcassets`; localization resources are under `Korpu/Base.lproj`.

Prefer creating new features as modules with their own router, presenter, and view/controller types to keep navigation and UI isolated.

---

## Multi-Target Architecture & Theme Management

### Core Concepts

- For each target, configure colors in assets and add them to the palette to achieve 70-80% (or more) automatic recoloring of the app.
- Store colors, shadows, images, and other theme configurations in the palette and configure them per component.
- Strive for complete color replacement throughout the app and components based on the target's color palette.
- **Minimize usage of `#if Target` blocks** to avoid code confusion when managing multiple targets.
- Maintain unique Assets for each project target.

### Target Detection and Working Rules

#### How Agent Determines Which Target to Work With

**1. Default Behavior: Work with BOTH Targets**

By default, when adding or modifying styling configurations, the agent should **ALWAYS work with BOTH targets simultaneously**:
- Add colors to both `KorpuAssets.xcassets` AND `KhidiAssets.xcassets`
- Update both `Korpu/Palette.swift` AND `Khidi/Palette.swift`
- Maintain identical structure and naming across both

**2. Explicit User Instructions**

User can explicitly specify which target to work with:
- "Add this color only to Korpu target"
- "Update Khidi palette only"
- "This feature is Khidi-specific"
- "This is for Korpu only"

**3. Context Analysis**

Agent should analyze the conversation context:
- If user mentions specific target name (Korpu/Khidi), focus on that target
- If user shares code with `#if KHIDI` or `#if KORPU`, understand the context
- If working on target-specific modules (KorpuModules/KhidiModules folders), adjust accordingly

#### Agent Response Patterns

**Scenario 1: User asks to add new component styling (no target specified)**

```
User: "Add styling for a new ProfileCard component"

Agent Response:
"I'll add styling for ProfileCard to both Korpu and Khidi targets.

Step 1: Add colors to Asset Catalogs

For KorpuAssets.xcassets:
- Create profileCardBackground color set
- Create profileCardBorder color set

For KhidiAssets.xcassets:
- Create khidiProfileCardBackground color set
- Create khidiProfileCardBorder color set

Step 2: Update Palette files
[provides code for both palettes]

Step 3: Usage in code
[target-agnostic code using Palette.*]
"
```

**Scenario 2: User specifies one target**

```
User: "Add a gradient for the header, but only for Khidi target"

Agent Response:
"I'll add the gradient configuration only for the Khidi target.

For KhidiAssets.xcassets:
- Create khidiHeaderGradientStart color set
- Create khidiHeaderGradientEnd color set

Khidi/Palette.swift:
[code for Khidi palette only]

Note: This is Khidi-specific. If you need this for Korpu later, let me know.
"
```

**Scenario 3: User asks about existing code with #if blocks**

```
User: "Here's my code with #if KHIDI, how do I refactor it?"

Agent Response:
"I see you're using conditional compilation. Let me refactor this to use the Palette pattern.

Step 1: Add to Korpu/Palette.swift
Step 2: Add to Khidi/Palette.swift
Step 3: Updated component code (no #if blocks)

This eliminates the #if blocks and makes the code cleaner.
"
```

#### Agent Checklist Before Responding

For EVERY styling-related request, agent should:

1. ✅ **Identify target scope:**
   - Both targets (default)
   - Specific target (if explicitly mentioned)
   - Current target context (from file paths or code analysis)

2. ✅ **Verify Asset Catalog updates needed:**
   - KorpuAssets.xcassets changes?
   - KhidiAssets.xcassets changes?
   - New color sets needed?
   - New image sets needed?

3. ✅ **Verify Palette updates needed:**
   - Korpu/Palette.swift changes?
   - Khidi/Palette.swift changes?
   - Naming consistency across both?
   - Comments added for clarity?

4. ✅ **Provide target-agnostic code:**
   - Uses `Palette.*` references
   - No `#if Target` blocks
   - Works automatically for both targets

5. ✅ **Explicit communication:**
   - Clearly state which targets are being updated
   - Explain the changes for each target
   - Warn if only one target is being modified

#### Common Patterns to Recognize

**Pattern 1: Target-Specific Assets**

```swift
// Korpu asset references (no prefix)
.backgroundMain
.checkboxOff
.buttonPrimary

// Khidi asset references (khidi prefix)
.khidiBackgroundMain
.khidiCheckboxOff
.khidiButtonPrimary
```

**Pattern 2: Target-Specific File Paths**

```
Korpu/Korpu/           → Korpu target files
Korpu/Khidi/           → Khidi target files
KorpuModules/          → Korpu-specific features
KhidiModules/          → Khidi-specific features
```

**Pattern 3: Conditional Compilation (to be refactored)**

```swift
#if KHIDI
// Khidi-specific code
#else
// Korpu-specific code
#endif
```

When agent sees this pattern, it should suggest refactoring to Palette approach.

#### Asset Catalog Configuration Tips

**Colors:**
- **Any Appearance**: Default color for light mode
- **Dark Appearance**: Color for dark mode (if supported)
- Use hex values or RGB sliders for precise colors
- Consider accessibility - maintain sufficient contrast ratios
- Test colors on actual devices, not just simulator

**Images:**
- Use PNG format for images with shadows or complex effects
- Provide all three resolutions: @1x, @2x, @3x
- **Render As**:
  - Original Image - for colored icons/images
  - Template Image - for monochrome icons that can be tinted
- **Resizing**: Preserve Vector Data (for PDFs) or specify resizing mode

---

## Typography Management

### Core Concepts

- All font configurations should be centralized in the Typography system
- Typography defines Primary and Secondary font families for each target
- Typography uses **method-based approach** with size parameter for flexibility
- Never hardcode font names or sizes directly in UI code
- Use Typography references to automatically adapt fonts across targets
- Always provide fallback fonts in case custom fonts fail to load

---

# UI Architecture Addendum (2025+)

This section extends and clarifies the existing AGENTS.md rules.
If any conflicts arise, **this addendum has higher priority**.

---

## Deprecated Components

### ❗ NeomorphismButton

`NeomorphismButton` is **DEPRECATED**.

- Must NOT be used in new code
- Allowed only for legacy screens
- Replaced by the `ConfigurableNeomorphicButton` hierarchy

All new UI must use the components described below.

---

## Neomorphic Button System (Current)

### Purpose

The neomorphic button system provides:

- unified visual appearance across the app;
- centralized state handling (`normal`, `highlighted`, `disabled`, `selected`);
- reusable press animations;
- flexible content configuration (icon + title layouts);
- full separation of **logic** and **visual style**.

---

## Final Recommendations

1. New design → new StyleProvider
2. No hardcoded colors, fonts, or shadows
3. Press responders = animation only
4. Content layout only via `IconTitleContentView`
5. Deprecated components must not appear in new code

---
