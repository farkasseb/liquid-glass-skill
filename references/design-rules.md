# Liquid Glass — Design Rules & HIG Guidance

> Cross-cutting design principles for Liquid Glass across all platforms.

## Visual Hierarchy

Two-layer system:

| Layer | Purpose | Elements | Material |
|-------|---------|----------|----------|
| **UI Layer** (top) | Navigation & controls | Tab bars, toolbars, nav bars, floating buttons, sidebars | Liquid Glass |
| **Content Layer** (bottom) | Brand & data | Text, images, cards, lists, media | Opaque/custom |

- **Don't use glass in the content layer** (HIG). Exception: transient interactive elements (slider knobs, toggle thumbs) during interaction only
- Brand expression belongs in the content layer — colors, imagery, typography

## Glass Variants

**Regular** (default): all adaptive effects; works in any size, over any content. Use most of the time.

**Clear**: highly translucent, no adaptive light/dark switching. Requires ALL 3: (1) over media-rich content, (2) dimming layer won't harm content, (3) content above is bold and bright. Add **35% opacity dark dimming** if background is bright. **Never mix Regular and Clear** in the same context.

**Identity**: no visual effect. For conditional opt-out.

### Size-Adaptive Behavior
- **Small elements** (nav bars, tab bars, buttons): flip light/dark based on background
- **Large elements** (menus, sidebars, sheets): adapt but DON'T flip — surface area too big
- Window loses focus (Mac/iPad): glass visually recedes

## Concentricity & Shapes

Inner radius = outer radius - padding. Three types: **fixed** (constant radius), **capsules** (radius = half height), **concentric** (calculated from parent).

Platform defaults: iPhone=capsule, iPad/Mac=concentric with window, macOS mini–medium=rounded rect, large/XL=capsule.

**Fallback radius trick**: concentric shape with a fallback — adapts when nested, fallback kicks in standalone. Watch for pinched/flared corners in nested containers — usually means the inner shape needs to be concentric.

## Tinting

- Tint **selectively** — only primary actions or functionally distinct elements. **Never tint all**
- Generates adaptive tones mapped to background brightness. Use system colors
- Don't use solid fills on glass buttons — breaks the material character

**Emphasis**: tinted+prominent > tinted > untinted.

## Scroll Edge Effects

Variable blur between content and Liquid Glass controls. **Not decorative** — only where scroll view is adjacent to floating interface elements.

`.soft` = iOS/iPadOS default (subtle blur). `.hard` = macOS default (stronger boundary, for interactive text/pinned headers).

**One per view edge. Don't mix or stack** soft and hard. In split views, each pane can have its own — keep consistent height.

## Toolbars

- Don't mix text + icons in same group (looks like one combined button)
- `.prominent` for key actions (Done, Submit) — one primary action, trailing side
- Monochrome icons reduce noise. Remove custom backgrounds/borders
- Separate groups with spacers

## Sidebars

- Float above content on glass. **Extend content beneath** (HIG) with `backgroundExtensionEffect` or horizontal scroll
- Text/controls above the extension. Remove legacy blur/material views

## Accessibility

Automatic with standard components:

| Setting | Adaptation |
|---------|-----------|
| **Reduce Transparency** | Frostier, more opaque |
| **Increase Contrast** | Solid black/white with contrasting border |
| **Reduce Motion** | Disables elastic/fluid properties |

Test all three with custom glass elements.

---

## Anti-Patterns

**Glass-on-glass**: Never layer glass on glass. Use `GlassEffectContainer` or flatten. Remove floating glass buttons when a glass sheet expands (Maps pattern).

**Content intersecting glass at rest**: Reposition/scale content. Use `backgroundExtensionEffect` for dynamic overlap during scroll, not static.

**Too many glass effects**: GPU-intensive. Limit to top-level elements. Consolidate with containers. Profile with Instruments.

**Custom bar backgrounds**: `UIBarAppearance`, `.toolbarBackground()`, `backgroundColor` blocks glass + scroll edge effects. Remove all.

**Nesting glass modifiers**: Parent + child = double translucency. Apply at highest level only. Padding goes OUTSIDE glass modifier scope — wrap in `background`/`overlay` (CNN lesson).

**Logos in toolbars**: Scroll edge effect causes legibility issues. Let logos scroll with content (American Airlines lesson).
