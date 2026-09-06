---
id: platforms/liquid-glass-api
lang: en
version: 1
source-lang: en
status: active
digest: 52679792
---

# Liquid Glass API reference

## Scope and verification

Symbol-level reference for the Liquid Glass APIs, verified against the Xcode 27 SDK interfaces (`iPhoneOS27.0.sdk`, `MacOSX27.sdk`) and Apple's documentation pages on 2026-09-06. Baseline: everything under "SwiftUI/UIKit/AppKit APIs" ships with iOS 26 / macOS 26 / tvOS 26 / watchOS 26; the "New in iOS 27" section requires 27.0 minimums and availability gating.

There is no back deployment: apps supporting iOS 25 or earlier must gate glass code with `if #available(iOS 26.0, *)` (or `#available(macOS 26.0, *)`) and provide a fallback. visionOS does not adopt Liquid Glass — the core glass symbols are `@available(visionOS, unavailable)` and keep that platform's own design language. Design rules: [Liquid Glass design](liquid-glass-design.md); recipes: [Liquid Glass implementation patterns](liquid-glass-patterns.md).

## SwiftUI: core glass APIs

| Symbol                                                        | Purpose                                                                  |
| ------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `glassEffect(_:in:)`                                          | Render a glass shape behind a view; capsule + `.regular` by default      |
| `Glass`                                                       | Material configuration: `.regular`, `.clear`, `.identity`                |
| `Glass.tint(_:)` / `Glass.interactive(_:)`                    | Tinted variant / system touch-and-pointer reactions                      |
| `GlassEffectContainer(spacing:content:)`                      | One render pass for a group; enables blending and morphing               |
| `glassEffectID(_:in:)`                                        | Stable identity for morphs inside a container + `Namespace`              |
| `glassEffectUnion(id:namespace:)`                             | Merge elements into one shape at rest (same shape and variant only)      |
| `glassEffectTransition(_:)`                                   | `.matchedGeometry` (near) or `.materialize` (far) add/remove transitions |
| `.buttonStyle(.glass)` / `.glassProminent` / `.glass(.clear)` | System glass button styles — preferred over raw effects on buttons       |

SwiftUI framework; iOS/iPadOS/macCatalyst/macOS/tvOS/watchOS 26.0+ — no visionOS (the material does not exist there); tvOS applies glass regardless of focus.

## SwiftUI: chrome behavior APIs

| Symbol                                            | Purpose                                                                                                    |
| ------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `tabBarMinimizeBehavior(_:)`                      | Floating tab bar minimizes on scroll; `.onScrollDown` is iPhone-only                                       |
| `ToolbarSpacer(.fixed/.flexible, placement:)`     | Splits toolbar items into separate glass groupings (iOS/macOS only)                                        |
| `sharedBackgroundVisibility(_:)` (ToolbarContent) | Drops an item's shared glass background (own grouping; iOS/macOS only)                                     |
| `scrollEdgeEffectStyle(_:for:)`                   | `.automatic` / `.soft` / `.hard` per edge; `nil` restores system default. Modifier unavailable on visionOS |
| `scrollEdgeEffectHidden(_:for:)`                  | Removes the edge effect for given edges (same visionOS gap)                                                |
| `backgroundExtensionEffect()`                     | Content visually extends under sidebars/inspectors                                                         |
| `tabViewBottomAccessory(content:)`                | Persistent accessory above the tab bar, collapsing with it (iOS family only)                               |

SwiftUI framework; iOS 26.0+ baseline with platform deviations noted per row (`tabBarMinimizeBehavior`, `backgroundExtensionEffect`, and the type `ScrollEdgeEffectStyle` also exist on visionOS).

## UIKit APIs

| Symbol                                                        | Purpose                                                             |
| ------------------------------------------------------------- | ------------------------------------------------------------------- |
| `UIGlassEffect(style:)`                                       | Glass material for `UIVisualEffectView`; `.regular` / `.clear`      |
| `UIGlassEffect.tintColor` / `.isInteractive`                  | Tint; interactive touch feedback                                    |
| `UIGlassContainerEffect` (`spacing`)                          | Container effect: nested glass views render as one combined surface |
| `UIButton.Configuration.glass()` (+ prominent/clear variants) | Glass button configurations (no Mac Catalyst)                       |
| `UIScrollView.topEdgeEffect` / `.bottomEdgeEffect`            | Per-edge `UIScrollEdgeEffect` (`style`, `isHidden`)                 |
| `UIScrollEdgeElementContainerInteraction`                     | Registers custom overlays to shape the scroll edge effect           |
| `UIBarButtonItem.hidesSharedBackground`                       | Item leaves the shared glass background                             |
| `UITabBarController.tabBarMinimizeBehavior`                   | Tab bar minimization (UIKit side)                                   |
| `UIBackgroundExtensionView`                                   | Background extension container                                      |

UIKit; iOS/iPadOS/macCatalyst/tvOS 26.0+ (glass effect and button configs unavailable on visionOS/watchOS; button configs also absent on Catalyst).

## AppKit APIs

| Symbol                                   | Purpose                                                              |
| ---------------------------------------- | -------------------------------------------------------------------- |
| `NSGlassEffectView`                      | Glass container: `contentView`, `cornerRadius`, `tintColor`, `style` |
| `NSGlassEffectContainerView` (`spacing`) | Merges descendant glass views into fewer render passes               |
| `NSButton.BezelStyle.glass`              | Glass bezel — preferred over custom glass behind buttons             |
| `NSBackgroundExtensionView`              | Background extension container (titlebar/sidebar/inspector)          |
| `NSGlassEffectView.effectIsInteractive`  | Interactive glass on macOS — 27.0+ only                              |

AppKit; macOS 26.0+ except `effectIsInteractive` (27.0+).

## New in iOS 27

Verified in the Xcode 27 SDK and iOS 27 release notes; gate on `#available(iOS 27, *)` / `macOS 27`:

- `toolbarMinimizationBehavior(_:for:)` (+ `toolbarMinimizationSafeAreaAdjustment`) — renames and extends `toolbarMinimizeBehavior`; supported placement `.navigationBar`.
- `UINavigationItem.navigationBarMinimization` (`UIBarMinimizationBehavior`: `.automatic/.never/.onScrollDown/.onScrollUp`) — replaces beta-era `barMinimizeBehavior` naming. WWDC26 sample code may show stale names.
- `Tab(role: .prominent)` / `TabRole.prominent` and `UITabBarController.prominentTabIdentifier` — prominent tab pinned at the trailing edge.
- `UITabBarController.Sidebar.preferredPlacement` (`.sidebar/.tabBar/.automatic`) — sidebar representation of a tab bar on iPhone when space allows.
- `ToolbarOverflowMenu` and `ToolbarContent.visibilityPriority(_:)` — toolbar reflow under space pressure.
- `ToolbarItemPlacement.topBarPinnedTrailing` — item pinned to the trailing edge regardless of reflow.
- `toolbarColorScheme(_:for: .statusBar)` / `toolbarVisibility(_:for: .statusBar)` — status-bar control.
- Environment `appearsActive` — existed since iOS 18; iOS 27 gives inactive windows a distinct look, so custom chrome should now key off it (not a new symbol).
- `NSGlassEffectView.effectIsInteractive` (macOS 27) and `UIMenuElement.preferredImageVisibility` (menu icons hidden by default).
- Runtime changes without new APIs: retuned material (diffusion, darkened edge, specular highlights), the user appearance slider (ultra clear ↔ fully tinted), uniform toolbar on scroll under floating bars, edge-to-edge sidebars with accent-colored icons on iPad/Mac, tighter uniform window corner radius on macOS, resizable iPhone apps.

## Development notes

1. Group custom glass in one `GlassEffectContainer` per functional area and keep the on-screen count low — every glass layer costs render passes; glass outside containers also samples inconsistently (glass cannot sample other glass).
2. `glassEffect` captures the view's content — order it after appearance modifiers.
3. Morph rules: `.matchedGeometry` works within the container's `spacing`; use `.materialize` beyond it. Under the default animation matchedGeometry adds scale/offset flourishes — pass an explicit animation to opt out. `glassEffectUnion` requires identical shape and `Glass` variant across members.
4. `.clear` needs a dimming layer over bright content (`.background(.black.opacity(0.3))` per Apple's example) and must never mix with `.regular` on related elements.
5. Accessibility adaptation is automatic for system components only — test custom glass under Reduce Transparency, Increase Contrast, Reduce Motion, and the iOS 27 appearance slider.
6. Platform gaps to gate: the core glass symbols are unavailable on visionOS (that platform keeps its own design language) but ship on watchOS 26; the `scrollEdgeEffectStyle`/`scrollEdgeEffectHidden` modifiers are visionOS-unavailable even though the `ScrollEdgeEffectStyle` type exists there; the `UIButton.Configuration.glass()` family is absent on Mac Catalyst per Apple's availability metadata (Catalyst code uses `UIGlassEffect`, which is available there); native macOS uses `NSGlassEffectView`.
7. Hide toolbar items via the item (`ToolbarItem`/`UIBarButtonItem` `isHidden`), never their content views.
8. iOS 27 `.automatic` scroll edge effect has its own visuals (no longer alternates soft/hard) — re-evaluate explicit `.soft` overrides when adopting the 27 SDK.
9. Building with the iOS 27 SDK makes iPhone apps resizable and requires the UIScene lifecycle and a launch-screen key — new projects get these by default from current templates.
10. `UIDesignRequiresCompatibility` (Info.plist) restores the pre-glass look; treat it as a last resort, never a new-project default.

## Official documentation index

- Adopting Liquid Glass — <https://developer.apple.com/documentation/TechnologyOverviews/adopting-liquid-glass>
- Applying Liquid Glass to custom views — <https://developer.apple.com/documentation/swiftui/applying-liquid-glass-to-custom-views>
- Liquid Glass overview — <https://developer.apple.com/documentation/TechnologyOverviews/liquid-glass>
- Sample: Landmarks — building an app with Liquid Glass — <https://developer.apple.com/documentation/swiftui/landmarks-building-an-app-with-liquid-glass>
- WWDC25-323 Build a SwiftUI app with the new design — <https://developer.apple.com/videos/play/wwdc2025/323/>
- WWDC26-269 What's new in SwiftUI — <https://developer.apple.com/videos/play/wwdc2026/269/>
- WWDC26-278 Modernize your UIKit app — <https://developer.apple.com/videos/play/wwdc2026/278/>
- iOS & iPadOS 27 release notes — <https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-27-release-notes>
