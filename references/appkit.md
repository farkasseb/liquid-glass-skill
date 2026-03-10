# Liquid Glass — AppKit APIs, Patterns & Gotchas

> All APIs require macOS 26+ (Tahoe) / Xcode 26 SDK unless noted otherwise.

## NSGlassEffectView

```swift
let glass = NSGlassEffectView()
glass.contentView = myContentView  // MUST use contentView — ensures legibility
glass.cornerRadius = 12
glass.tintColor = .controlAccentColor
```

Always set content via `contentView`. Don't place glass as a sibling behind content.

## NSGlassEffectContainerView

Groups glass elements: shared rendering, fluid merging, visual correctness, performance (single sampling pass).

```swift
let container = NSGlassEffectContainerView()
container.spacing = 10  // Merge distance
let glass1 = NSGlassEffectView()
glass1.contentView = button1
container.addSubview(glass1)
```

## NSBackgroundExtensionView

Extends content behind window chrome. System creates mirrored + blurred replica automatically.

```swift
let ext = NSBackgroundExtensionView()
ext.contentView = posterImageView  // Positioned in safe area; replica fills outside
view.addSubview(ext)
```

## Window Corners & Layout Regions

**Window corner radius**: windows with toolbars use larger radius (concentric with glass toolbar elements); titlebar-only = smaller radius. Larger corners can **clip content** near edges.

### `NSView.LayoutRegion` — corner-aware layout guides:

```swift
let guide = NSView.LayoutRegion.safeArea
    .withCornerAdaptation(.horizontal)
    .layoutGuide(for: myView)

NSLayoutConstraint.activate([
    button.trailingAnchor.constraint(equalTo: guide.trailingAnchor),
    button.bottomAnchor.constraint(equalTo: guide.bottomAnchor),
])
```

## Split View & Sidebars

Use `NSSplitViewController` — AppKit provides glass automatically:
- **Sidebar**: floating glass above content
- **Inspector**: edge-to-edge glass alongside content

```swift
// Extend content under sidebar — set on CONTENT item, not sidebar:
contentSplitItem.automaticallyAdjustsSafeAreaInsets = true
```

**Remove legacy `NSVisualEffectView`** from sidebars — it blocks the new glass.

### Split View Accessories

```swift
splitViewItem.addTopAlignedAccessoryViewController(accessory)
// Or: addBottomAlignedAccessoryViewController
```

Per-split-item, influences scroll edge effect size, insets content safe area.

## Scroll Edge Effects

Lives inside `NSScrollView`. Two styles: **soft** (progressive fade) and **hard** (opaque backing). Applied automatically under toolbar items, titlebar accessories, and split item accessories. Adapts size/shape as floating elements come and go.

## Toolbars

Items auto-group on glass backgrounds by control type.

```swift
// Remove glass from non-interactive items (status text, titles):
statusItem.isBordered = false

// Prominent style + custom tint:
item.style = .prominent
item.backgroundTintColor = .systemBlue

// Badge:
item.badge = NSItemBadge(count: 3)
```

Toolbar glass switches light/dark based on scrolled content brightness (via `NSAppearance`).

## Controls & Buttons

Sizes: `.mini`, `.small`, `.regular`, `.large`, `.extraLarge` (new). All slightly taller than previous macOS.

```swift
button.bezelStyle = .glass           // Glass material (NSButton only)
button.bezelColor = .systemBlue      // Tint the glass
button.borderShape = .capsule        // or .roundedRect

// Compact metrics for existing dense layouts (inherited down hierarchy):
view.prefersCompactControlSizeMetrics = true
```

## Tint Prominence

```swift
view.tintProminence = .automatic  // System decides
view.tintProminence = .none       // Minimal tint
view.tintProminence = .secondary  // Subdued (e.g., destructive buttons)
view.tintProminence = .primary    // Most prominent (default button auto-gets this)
```

Slider: `.none` = no track fill; `.secondary`/`.primary` = filled. Use `slider.neutralValue` for bidirectional fill anchor.

---

## Gotchas

1. **Remove `NSVisualEffectView` from sidebars** — blocks the new glass from showing through.
2. **Set `contentView`, don't add glass as sibling** — glass must manage its content for legibility.
3. **`isBordered = false` for non-interactive toolbar items** — status text/labels shouldn't look like buttons.
4. **`automaticallyAdjustsSafeAreaInsets` on CONTENT, not sidebar** — extends content frame beneath floating sidebar.
5. **Use Auto Layout** — controls are taller in macOS 26. Use `prefersCompactControlSizeMetrics` only for compatibility.
6. **Window corners clip content** — use `NSView.LayoutRegion` with corner adaptation for edge-positioned content.
7. **Glass bezel only on `NSButton`** — don't apply to other control types.
8. **`#available` guard:**
```swift
if #available(macOS 26, *) {
    let glass = NSGlassEffectView()
    glass.contentView = myView
}
```
