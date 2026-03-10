---
name: liquid-glass
description: >
  Reference for Apple's Liquid Glass design system (iOS 26+, macOS 26+, Xcode 26).
  Proactively use this skill when: (1) writing or reviewing SwiftUI/UIKit/AppKit code that uses
  glass effects, glass buttons, scroll edge effects, background extension, toolbar grouping,
  tab bar minimization, or sheet presentations targeting iOS 26+/macOS 26+,
  (2) migrating an app to iOS 26 or macOS Tahoe design,
  (3) discussing Liquid Glass design patterns, visual hierarchy, tinting, or concentricity,
  (4) debugging glass rendering issues or performance.
  TRIGGER when: code references glassEffect, GlassEffectContainer, glassEffectID,
  UIGlassEffect, UIGlassContainerEffect, NSGlassEffectView, NSGlassEffectContainerView,
  UIBackgroundExtensionView, NSBackgroundExtensionView, backgroundExtensionEffect,
  scrollEdgeEffectStyle, .glass buttonStyle, .glassProminent, clearGlass,
  tabBarMinimizeBehavior, GlassEffectTransition, glassEffectUnion,
  or user mentions Liquid Glass, iOS 26 design, macOS Tahoe design, new design system.
---

# Liquid Glass Design System Reference

> Apple's Liquid Glass — a translucent, dynamic material for controls and navigation — introduced at WWDC25 for iOS 26, macOS 26 (Tahoe), iPadOS 26, watchOS 26, tvOS 26, visionOS 26.
>
> **Sources**: WWDC25 sessions 219, 323, 284, 310, 356; Meet with Apple 208; Apple HIG; Landmarks sample project.

## Resource Routing Table

| If the task involves...                        | Read first                | Then maybe read           |
|------------------------------------------------|---------------------------|---------------------------|
| Implementing glass in SwiftUI                  | references/swiftui.md     | references/design-rules.md |
| Implementing glass in UIKit                    | references/uikit.md       | references/design-rules.md |
| Implementing glass in AppKit                   | references/appkit.md      | references/design-rules.md |
| Reviewing design / visual hierarchy / critique | references/design-rules.md |                           |
| Migrating existing app to iOS 26               | references/migration.md   | references/design-rules.md |
| Debugging visual glitch with glass             | references/design-rules.md (Anti-Patterns) |            |
| Performance issues with glass                  | references/design-rules.md (Anti-Patterns) | Framework file |

## Glass Variants Quick Matrix

| Variant    | When to use                                      | Visual effect                       |
|------------|--------------------------------------------------|-------------------------------------|
| `.regular` | Default for all standard UI controls             | Adaptive blur, light/dark automatic |
| `.clear`   | Media-rich backgrounds where blur would obscure  | Highly translucent, minimal blur    |
| `.identity`| Conditionally disable glass (e.g., `if #available`) | No visual effect applied          |

**Clear requires ALL 3**: (1) over media-rich content, (2) dimming layer won't harm content, (3) content above is bold and bright. Add 35% dark dimming if background is bright.

## API Quick Reference

| API                                  | Framework | Purpose                                      |
|--------------------------------------|-----------|----------------------------------------------|
| `.glassEffect(_:in:)`               | SwiftUI   | Apply glass material to any view              |
| `Glass` struct                       | SwiftUI   | Configure variant, tint, interactivity        |
| `GlassEffectContainer`              | SwiftUI   | Group glass elements for shared rendering     |
| `.glassEffectID(_:in:)`             | SwiftUI   | Tag elements for morph transitions            |
| `GlassEffectTransition`             | SwiftUI   | Animate morphing between glass elements       |
| `.glassEffectUnion(id:namespace:)`  | SwiftUI   | Merge glass elements at rest                  |
| `.buttonStyle(.glass)`              | SwiftUI   | Glass button style                            |
| `.buttonStyle(.glassProminent)`     | SwiftUI   | Tinted glass button style                     |
| `.backgroundExtensionEffect()`      | SwiftUI   | Extend content behind system bars             |
| `.scrollEdgeEffectStyle(_:for:)`    | SwiftUI   | Configure scroll edge transitions             |
| `UIGlassEffect`                      | UIKit     | Glass material for UIVisualEffectView         |
| `UIGlassContainerEffect`            | UIKit     | Container for grouped glass elements          |
| `UIButton.Configuration.glass()`    | UIKit     | Glass button configuration                    |
| `UIBackgroundExtensionView`          | UIKit     | Extend content behind bars                    |
| `NSGlassEffectView`                  | AppKit    | Glass material view for macOS                 |
| `NSGlassEffectContainerView`        | AppKit    | Container for grouped glass on macOS          |
| `NSBackgroundExtensionView`          | AppKit    | Background extension for macOS                |
| `NSView.LayoutRegion`               | AppKit    | Corner-aware layout regions                   |

## 5 Core Design Principles

1. **Two layers**: UI layer (glass) floats above content layer (brand/data) — never mix them
2. **Sparing use**: Apply glass only to the most important functional elements (navigation, primary controls)
3. **No stacking**: Never place glass on glass — glass cannot sample other glass correctly
4. **Concentricity**: Shapes nest with calculated radii — inner radius = outer radius - padding
5. **Selective tinting**: Tint only primary actions or functionally distinct elements, never everything

## Top Anti-Patterns

| Anti-Pattern | Why it fails | See also |
|---|---|---|
| Glass in content layer | Glass is for navigation/controls only; content should be brand/data | design-rules.md § Visual Hierarchy |
| Glass-on-glass stacking | Glass cannot sample other glass; visual artifacts result | design-rules.md § Anti-Patterns |
| Custom bar backgrounds | `UIBarAppearance`/`backgroundColor` blocks glass + scroll edge effects | migration.md § Audit Checklist |
| Too many glass effects | GPU-intensive; limit to top-level navigational elements | design-rules.md § Anti-Patterns |
| Missing `GlassEffectContainer` | Grouped elements render independently instead of blending | Framework file § Container APIs |
| Nesting glass modifiers | Apply at highest appropriate level only; padding OUTSIDE glass scope | migration.md § CNN lesson |
| Hard-coded control heights | Controls are slightly taller in iOS 26; use Auto Layout | Framework file § Gotchas |
| Logos in toolbars | Scroll edge effect causes legibility issues; let logos scroll with content | migration.md § American Airlines |
