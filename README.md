# Liquid Glass Skill

AI skill reference for Apple's Liquid Glass design system — APIs, patterns, and migration guidance for iOS 26+/macOS 26+.

## Why this exists

Liquid Glass shipped after the training cutoff of current AI models. When you ask Claude Opus 4.6 or Claude Sonnet 4.6 about `glassEffect`, `UIGlassEffect`, or `NSGlassEffectView`, they don't know these APIs exist.

This skill gives AI coding assistants accurate, verified reference material for the entire Liquid Glass design system across SwiftUI, UIKit, and AppKit — including gotchas, anti-patterns, and real-world migration lessons that no Apple doc covers.

### What Apple docs cover vs. what this skill uniquely provides

| Content | In Apple Docs? | In This Skill? |
|---------|---------------|----------------|
| SwiftUI Glass core APIs | Yes | Yes (more concise for code gen) |
| UIKit glass patterns and gotchas | Partial | Yes (consolidated with gotchas) |
| AppKit glass patterns and gotchas | Partial | Yes (consolidated with gotchas) |
| Framework-specific gotchas | No | **Only source** |
| Anti-patterns with fixes | No | **Only source** |
| Real-world migration lessons (CNN, Slack, Sky Guide, etc.) | No | **Only source** |

## Installation

### Claude Code (as a skill)

Copy the `SKILL.md` and `references/` directory to your Claude skills folder:

```bash
mkdir -p ~/.claude/skills/liquid-glass
cp SKILL.md ~/.claude/skills/liquid-glass/
cp -r references ~/.claude/skills/liquid-glass/
```

The skill activates automatically when your conversation involves glass APIs or iOS 26 design.

### Other AI tools (as context)

The files are plain markdown — feed them as context to any AI coding assistant. Each framework file is self-contained (~140–250 lines):

- **SwiftUI work**: `references/swiftui.md`
- **UIKit work**: `references/uikit.md`
- **AppKit work**: `references/appkit.md`
- **Design review**: `references/design-rules.md`
- **Migration planning**: `references/migration.md`

## File structure

```
SKILL.md                      # Router + API quick reference + core principles + anti-patterns
references/
├── swiftui.md                # SwiftUI APIs + Landmarks code patterns + gotchas
├── uikit.md                  # UIKit APIs + code patterns + gotchas
├── appkit.md                 # AppKit APIs + code patterns + gotchas
├── design-rules.md           # HIG guidance, hierarchy, concentricity, tinting, accessibility
└── migration.md              # Adoption checklist + real-world lessons from 7 apps
```

## Sources

Synthesized from primary Apple sources and verified against shipped APIs:

- [Meet Liquid Glass](https://developer.apple.com/videos/play/wwdc2025/219/) — WWDC25-219: core design principles, variants, tinting, accessibility
- [Build a SwiftUI app with the new design](https://developer.apple.com/videos/play/wwdc2025/323/) — WWDC25-323: SwiftUI APIs, Landmarks sample walkthrough
- [Build a UIKit app with the new design](https://developer.apple.com/videos/play/wwdc2025/284/) — WWDC25-284: UIKit APIs, glass effects, transitions
- [Build an AppKit app with the new design](https://developer.apple.com/videos/play/wwdc2025/310/) — WWDC25-310: AppKit APIs, toolbar grouping, layout regions
- [Get to know the new design system](https://developer.apple.com/videos/play/wwdc2025/356/) — WWDC25-356: design language, shapes, concentricity, scroll edges
- [Showcase: Learn how apps are integrating the new design and Liquid Glass](https://developer.apple.com/videos/play/meet-with-apple/208/) — adoption stories from LTK, Slack, CNN, Lowe's, Sky Guide, Tide Guide, American Airlines
- Apple HIG pages: Materials, Scroll Views, Buttons, Toolbars, Tab Bars, Sidebars
- [Landmarks: Building an app with Liquid Glass](https://developer.apple.com/documentation/swiftui/landmarks-building-an-app-with-liquid-glass) — official sample project

## Contributing

Found an inaccuracy, a missing API, or have experience adopting Liquid Glass in your own app? PRs welcome. Please include the source (Apple doc URL, Xcode header, or your own adoption experience) for any additions.
