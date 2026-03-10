# Liquid Glass — SwiftUI APIs, Patterns & Gotchas

> All APIs require iOS 26+ / macOS 26+ / Xcode 26 SDK unless noted otherwise.

## Glass Effect

### `glassEffect(_:in:)`

Renders a glass shape anchored behind a view's bounds (including padding). Automatically applies vibrant text color for legibility.

```swift
nonisolated func glassEffect(
    _ glass: Glass = .regular,
    in shape: some Shape = DefaultGlassEffectShape() // Capsule
) -> some View
```

```swift
Text("Hello").padding().glassEffect()                                    // Basic
Text("Status").padding().glassEffect(in: .rect(cornerRadius: 16.0))     // Custom shape
Text("Action").padding().glassEffect(.regular.tint(.orange).interactive()) // Tinted + interactive
```

## Glass Struct

```swift
Glass.regular              // Default adaptive glass
Glass.clear                // Highly translucent (media-rich backgrounds only)
Glass.identity             // No effect (conditional opt-out, e.g., if #available)

// Chainable modifiers:
.tint(.blue)               // Vibrant adaptive tint (for primary actions, not decoration)
.interactive()             // Press/hover feedback: scale, bounce, shimmer (iOS custom controls)
```

## GlassEffectContainer

Groups glass elements: shared rendering pass, fluid merging, morph transitions. **Essential for visual correctness** — glass can't sample other glass.

```swift
GlassEffectContainer(spacing: 40.0) {
    HStack(spacing: 40.0) {
        Image(systemName: "scribble.variable")
            .frame(width: 80, height: 80)
            .glassEffect()
        Image(systemName: "eraser.fill")
            .frame(width: 80, height: 80)
            .glassEffect()
    }
}
```

- **`spacing`**: elements merge when within this distance. If container spacing > stack spacing, effects blend at rest
- Improves rendering performance (single sampling pass)

## Glass Morphing

### `glassEffectID(_:in:)` + `GlassEffectTransition`

```swift
@State private var isExpanded = false
@Namespace private var namespace

GlassEffectContainer(spacing: 40.0) {
    HStack(spacing: 40.0) {
        Image(systemName: "scribble.variable")
            .frame(width: 80, height: 80)
            .glassEffect()
            .glassEffectID("pencil", in: namespace)
        if isExpanded {
            Image(systemName: "eraser.fill")
                .frame(width: 80, height: 80)
                .glassEffect()
                .glassEffectID("eraser", in: namespace)
        }
    }
}
Button("Toggle") { withAnimation { isExpanded.toggle() } }
    .buttonStyle(.glass)
```

**Transition types**: `.matchedGeometry` (default, smooth morph for shapes within container spacing) and `.materialize` (fade with glass animation for distant shapes).

Requirements: both views in same `GlassEffectContainer`, both with `.glassEffect()`, same `@Namespace`, trigger with `withAnimation`.

To use an explicit transition:
```swift
.transition(GlassEffectTransition.materialize)
```

## Glass Union

### `glassEffectUnion(id:namespace:)`

Merges multiple glass elements into one shape at rest. Separate on interaction.

```swift
GlassEffectContainer(spacing: 20.0) {
    HStack(spacing: 20.0) {
        ForEach(symbolSet.indices, id: \.self) { item in
            Image(systemName: symbolSet[item])
                .frame(width: 80, height: 80)
                .glassEffect()
                .glassEffectUnion(id: item < 2 ? "1" : "2", namespace: namespace)
        }
    }
}
```

## Button Styles

```swift
Button("Action") { }.buttonStyle(.glass)             // Standard glass
Button("Primary") { }.buttonStyle(.glassProminent)    // Tinted/prominent
```

Buttons automatically morph into menus, popovers, and dialogs when they trigger presentations.

## Concentric Shapes

Auto-concentricity with containers (windows, sheets, device edges):

```swift
.clipShape(.rect(cornerRadius: 0, style: .containerConcentric))        // Concentric
.clipShape(.rect(cornerRadius: 12, style: .containerConcentric))       // With fallback radius
```

Platform defaults: iPhone=capsule, iPad/Mac=concentric with window, macOS mini–medium=rounded rect, macOS large/XL=capsule.

## Background Extension

### `.backgroundExtensionEffect()`

Extends content behind system bars. Creates a **mirrored + blurred replica** outside the safe area.

```swift
// From Landmarks — hero image extends behind nav bar:
Image(landmark.backgroundImageName)
    .resizable()
    .aspectRatio(contentMode: .fill)
    .frame(minWidth: 0, maxWidth: .infinity, minHeight: 0, maxHeight: .infinity)
    .backgroundExtensionEffect()
```

Can be applied per-view. Text and controls must be layered above.

## Scroll Edge Effects

### `.scrollEdgeEffectStyle(_:for:)`

Variable blur where glass UI overlaps scrolling content. System bars get this automatically; add for custom floating controls.

```swift
.scrollEdgeEffectStyle(.soft, for: .top)    // iOS/iPadOS default
.scrollEdgeEffectStyle(.hard, for: .bottom) // macOS default — stronger boundary
```

## Tab Bar

```swift
TabView { /* tabs */ }
    .tabBarMinimizeBehavior(.onScrollDown)
    .tabViewBottomAccessory { NowPlayingView() }
```

Read `tabViewBottomAccessoryPlacement` from environment to adjust content when accessory collapses inline.

Search tab: `Tab(role: .search) { SearchView() }` — field replaces tab bar, other tabs compress.

## Toolbars

```swift
.toolbar {
    ToolbarSpacer(.flexible)
    ToolbarItem { ShareLink(item: landmark, preview: landmark.sharePreview) }
    ToolbarSpacer(.fixed)
    ToolbarItemGroup {
        LandmarkFavoriteButton(landmark: landmark)
        LandmarkCollectionsMenu(landmark: landmark)
    }
    ToolbarSpacer(.fixed)
    ToolbarItem { Button("Info", systemImage: "info") { } }
}

// Remove glass background from an item:
.sharedBackgroundVisibility(.hidden)

// Badge:
Button("Notifications", systemImage: "bell") { }.badge(3)
```

## Sheets

Partial-height sheets are inset with glass by default. Full-height transitions to opaque. **Remove custom `presentationBackground`.**

Zoom transition from toolbar button:
```swift
ToolbarItem { Button("Details") { showSheet = true } }
.sheet(isPresented: $showSheet) {
    DetailView().navigationTransition(.zoom(sourceID: "details", in: namespace))
}
```

---

## Code Pattern: Landmarks BadgesView

Real Landmarks sample — combines Container + morphing + platform conditionals:

```swift
struct BadgesView: View {
    @Namespace private var namespace
    @State private var isExpanded = false
    @Environment(ModelData.self) var modelData

    var body: some View {
        GlassEffectContainer(spacing: Constants.badgeGlassSpacing) {
            VStack(alignment: .center, spacing: Constants.badgeButtonTopSpacing) {
                if isExpanded {
                    VStack(spacing: Constants.badgeSpacing) {
                        ForEach(modelData.earnedBadges) { badge in
                            BadgeLabel(badge: badge)
                                .glassEffect(.regular, in: .rect(cornerRadius: Constants.badgeCornerRadius))
                                .glassEffectID(badge.id, in: namespace)
                        }
                    }
                }
                Button {
                    withAnimation { isExpanded.toggle() }
                } label: {
                    Image(systemName: isExpanded ? "chevron.down" : "chevron.up")
                }
                .buttonStyle(.glass)
                #if os(macOS)
                .tint(.clear)
                #endif
                .glassEffectID("togglebutton", in: namespace)
            }
            .frame(width: Constants.badgeFrameWidth)
        }
    }
}
```

---

## Gotchas

1. **Apply `glassEffect` AFTER appearance modifiers** — glass captures content at the point it's applied. `.font()`, `.padding()` go before `.glassEffect()`.
2. **Container spacing > layout spacing = blend at rest** — match `GlassEffectContainer(spacing:)` to your stack spacing intentionally.
3. **Remove custom `presentationBackground`** and **custom toolbar backgrounds** — they block glass and scroll edge effects.
4. **`GlassEffectContainer` must wrap both source and destination** for morph transitions.
5. **Use `Glass.identity` for conditional opt-out** — better than conditionals that change view hierarchy.
