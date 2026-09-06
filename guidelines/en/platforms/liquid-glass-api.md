---
id: platforms/liquid-glass-api
lang: en
version: 3
source-lang: en
status: active
digest: 7f7591e8
---

# Liquid Glass API reference

## Scope and verification

Reference for iOS 27 and native macOS 27 apps built with Xcode 27, with deployment targets set to 27.0. Verified on 2026-09-06 against Apple documentation and Xcode 27 beta build `27A5252f` (`iPhoneOS27.0.sdk`, `MacOSX27.sdk`). SDK declarations establish platform availability; the running OS determines rendering and interaction behavior. Beta declarations may change.

The tables describe support on these two platforms, not historical introduction versions. Use UIKit for iOS and AppKit for native macOS. Isolate platform-specific APIs with `#if os(iOS)` / `#if os(macOS)` or platform-specific files.

## SwiftUI glass APIs

All effects and styles in this section are available on both iOS 27 and macOS 27. Import `SwiftUI`; some declarations live in its `SwiftUICore` dependency.

| Symbol                                                        | Availability and use                                                          |
| ------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `glassEffect(_:in:)`                                          | Glass behind content; defaults to `.regular` and a capsule                    |
| `Glass.regular` / `.clear` / `.identity`                      | Adaptive / clear / no glass effect                                            |
| `Glass.tint(_:)` / `.interactive(_:)`                         | Configure tint and interactive response, including mouse interaction on macOS |
| `GlassEffectContainer(spacing:content:)`                      | Render related effects together; spacing controls blending                    |
| `glassEffectID(_:in:)`                                        | Stable element identity for morphing within a namespace                       |
| `glassEffectUnion(id:namespace:)`                             | Combine matching shapes and variants with the same union ID and namespace     |
| `glassEffectTransition(_:)`                                   | `.matchedGeometry`, `.materialize`, or `.identity`                            |
| `.buttonStyle(.glass)` / `.glassProminent` / `.glass(.clear)` | System glass button styles; preserve semantic actions and roles               |

## SwiftUI navigation and layout

| Symbol                                                                 | Availability and use                                        |
| ---------------------------------------------------------------------- | ----------------------------------------------------------- |
| `tabBarMinimizeBehavior(_:)`                                           | iOS: minimize the iPhone tab bar while scrolling            |
| `ToolbarSpacer(_:placement:)` / `sharedBackgroundVisibility(_:)`       | Both: separate toolbar groups and control their backgrounds |
| `scrollEdgeEffectStyle(_:for:)` / `scrollEdgeEffectHidden(_:for:)`     | Both: configure style or visibility per scroll edge         |
| `backgroundExtensionEffect()`                                          | Both: extend background content beneath adjacent UI         |
| `tabViewBottomAccessory(content:)` / `tabViewBottomAccessoryPlacement` | iOS: adapt accessory to `.inline` / `.expanded` / `nil`     |
| `tabViewBottomAccessory(isEnabled:content:)`                           | iOS: dynamically show or hide the accessory                 |

## UIKit and AppKit

| Symbol                                                                                              | Availability and use                                                                               |
| --------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `UIGlassEffect(style:)` / `UIGlassContainerEffect`                                                  | iOS: custom and grouped effects in `UIVisualEffectView`; configure `tintColor` and `isInteractive` |
| `UIButton.Configuration.glass()` / `.prominentGlass()` / `.clearGlass()` / `.prominentClearGlass()` | iOS: system button configurations                                                                  |
| `UIScrollView.topEdgeEffect` / `.bottomEdgeEffect`                                                  | iOS: configure `style` and `isHidden` on each edge effect                                          |
| `UIScrollEdgeElementContainerInteraction` / `UIBackgroundExtensionView`                             | iOS: register custom overlays / extend backgrounds                                                 |
| `UIBarButtonItem.hidesSharedBackground` / `.isHidden`                                               | iOS: hide the shared background / the item itself                                                  |
| `UITabBarController.tabBarMinimizeBehavior`                                                         | iOS: UIKit tab bar minimization                                                                    |
| `NSGlassEffectView` / `NSGlassEffectContainerView`                                                  | macOS: custom glass and grouping, with `contentView` and `spacing`                                 |
| `NSButton.BezelStyle.glass` / `NSBackgroundExtensionView`                                           | macOS: glass buttons / background extension                                                        |
| `NSGlassEffectView.effectIsInteractive`                                                             | macOS: interactive mouse response                                                                  |

## Adaptive navigation and window behavior

| Symbol                                                                                     | Availability and use                                                                                          |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| `toolbarMinimizationBehavior(_:for:)`                                                      | iOS navigation bars: minimization policy; use the current name rather than beta `toolbarMinimizeBehavior`     |
| `toolbarMinimizationSafeAreaAdjustment(_:for:)` / `toolbarMinimizationRestoration(_:for:)` | iOS navigation bars: safe-area adjustment and restoration policy                                              |
| `UINavigationItem.navigationBarMinimization`                                               | iOS: a `UIBarMinimization` value with `minimizationBehavior`, `safeAreaAdjustment`, and `restorationBehavior` |
| `TabRole.prominent` / `UITabBarController.prominentTabIdentifier`                          | iOS: designate the prominent tab                                                                              |
| `UITabBarController.Sidebar.preferredPlacement`                                            | iOS: opt into sidebar placement when space permits; check `isAvailable`                                       |
| `ToolbarContent.visibilityPriority(_:)`                                                    | Both: toolbar overflow priority                                                                               |
| `ToolbarOverflowMenu` / `.topBarPinnedTrailing`                                            | iOS only; unavailable on native macOS                                                                         |
| `toolbarColorScheme(_:for:)` / `toolbarVisibility(_:for:)` with `.statusBar`               | iOS: status-bar styling and visibility                                                                        |
| `appearsActive`                                                                            | Both: adapt custom content to window activity                                                                 |
| `UIMenuElement.preferredImageVisibility`                                                   | iOS: override default menu icon visibility where needed                                                       |

Validate the appearance slider, inactive-window styling, and scroll edges as part of the interface. UIKit apps must use the scene lifecycle and provide a launch screen. Layout must accommodate resizable iPhone windows and preserve access to essential actions.

## Implementation checks

- Build and type-check separately for iOS 27 and macOS 27. Shared SwiftUI code still needs platform separation for navigation, toolbars, and framework-specific views.
- Apply appearance modifiers before `glassEffect`, then identity, union, or transition modifiers. Use containers for related effects and measure actual rendering cost.
- Use semantic buttons for actions. `ToolbarItem` has no `isHidden` property; conditionally include the item in SwiftUI.
- Test custom labels, motion, safe areas, and backgrounds under accessibility settings. System material adaptation does not validate app content.
- Recheck beta names against the selected SDK; WWDC samples may retain earlier names.

## Official documentation

- [Glass](https://developer.apple.com/documentation/swiftui/glass)
- [GlassEffectContainer](https://developer.apple.com/documentation/swiftui/glasseffectcontainer)
- [Applying Liquid Glass to custom views](https://developer.apple.com/documentation/swiftui/applying-liquid-glass-to-custom-views)
- [UIGlassEffect](https://developer.apple.com/documentation/uikit/uiglasseffect)
- [NSGlassEffectView](https://developer.apple.com/documentation/appkit/nsglasseffectview)
- [iOS and iPadOS 27 release notes](https://developer.apple.com/documentation/ios-ipados-release-notes/ios-ipados-27-release-notes)
- [What’s new in SwiftUI — WWDC26](https://developer.apple.com/videos/play/wwdc2026/269/)
- [Modernize your UIKit app — WWDC26](https://developer.apple.com/videos/play/wwdc2026/278/)

Design: [Liquid Glass design](liquid-glass-design.md). Recipes: [Liquid Glass patterns](liquid-glass-patterns.md).
