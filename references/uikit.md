# Liquid Glass — UIKit APIs, Patterns & Gotchas

> All APIs require iOS 26+ / Xcode 26 SDK unless noted otherwise.

## UIGlassEffect

Glass material via `UIVisualEffectView`. Distinct from `UIBlurEffect` — designed for the interactive control layer only.

```swift
let glassEffect = UIGlassEffect()          // Regular style (default)
let clearGlass = UIGlassEffect(style: .clear) // Clear variant

let effectView = UIVisualEffectView(effect: nil)
view.addSubview(effectView)

// Materialize with animation:
UIView.animate(withDuration: 0.3) { effectView.effect = glassEffect }

// Dematerialize (NOT alpha = 0):
UIView.animate(withDuration: 0.3) { effectView.effect = nil }
```

### Properties

```swift
glassEffect.isInteractive = true       // Scale, bounce, shimmer on touch
glassEffect.tintColor = .systemBlue    // Vibrant adaptive tint
```

**Size-adaptive**: larger → more opaque; smaller → clearer + auto light/dark switching. Default shape is capsule.

**Content**: add to `effectView.contentView`. Labels auto-become vibrant. Use dynamic colors (`.label`, `.secondaryLabel`).

## UIGlassContainerEffect

Groups glass elements for shared rendering, fluid merging, visual correctness, and performance (single sampling pass).

```swift
let container = UIGlassContainerEffect()
container.spacing = 12  // Merge distance
let containerView = UIVisualEffectView(effect: container)

let child1 = UIVisualEffectView(effect: UIGlassEffect())
let child2 = UIVisualEffectView(effect: UIGlassEffect())
containerView.contentView.addSubview(child1)
containerView.contentView.addSubview(child2)
```

**Split animation**: add at same position without animation, then animate apart.

## Glass Button Configurations

```swift
UIButton.Configuration.glass()               // Standard glass
UIButton.Configuration.prominentGlass()      // Tinted with app tint color
UIButton.Configuration.clearGlass()          // Clear variant
UIButton.Configuration.prominentClearGlass() // Prominent clear
```

## UIBackgroundExtensionView

Extends content behind system bars. Creates mirrored + blurred replica outside safe area. **Extension view is a sibling, not a parent.**

```swift
let extensionView = UIBackgroundExtensionView()
view.addSubview(extensionView)
extensionView.contentView.addSubview(heroImageView)

// Manual positioning:
extensionView.automaticallyPlacesContentView = false
// Use extensionView.safeAreaLayoutGuide for sidebar-aware constraints
```

Details/overlays must be **siblings** of the extension view, not subviews.

## Scroll Edge Effects

System bars get scroll edge effects automatically. For custom floating controls:

```swift
let interaction = ScrollEdgeElementContainerInteraction()
interaction.contentScrollView = scrollView
interaction.edge = .bottom
interaction.style = .soft  // or .hard
floatingBar.addInteraction(interaction)
```

## Tab Bar

```swift
tabBarController.tabBarMinimizeBehavior = .onScrollDown

// Bottom accessory:
let accessory = UITabAccessory(contentViewController: nowPlayingVC)
tabBarController.bottomAccessory = accessory
// Observe inline/expanded state via tabAccessoryEnvironment trait
```

## Navigation Bar & Toolbars

**Bar background is transparent by default.** Remove all `UIBarAppearance`/`backgroundColor` customization.

### Item grouping

Items auto-group on shared glass backgrounds. Image buttons share; text/Done/prominent buttons get separate backgrounds.

```swift
// Split groups with fixedSpace:
navigationItem.rightBarButtonItems = [doneBtn, .fixedSpace(20), shareBtn, infoBtn]

// Prominent primary action:
doneButton.style = .prominent

// Keep single background across flexible space:
flexSpace.hidesSharedBackground = false  // default true = separates

// Tint a symbol:
flagButton.tintColor = .systemOrange

// Badge:
bellButton.badge = UIBarButtonItemBadge(count: 5)
```

## Sheets & Action Sheets

Sheets adopt glass automatically. **Remove custom `presentationBackground`.**

**Zoom transition from toolbar button:**
```swift
detailVC.preferredTransition = .zoom { _ in
    return self.navigationItem.rightBarButtonItems?.first
}
```

**Action sheets** anchor to source on **both iPhone and iPad**:
```swift
alert.popoverPresentationController?.sourceItem = button
// With sourceItem: inline, no cancel button (dismiss by tapping elsewhere)
// Without sourceItem: centered, with cancel button
```

---

## Code Pattern: Background Extension (TV app style)

```swift
class ShowVC: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        let ext = UIBackgroundExtensionView()
        ext.automaticallyPlacesContentView = false
        view.addSubview(ext)

        ext.contentView.addSubview(imageView)
        imageView.leadingAnchor.constraint(
            equalTo: ext.safeAreaLayoutGuide.leadingAnchor
        ).isActive = true

        // Details as SIBLING:
        view.addSubview(detailsView)
    }
}
```

---

## Gotchas

1. **Use `effect` property, NOT `alpha`** — animate between `UIGlassEffect()` and `nil`. Setting alpha skips the glass materialize/dematerialize animation.
2. **Extension views are siblings, NOT subviews** — detail views must also be siblings of the extension view.
3. **Remove `UIBarAppearance`/`backgroundColor`** — blocks glass AND scroll edge effects.
4. **`isInteractive = true` for custom glass** — without this, custom glass views won't have press feedback like system controls.
5. **Action sheet without `sourceItem` = centered + cancel button** — always set source for inline anchored appearance.
6. **`#available` guard for mixed targets:**
```swift
if #available(iOS 26, *) {
    let effect = UIGlassEffect()
    // apply glass
}
```
