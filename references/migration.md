# Liquid Glass — Migration Guide

> Checklist and lessons for adopting Liquid Glass in existing apps.

## Quick Start

1. **Build with Xcode 26 SDK** — standard components auto-adopt glass
2. **Run on iOS 26 / macOS 26** — see changes immediately

## Audit Checklist

### Remove Conflicts (High Priority)
- [ ] Remove custom bar backgrounds (`UIBarAppearance`, `.toolbarBackground()`, `backgroundColor`)
- [ ] Remove custom sheet/popover backgrounds (`presentationBackground`, `UIVisualEffectView`)
- [ ] Remove legacy sidebar materials (`NSVisualEffectView`, `UIVisualEffectView`)
- [ ] Review control dimensions — use Auto Layout, not hard-coded heights
- [ ] Check safe areas for sidebars/inspectors with `backgroundExtensionEffect`

### Enhance (Medium Priority)
- [ ] Add `backgroundExtensionEffect` for hero images/headers behind bars
- [ ] Review toolbar grouping — add spacers, separate primary actions with `.prominent`
- [ ] Consider tab bar minimization for reading screens (`.onScrollDown`)
- [ ] Add scroll edge effects for custom floating controls
- [ ] Set `sourceItem`/`sourceView` on all action sheets (both iPhone and iPad)

### Polish (Lower Priority)
- [ ] Tint only primary actions — brand color in content layer
- [ ] Section headers: title-style capitalization (no longer all caps)
- [ ] Add SF Symbols to menu items (leading edge icons)
- [ ] Move logos out of toolbar (scroll edge legibility issues)
- [ ] Test: Reduce Transparency, Increase Contrast, Reduce Motion
- [ ] Profile GPU with Instruments

## Opt-Out & Backward Compatibility

**Full opt-out** via Info.plist:
```xml
<key>UIDesignRequiresCompatibility</key>
<true/>
```

**Conditional glass** for mixed-deployment targets:
```swift
extension View {
    @ViewBuilder func conditionalGlass() -> some View {
        if #available(iOS 26, *) {
            self.glassEffect()
        } else {
            self
        }
    }
}
```

## Real-World Lessons

### LTK (Shopping)
- Start small: button → pattern → screen → system. "An upgrade, not a rebuild"
- 70% compile time reduction from modernizing alongside glass adoption

### Slack (Messaging)
- Phased: Sept=most-used elements, Oct=riskier changes, Nov=media, Dec=landscape
- Prototyped multiple header designs before settling. "Using native controls felt like swimming with the OS"

### CNN (News)
- **Nested glass modifiers** = double translucency, unpredictable rendering. Apply at highest level only
- Padding fix: wrap in `background`/`overlay`, put padding OUTSIDE glass modifier scope
- GPU-intensive in scrollable views — limit to static/top-level components

### Lowe's (Retail)
- On-device demos > screenshots for leadership buy-in. `UIDesignRequiresCompatibility` rollback ready but never needed

### Sky Guide (Astronomy)
- Prototyped in 2 days to verify perf with real-time rendering. Backward compat wrappers: estimated 1.5 weeks, done in 1.5 days

### Tide Guide (Marine Weather)
- "Adopt then redesign" — best aspects come from latest system components
- Custom glass for unique interactions (tide chart highlight, glass popovers replacing context menus)

### American Airlines
- Removed logo from toolbar — scroll edge effect caused legibility issues. Logo scrolls with content instead
